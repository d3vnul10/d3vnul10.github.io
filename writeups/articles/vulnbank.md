---
layout: default
title: VulnBank Writeup
---

# 💀 VulnBank
### Category: WebHacking
### URL: 
<hr>

### Description:
Vuln bank is an intentionally Vulnerable bank built by [@commando_skiipz](https://x.com/commando_skiipz)
![image](/images/vulnbank/home_page.png)

Vulnerability 1: SQLI at login page leading to full Acc takeover

Step to Reproduce: 
1.Visit https://vulnbank.org/login

2.Input ```X'OR'1'='1``` as username and password
![image](/images/vulnbank/sqli_image.png)

3.You will be redirected to https://vulnbank.org/dashboard and logged in as the first user in the db
![image](/images/vulnbank/dashboard.png)

Vulnerality 2: API docs disclosure leaking internal banking endpoints

Step to Reproduce: 
1.Visit https://vulnbank.org/api/docs/

2.Grab the doc there We will need it later
![image](/images/vulnbank/home_page.png)

Vulnerality 3: Account Takeover through reset password token leakage 

Step to Reproduce: 
1.Visit https://vulnbank.org/forgot-password with burp or any other proxy software on to capture request

2.Input a name e.g admin,  you will redirect to https://vulnbank.org/reset-password 

3.Enter any invalid and notice the error message said invalid token

4.Go back to burp and look for the following requests ```POST /api/v3/forgot-password``` and ```/api/v3/reset-password```. Send them to Reapter

5.Change the part from ```POST /api/v3/forgot-password``` to ```POST /api/v1/forgot-password``` hit the send button and noticed the the rest password code got leaked
![image](/images/vulnbank/codeleaked.png)

6.change the requests ```/api/v3/reset-password``` to ```/api/v1/reset-password``` and replaced by the code we got from step 5 and boom password changed successfully
![image](/images/vulnbank/hacked.png)

Then login with the new creds and boom we got just compromise the admin acc 
![image](/images/vulnbank/admin.png)

I'm going to stop here as we have gain the the highest user priv from the lab

<hr>

Others vuln Discovered:

1. IDOR on `/transactions/{accountNumber}` `/check_balance/{accountNumber}` leading to other information access  
2. SSRF at `/upload_profile_picture_url` leaking Internal Secret  
3. Unauthorise admin account creation at `/admin/create_admin` through weak jwt secret and unverify jwt signature  
4. Unauthorise loan deletion and approval through weak jwt secret and unverify jwt signature  
5. Cross site scripting in several input field  
6. Bussiness logic leading to negative money transfer than leads deduction from receivers Account  

Thanks for reading

---

<a href="https://d3vnul10.github.io" style="display:inline-block; padding:8px 12px; background:#00ff00; color:#000; text-decoration:none; border-radius:4px;">
⬅ Back to Home
</a>
