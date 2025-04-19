# Penetration Testing Lab
## Overview
A lab to practice ethical hacking using Kali Linux to test a pfSense-protected network with a DMZ web server and LAN clients.
## Network Setup

This lab builds on the Home Network Security Lab ([link](https://github.com/TsikyLalaina/security-lab)).

### Existing Components
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
