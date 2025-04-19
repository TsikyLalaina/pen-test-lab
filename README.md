# Penetration Testing Lab

A hands-on project to practice ethical hacking using Kali Linux, targeting a pfSense-protected network with a DMZ web server and LAN clients. Built in VirtualBox on Arch Linux with Hyprland, this lab focuses on reconnaissance, enumeration, simulated attacks, and firewall log analysis to enhance network security skills for a system and network administrator internship.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Network Setup](#network-setup)
- [Penetration Testing](#penetration-testing)
- [Challenges and Solutions](#challenges-and-solutions)
- [Future Improvements](#future-improvements)
- [Tools Used](#tools-used)

---

## Project Overview

This lab extends the [Home Network Security Lab](https://github.com/TsikyLalaina/security-lab) by adding a penetration testing environment to identify vulnerabilities and strengthen defenses. Key objectives include:

- **Simulate Attacks**: Use Kali Linux to perform reconnaissance, enumeration, and controlled attacks.
- **Analyze Defenses**: Leverage pfSense firewall logs to monitor and block malicious traffic.
- **Harden Security**: Apply lessons to improve network security.

The lab targets a DMZ web server (`192.168.2.101`) and LAN clients (`192.168.10.100`, `192.168.10.103`) behind a pfSense firewall, with a Kali Linux VM as the attacker.

---

## Network Setup

### Existing Components (Home Network Security Lab)

- **pfSense VM**: Firewall managing:
  - WAN: NAT to Internet
  - LAN: `192.168.10.1/24` (DHCP range: `192.168.10.100–200`)
  - DMZ: `192.168.2.1/24`
- **LAN Client (Ubuntu)**: `192.168.10.100`, Ubuntu, 2GB RAM, 10GB disk.
- **DMZ Web Server (Alpine)**: `192.168.2.101`, Alpine Linux, Nginx, 512MB RAM, 2GB disk.
- **Firewall Rules**:
  - LAN: Allow to DMZ (`192.168.2.101:80`), allow to Internet, block RFC1918 (excluding lab subnets).
  - DMZ: Allow to Internet, block to LAN, block RFC1918.
- **Topology**: See diagram ([topology.png](https://github.com/TsikyLalaina/security-lab/blob/main/topology.png)).

![pfSense Web UI](https://github.com/TsikyLalaina/security-lab/blob/main/pfSenseWebUI.png)

![pfSense Console](https://github.com/TsikyLalaina/security-lab/blob/main/pfSense.png)

### New Components

#### Kali Linux VM (Attacker)

- **Download**: Kali Linux 2024.4 ISO from [kali.org](https://www.kali.org/get-kali/).
- **VM Specs**: 4GB RAM, 20GB disk, Internal Network `LAN`.
- **Installation**:
  - Hostname: `kali-attacker`
  - User: `pentester` (password: `kali123!`)
  - Software: Default pentesting tools.
- **Network**: Assigned IP `192.168.10.102` via DHCP from pfSense.
- **Verification**: Confirmed pfSense web UI access (`https://192.168.10.1`) via Firefox.

#### Debian Minimal VM (Lightweight LAN Client)

- **Rationale**: Ubuntu VM (`192.168.10.100`) was too heavy; added a lighter client.
- **Download**: Debian 12.7.0 netinst ISO from [debian.org](https://www.debian.org/distrib/netinst).
- **VM Specs**: 512MB RAM, 4GB disk, Internal Network `LAN`.
- **Installation**:
  - Hostname: `debian-client`
  - User: `debianuser` (password: `debian123`)
  - Software: Minimal install (only standard utilities, no desktop).
- **Network**: Assigned IP `192.168.10.103` via DHCP.
- **Verification**: Accessed pfSense web UI (`https://192.168.10.1`) using `curl` and `lynx`.

![DMZ Web Server](https://github.com/TsikyLalaina/security-lab/blob/main/AlpineDMZserver.png)

### Firewall Adjustments

- **Temporary Rule**: Added "Allow LAN to DMZ for pentesting" (Protocol: Any, Source: LAN net, Destination: DMZ net, Log: Enabled).
- **Web UI Access**: Ensured LAN can access pfSense (`192.168.10.1:443`) for Debian VM.

---

## Penetration Testing

### Reconnaissance

- **LAN Scan**: `nmap -sP 192.168.10.0/24`
  - Found: `192.168.10.1` (pfSense), `192.168.10.100` (Ubuntu), `192.168.10.102` (Kali), `192.168.10.103` (Debian).
- **DMZ Scan**: `nmap -sP 192.168.2.0/24`
  - Found: `192.168.2.1` (pfSense), `192.168.2.101` (Alpine).
- **Service Scan**: `nmap -sV -p- 192.168.2.101`
  - Result: Port 80 open, running Nginx.

### Enumeration

- **Web Access**: Visited `http://192.168.2.101`
  - Confirmed: "Welcome to DMZ Web Server".
- **Directory Scan**: `gobuster dir -u http://192.168.2.101 -w /usr/share/wordlists/dirb/common.txt`
  - Result: No notable directories found.
- **Vulnerability Scan**: `nikto -h http://192.168.2.101`
  - Result: No critical vulnerabilities; checked for outdated headers.

### Simulated Attacks

- **Port Scan**: `nmap -A -T4 192.168.2.101`
  - Aggressive scan to trigger firewall logs.
- **Brute Force**: `hydra -l admin -P /usr/share/wordlists/rockyou.txt http://192.168.2.101`
  - Dummy attempt (no login page); generated traffic for logging.

### Log Analysis

- **Firewall Logs**: Checked Status > System Logs > Firewall.
  - Observed blocked traffic from Kali (`192.168.10.102`) during aggressive scans.
  - Confirmed allowed HTTP traffic to `192.168.2.101:80`.
- **Packet Capture**: Used Diagnostics > Packet Capture on DMZ interface (Filter: `host 192.168.10.102`).
  - Analyzed with Wireshark on Arch host.

### Hardening

- **Firewall**:
  - Modified "Allow LAN to DMZ for pentesting" to TCP, port 80 only.
  - Added "Block non-HTTP DMZ scans" (Block TCP/UDP, ports 1–79, 81–65535, Source: LAN net, Destination: DMZ net).
- **Nginx**: Updated on DMZ server (`apk upgrade nginx`).

---

## Challenges and Solutions

- **Ubuntu VM Heavy**: Added Debian Minimal VM (`192.168.10.103`) as a lightweight LAN client to access pfSense web UI.
- **Firewall Blocking**: Adjusted rules to allow pentesting traffic while logging attacks.
- **No Vulnerabilities**: DMZ web server was minimal; future tests could add a vulnerable app (e.g., DVWA).

---

## Future Improvements

- Add a vulnerable web app (e.g., Damn Vulnerable Web App) to the DMZ server.
- Simulate advanced attacks (e.g., SQL injection, XSS).
- Integrate Suricata for IDS/IPS to enhance threat detection.

---

## Tools Used

- **OS**: Arch Linux with Hyprland
- **Virtualization**: VirtualBox
- **Attacker**: Kali Linux 2024.4
- **Clients**: Ubuntu, Debian Minimal
- **Target**: Alpine Linux with Nginx
- **Firewall**: pfSense 2.7.2
- **Pentesting Tools**: Nmap, Gobuster, Nikto, Hydra

---

*Last updated: April 19, 2025*