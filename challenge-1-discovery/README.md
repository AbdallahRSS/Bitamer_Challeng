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
  ```bash
   amass enum -d bitamer.com -o amass.txt
  ```


![Alt text for screen readers](https://github.com/AbdallahRSS/Bitamer_Challeng/blob/8ada053477d1d9a3978c5066643ddfb3d433b529/challenge-1-discovery/screenshots/Screenshot%202025-08-22%20075556.png)


  ### Findings
  Here is the raw Amass output stored in [`amass.txt`](scanefiles/amass.txt).
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

##4. Results of Subfinder
We used **Subfinder** to discover subdomains of `bitamer.com`.

```bash
subfinder -d bitamer.com -o subfinder.txt
```

All discovered subdomains were saved in 

