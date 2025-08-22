# Challenge 1 – Discovery

## 1. Overview
The objective of this challenge is to perform reconnaissance on the target domain `bitamer.com`:
- Enumerate subdomains.
- Identify open ports and services.

## 2. Tools & Setup
The following tools were used:
- **Amass** – passive subdomain enumeration.  
- **Subfinder** – for extended subdomain discovery.
- **Nmap** – service/version detection.

  ## 3. Initial Results with Amass

  ### Findings
Running `amass` against `bitamer.com` produced the following:
- **NS records**:  
  - `journey.ns.cloudflare.com`  
  - `stan.ns.cloudflare.com`
- **A/AAAA records**: all resolved to Cloudflare IP ranges.  
- **ASN / Netblocks**:  
  - ASN **13335** – CLOUDFLARENET (Cloudflare, Inc.)  
  - Ranges include `172.64.0.0/18`, `162.159.32.0/20`, `2606:4700:50::/44`, etc.

### Why This Was Not Useful
- All discovered assets point to **Cloudflare infrastructure**.  
- Scanning Cloudflare IPs only reveals the WAF/reverse proxy, not the real backend servers.  
- Therefore, Amass results alone are of **low practical value** for this challenge.

