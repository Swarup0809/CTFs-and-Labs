# Agent Sudo – TryHackMe Write-up

## Overview

This repository documents the exploitation of the **Agent Sudo** TryHackMe room. The assessment focused on service enumeration, web application analysis, credential discovery, steganographic analysis, initial access, and Linux privilege escalation. The objective was to obtain root access by following a structured penetration testing methodology. 

---

## Objectives

- Enumerate exposed services
- Identify application misconfigurations
- Discover valid credentials
- Analyze hidden files and embedded data
- Gain initial SSH access
- Escalate privileges to root

---

## Lab Information

| Category | Details |
|----------|---------|
| Platform | TryHackMe |
| Room | Agent Sudo |
| Operating System | Ubuntu Linux |
| Difficulty | Easy |
| Focus Areas | Enumeration, Web Exploitation, Steganography, Password Cracking, Privilege Escalation |

---

# Enumeration

## Service Discovery

The first step was to identify the services exposed by the target.

### Command

```bash
nmap -sC -sV <TARGET_IP>
```

### Why Nmap?

Nmap is used to enumerate open ports, detect running services, and identify service versions. This establishes the initial attack surface before interacting with the target.

### Result

The scan identified the following services:

- HTTP
- FTP
- SSH

The HTTP web application displayed the following message:

```
Dear agents,

Use your own codename as user-agent to access the site.

From,
Agent R
```

<p align="center">
<img src="/ss/http_img.png" width="900">
</p>

<p align="center">
<b>Figure 1.</b> Text displayed on the webpage of the target ip.
</p>
---

# User-Agent Enumeration

The webpage suggested that access depended on the HTTP User-Agent header.

## Why Burp Suite?

Burp Suite allows interception and modification of HTTP requests without changing the browser itself, making it suitable for testing request headers.

The intercepted request was sent to **Intruder**.

A payload containing characters from **A-Z** was used to brute-force the User-Agent header.

### Observation

Most requests returned:

```
HTTP 200
```

One request returned:

```
HTTP 302
```

when the User-Agent value was:

```
C
```

A redirect indicated that the correct user-agent had been discovered.

The modified request was forwarded using **Repeater**, replacing the header with:

```
User-Agent: C
```

### Screenshot

```md
![Burp Intruder](02_images/burp_intruder.png)
```

```md
![HTTP 302 Response](02_images/http302.png)
```

---

# FTP Enumeration

Anonymous FTP access was checked before attempting authentication.

### Command

```bash
nmap --script ftp-anon <TARGET_IP> -p 21
```

### Why ftp-anon?

The `ftp-anon` NSE script quickly verifies whether anonymous authentication is enabled on an FTP server.

### Result

Anonymous login was disabled.

### Screenshot

```md
![FTP Anonymous Check](02_images/ftp_anon.png)
```

---

# SSH Credential Brute Force

The webpage referenced an agent named **Chris**, suggesting a potential username.

The password appeared weak, making password brute forcing a reasonable approach.

## Why Hydra?

Hydra performs online password attacks against network authentication services.

### Command

```bash
hydra -l chris -P <WORDLIST> ssh://<TARGET_IP>
```

### Result

```
Username : chris
Password : crystal
```

### Screenshot

```md
![Hydra](02_images/hydra.png)
```

---

# FTP File Retrieval

The recovered credentials were used to authenticate to the FTP server.

### Command

```bash
ftp <TARGET_IP>
```

```bash
get TO_agentJ.txt
get cute-alien.jpg
get cutie.png
```

### Downloaded Files

- TO_agentJ.txt
- cute-alien.jpg
- cutie.png

### Screenshot

```md
![FTP Download](02_images/ftp_download.png)
```

---

# Image Analysis

The downloaded files suggested that sensitive information was hidden within one of the images.

---

## Metadata Analysis

### Why ExifTool?

ExifTool extracts metadata from files and can reveal abnormal structures or hidden content.

### Command

```bash
exiftool cutie.png
```

### Observation

The output indicated additional data after the PNG IEND chunk.

### Screenshot

