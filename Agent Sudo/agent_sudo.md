# Agent Sudo – TryHackMe Write-up

## Overview

This repository documents the exploitation of the **Agent Sudo** TryHackMe room. The assessment focused on service enumeration, web application analysis, credential discovery, steganographic analysis, initial access, and Linux privilege escalation. The objective was to obtain root access by following a structured penetration testing methodology. :contentReference[oaicite:0]{index=0}

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

## 1. Service Enumeration

### Objective

Identify the services exposed by the target to determine the available attack surface.

### Why Nmap?

Nmap is the industry-standard network reconnaissance tool used to discover live hosts, identify open ports, detect running services, and enumerate service versions. It provides the foundation for every penetration test by revealing potential entry points.

### Command

```bash
nmap -sC -sV <TARGET_IP>
```

### Command Explanation

| Option | Purpose |
|---------|---------|
| `-sC` | Runs Nmap's default NSE scripts for basic enumeration. |
| `-sV` | Detects service versions running on open ports. |

### Observation

The scan identified three exposed services:

- HTTP (80)
- FTP (21)
- SSH (22)

The web application contained a message instructing users to access the site using a specific **User-Agent**, indicating that HTTP header manipulation would likely be required.
