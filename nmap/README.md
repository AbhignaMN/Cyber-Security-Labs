# Nmap Practice — Day 1 (26 Nov 2026)

Targets: my own Kali VM (127.0.0.1, 10.0.2.15) and scanme.nmap.org
(a host the Nmap project provides for light, authorised testing).
Rule: only scan systems I own or have explicit permission to test.

## Port states
| State    | Meaning                                         | Seen in my scans          |
|----------|-------------------------------------------------|---------------------------|
| open     | a program is listening and accepting connections | 22, 8000 on localhost     |
| closed   | port reachable, nothing listening               | 80 on localhost           |
| filtered | firewall dropped the probe, state unknown       | 443 on scanme.nmap.org    |

## Commands and what they do

### Basics
- `nmap <target>` — scans the 1,000 most common TCP ports
- `nmap -sn 10.0.2.0/24` — host discovery only (which machines are up), no port scan

### Choosing ports
- `-p 22` / `-p 22,80,8000` / `-p 1-1000` — specific ports, lists, ranges
- `-p-` — all 65,535 ports (services often hide on unusual ports)
- `--top-ports 20` — the N most common ports

### Scan types
- `-sS` — SYN "half-open" scan: sends SYN, gets SYN-ACK, replies RST instead of
  completing the handshake. Fast and quieter. Needs root (raw packets).
- `-sT` — TCP connect scan: completes the full 3-way handshake. No root needed,
  but noisier and logged by the target.
  (My scans: -sS reported closed ports as "reset", -sT as "conn-refused".)
- `-sU` — UDP scan. Slow, so limit ports.
- `-Pn` — skip the "is the host up?" ping check (useful when ICMP is blocked)

### Finding what's running
- `-sV` — detects service name and version
- `-O` — guesses the operating system (needs root)
- `-sC` — runs default NSE scripts (SSH host keys, HTTP title, server header…)
- `--script <name>` — runs a specific script, e.g. `--script http-title`
- `--script vuln` — vulnerability-check scripts (lab targets only)
- `-A` — aggressive: -O + -sV + -sC + traceroute. Loud.

### Speed and output
- `-T0` to `-T5` — timing (T4 is the common choice)
- `-v` — verbose progress
- `-oN file.txt` — save normal output; `-oA name` — save all 3 formats

### Go-to full scan
```
sudo nmap -sS -sV -sC -p- -T4 -oN full_scan.txt <target>
```
SYN scan of every port, with versions and default scripts, saved to a file.

## Findings

### scanme.nmap.org
- 22/tcp open — OpenSSH 6.6.1p1 (Ubuntu)
- 80/tcp open — Apache httpd 2.4.7 (Ubuntu), title "Go ahead and ScanMe!"
- 9929/tcp open — Nping echo
- 443/tcp filtered
- Observation: OpenSSH 6.6.1 and Apache 2.4.7 are very old versions with
  publicly known CVEs. In a real assessment this would be reported as
  "outdated services exposed — upgrade to supported versions."

### Local VM (127.0.0.1)
- 22/tcp open — OpenSSH 10.4p1 Debian
- 8000/tcp open — Python SimpleHTTPServer (test service I started)
- -sn on 10.0.2.0/24 found 3 hosts: my VM (10.0.2.15) plus VirtualBox's
  gateway (10.0.2.2) and DNS (10.0.2.3)

## Files
- scan1.txt — `nmap -sC -sV` on scanme.nmap.org
- full_scan.txt — full SYN + version + script scan of all ports on localhost