```md
![ExifTool](02_images/exiftool.png)
```

---

## String Extraction

### Why strings?

The `strings` utility extracts readable ASCII text from binary files, often revealing embedded information.

### Command

```bash
strings cutie.png
```

### Screenshot

```md
![Strings Output](02_images/strings.png)
```

---

## Embedded File Detection

### Why Binwalk?

Binwalk scans binary files to identify embedded files, compressed archives, or appended data.

### Command

```bash
binwalk cutie.png
```

### Screenshot

```md
![Binwalk](02_images/binwalk.png)
```

---

## Password Hash Extraction

### Why zip2john?

The embedded archive was password protected.

`zip2john` extracts password hashes into a format compatible with John the Ripper.

### Command

```bash
zip2john <archive.zip> > hash.txt
```

### Password Recovery

The recovered archive password was:

```
alien
```

---

## Archive Extraction

### Why 7z?

7-Zip supports extraction of encrypted archives after providing the recovered password.

### Command

```bash
7z x <archive.zip>
```

Password:

```
alien
```

### Screenshot

```md
![7z Extraction](02_images/7z.png)
```

---

# Decoding Hidden Data

The extracted content contained Base64 encoded data.

### Why Base64?

Base64 is an encoding scheme commonly used to store binary information as readable text.

### Command

```bash
base64 -d <file>
```

### Result

```
Area51
```

This value appeared to be a passphrase for the remaining image.

---

# Steganography Analysis

The second image (`cute-alien.jpg`) was inspected.

### Initial Verification

### Command

```bash
head cute-alien.jpg
```

The output began with:

```
JFIF
```

confirming it was a valid JPEG image.

---

## Hidden File Extraction

### Why Steghide?

Steghide extracts files embedded inside images using steganography.

### Command

```bash
steghide extract -sf cute-alien.jpg
```

Passphrase:

```
Area51
```

### Result

A hidden file named:

```
message.txt
```

was extracted.

### Screenshot

```md
![Steghide](02_images/steghide.png)
```

---

# Credential Recovery

The extracted message contained SSH credentials.

```
Hi James,

Glad you find this message.
Your login password is hackerrules!

Your buddy,
Chris
```

These credentials were used to access the target as **James**.

### Screenshot

```md
![Message](02_images/message.png)
```

---

# Privilege Escalation

After logging in as James, sudo permissions were enumerated.

### Command

```bash
sudo -l
```

### Result

```
(ALL, !root) /bin/bash
```

### Why sudo -l?

This command lists commands the current user is permitted to execute via sudo and often reveals privilege escalation opportunities.

The discovered configuration was researched and matched a known privilege escalation technique.

Reference:

- ExploitDB

### Screenshot

```md
![Sudo Permissions](02_images/sudo_l.png)
```

---

# Root Access

The identified sudo misconfiguration was exploited using the corresponding privilege escalation technique.

### Command

```bash
<Privilege Escalation Command>
```

### Result

Root shell obtained successfully.

### Screenshot

```md
![Root Access](02_images/root.png)
```

---

# Tools Used

| Tool | Purpose |
|------|---------|
| Nmap | Network discovery, port scanning and service enumeration |
| Burp Suite | Intercepting and modifying HTTP requests |
| Hydra | Online password brute forcing |
| FTP | File retrieval from the FTP server |
| ExifTool | Metadata analysis |
| strings | Extraction of readable strings from binary files |
| Binwalk | Detection of embedded files and archives |
| zip2john | Extracting password hashes from ZIP archives |
| 7-Zip | Extracting encrypted archives |
| Base64 | Decoding encoded data |
| Steghide | Extracting hidden files embedded inside images |
| sudo | Enumerating executable commands for privilege escalation |

---

# Skills Demonstrated

- Network Enumeration
- HTTP Request Manipulation
- FTP Enumeration
- Password Brute Forcing
- Metadata Analysis
- Binary File Inspection
- Archive Password Recovery
- Base64 Decoding
- Steganography Analysis
- SSH Authentication
- Linux Privilege Escalation
