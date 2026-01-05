I started by running an Nmap scan to see what services were exposed on the target machine. This helps understand the attack surface before diving deeper.

First, I ran a standard scan with default scripts and version detection:

```bash
sudo nmap -sS -p- 10.80.160.139 -oN port-scan.txt -T5
