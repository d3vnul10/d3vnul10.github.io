---
layout: default
title: Home
---

<div class="nav-menu">
  <a href="./index.html" class="nav-link">Main</a>
  <a href="/writeups/thm/index.html" class="nav-link">TryHackMe</a>
  <a href="/writeups/htb/index.html" class="nav-link">HackTheBox</a>
  <a href="/writeups/pg/index.html" class="nav-link">ProvingGround</a>
  <a href="/writeups/picoctf/index.html" class="nav-link">PicoCTF</a>
  <a href="/writeups/rootme/index.html" class="nav-link">ROOTME</a>
  <a href="/writeups/bbp/index.html" class="nav-link">BBP Writeups</a>
  <a href="/writeups/articles/index.html" class="nav-link">Random Articles</a>
  <a href="/writeups/livectf/index.html" class="nav-link">Live CTF</a>
</div>

<style>
.nav-menu {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  overflow-x: auto; 
  padding-bottom: 10px;
  -webkit-overflow-scrolling: touch;
}

.nav-link {
  text-decoration: none;
  font-weight: bold;
}

.nav-link:hover {
  text-decoration: underline;
}

/* Profile Section */
.profile-container {
  display: flex;
  flex-wrap: wrap; /* allows wrapping on smaller screens */
  gap: 20px;
  margin-bottom: 30px;
  align-items: flex-start;
}

.profile-container img {
  width: 300px; /* fixed width for large screens */
  height: 250px; /* fixed height */
  border-radius: 8px;
  padding: 5px;
  flex: 0 0 auto; /* prevents shrinking on large screens */
}

.profile-info {
  font-family: 'Courier New', monospace;
  line-height: 1.6;
  flex: 1 1 300px; /* flexible on smaller screens */
  min-width: 200px;
}

/* Responsive adjustments */
@media (max-width: 768px) {
  .profile-container {
    flex-direction: column; /* stack image and text */
    align-items: center;
    text-align: center;
  }

  .profile-info {
    flex: 1 1 100%; /* full width text */
  }
}

@media (max-width: 480px) {
  .profile-container img {
    width: 90%; /* shrink image on very small screens */
    height: auto; /* maintain aspect ratio */
  }

  .nav-menu {
    gap: 10px;
  }
}
</style>

* * *
### Welcome to my personal CyberSec blog
* * *

Hey there ⚡, welcome to my Cybersecurity Blog! I’m **d3vnul10**. Here, I post hacking guides, writeups, exploits, and anything I uncover in the world of hacking.

<div class="profile-container">

  <img src="/images/me.jpeg" alt="d3vnul10"/>

  <div class="profile-info">
    <p><strong>ID:</strong> Aderogba Rufai</p>
    <p><strong>Whoami:</strong> d3vnul10</p>
    <p><strong>Role:</strong> CTF Player | Bug Hunter | Penetration Tester | Sec. Breaker | Learner</p>
    <p><strong>Medium:</strong> <a href="https://devnull-0.medium.com" target="_blank">@devnull-0</a> (check out previous writeups)</p>
    <p><strong>Skills:</strong> Web Hacking | API Hacking | AppSec | Binary Exploitation</p>
    <p><strong>Programming:</strong> Bash | Python | PHP | JavaScript | C</p>
    <p><strong>Learning Platforms:</strong> PortSwigger | THM | HTB | PentestLab | OffSec | etc.</p>
    <p><strong>CTF Platforms:</strong> RootMe | PwnTillDawn | PicoCTF | OverTheWire | etc.</p>
  </div>

</div>

* * *
### **TryHackMe Writeups:~#**
- [[Oct 21 2025]] [Develpy](https://devnull-0.medium.com/develpy-tryhack-writeup-2ec70ff6d3ba) `python2, input injection`
- [[Oct 21 2025]] [PickleRick](https://devnull-0.medium.com/pickle-rick-e8e8a5c6df21) `Info leak, Cmdline Injection`
- [[Oct 21 2025]] [TomGhost](https://devnull-0.medium.com/tomghost-thm-3450ccef5b12) `CVE-2020-1938`
- [[Mar 06 2026]] [Relevant](https://devnull-0.medium.com/relevant-hacking-a-window-machine-in-10-mins-473fa7d60918) `Window Machine, Unsecure ftp`

* * *

* * *
### **ProvingGrounds:~#**                                                                                                                                                

* * *

* * *
### **Hack The Box Writeups:~#**

* * *

* * *
### **PicoCTF:~#**                                                                                                                                                
- [[Oct 23 2025]] [Crack The Gate1](https://devnull-0.medium.com/crack-the-gate-1-picomini-2025-56b29d437f1d) `debug message, auth bypass`
- [[Oct 23 2025]] [Crack The Gate2](https://devnull-0.medium.com/crack-the-gate-2-picomini-d6aae6cbca03) `Rate limit   Bypass, x-Forwarded-For`
- [[Oct 28 2025]] [Byp4ss3d](https://devnull-0.medium.com/byp4ss3d-picomini-by-cmu-africa-271ea202309c) `.htaccess overide, file upload, shell`
- [[Nov 09 2025]] [JAuth](https://devnull-0.medium.com/jauth-picoctf-3325e62c4321) `JWT, ALG:NONE, piv ESc`
- [[Dec 17 2025]] [Input injection1](https://devnull-0.medium.com/input-injection-1-picogym-77fb1ae909b7) `Binary, Buffer overflow`
- [[Dec 17 2025]] [Input injection2](https://devnull-0.medium.com/input-injection-2-picogym-96875d1f4159) `Binary, Buffer overflow`

* * *

* * *
### **RootMe:~#**
- [[Nov 17 2025]] [Blind SSTI](https://devnull-0.medium.com/python-blind-ssti-filters-bypass-9585f421db67) `SSTI, python, filter bypass`

* * *

* * *
### **BBP writeups:~#**                                                                                                                                                

* * *

* * *
### **Articles:~#**
- [[Mar 29 2026]] [VulnBank](https://d3vnul10.github.io/writeups/articles/vulnbank.html) `API Hacking, Priv Esc, WebHacking Owasp`
- [[Mar 31 2026]] [Snare](https://d3vnul10.github.io/writeups/articles/snare.html) `PwnTillDawn LFI Boot2Root`
- [[Apr 10 2026]] [NFS Hacking](https://d3vnul10.github.io/writeups/articles/badger.html) `NFS Boot2Root`

* * *

* * *
### **Live CTF:~#**
- [[Oct 27 2025]] [Hack the night](https://devnull-0.medium.com/deadface-ctf-2025-part3-hack-the-night-10d29525be79) `DeadfaceCTF 2025, web exploitation`
- [[Oct 27 2025]] [Stolen Secret](https://devnull-0.medium.com/deadface-ctf-2025-part-1-stolen-secrets-ad71c9397209) `DeadfaceCTF 2025, Forensics pcap file`
- [[Oct 27 2025]] [SQLite007](https://devnull-0.medium.com/deadface-ctf-2025-part-2-sqlite007-000c4609e020) `DeadfaceCTF 2025, SQLI, web`
- [[Oct 27 2025]] [EpicSales](https://devnull-0.medium.com/epicsales-deadface-2025-92512872d905) `DeadfaceCTF 2025, SQL`
- [[May 01 2026]] [CYSSSA CTF](https://devnull-0.medium.com/cysssa-ctf-2026-write-ups-319dd81c0944)`CYSSSA CTF, Academic competition`
  
* * *
