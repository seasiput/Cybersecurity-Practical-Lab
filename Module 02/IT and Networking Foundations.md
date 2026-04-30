**Module 02 — Network Context, Service Exposure, Troubleshooting, Monitoring, and Architecture Thinking**

This guide is designed to establish workflows that align closely with operational needs. The focus of this module is not on overly basic networking introductions, but rather on how a student reads host conditions, validates services, tests connectivity, analyzes traffic and logs, and concludes the practice with an assessment that can be understood professionally.

**1. Learning Objectives**
- Build a working understanding of how hosts, interfaces, routes, resolvers, services, and local targets are interconnected within a shared environment.
- familiarize to distinguishing between internal host observations and workstation assessment validations.
- Perform network troubleshooting in a disciplined, evidence-based sequence rather than based on guesswork.
- Read logs, packet summaries, and service outputs to produce interpretations that are more valuable than just command results.
- Conclude the practice with a brief assessment that serves as a bridge to Module 03: Operating Systems & Scripting.

**2. Environment Roles**
<img width="1663" height="611" alt="image" src="https://github.com/user-attachments/assets/6f6c8b89-146b-4a7f-a363-fcf507e06318" />

**3. Practical Workflow**
- Set up the Module 02 workspace and ensure host context is clear without repeating unnecessary basic orientation.
- Read the baseline host and network stack on Ubuntu.
- Perform controlled service and exposure validation from Kali against local targets and relevant hosts.
- Execute connectivity troubleshooting in a disciplined sequence.
- Read log and PCAP samples to build operational monitoring context.
- Compile architecture and segmentation notes relevant to the lab environment.
- Conclude the practice with an assessment linking networking foundations to Module 03.

**Module 02_Phase 1 — Workspace and Baseline Host Context**
The goal of this phase is to immediately establish the Module 02 workspace and read the primary host's network stack with a more operational perspective.

**Phase 1.1. Build Module 02 Workspace on Ubuntu**
```
mkdir -p ~/module02/{evidence/baseline,reports,notes,services,troubleshooting,monitoring,architecture,tmp} 
cd ~/module02
ls -R ~/module02  
ls -la
```

Command Function: 
- Creates a specific work structure for networking, monitoring, and architecture notes.
- It ensures evidence remains neatly separated by technical category.

Expected Output:
- The active path is at ~/module02.
- We are positioned to work like an analyst who organizes evidence by work domain, rather than placing all output in one location.

```
jccsah001012@localhost:~/module02$ mkdir -p ~/module02/{evidence/baseline,reports,notes,services,troubleshooting,monitoring,architecture,tmp}
jccsah001012@localhost:~/module02$ ls -R ~/module02
/home/jccsah001012/module02:
architecture  baseline  evidence  monitoring  notes  reports  services  tmp  troubleshooting

/home/jccsah001012/module02/architecture:

/home/jccsah001012/module02/baseline:
dns_check.txt

/home/jccsah001012/module02/evidence:
baseline

/home/jccsah001012/module02/evidence/baseline:

/home/jccsah001012/module02/monitoring:

/home/jccsah001012/module02/notes:

/home/jccsah001012/module02/reports:

/home/jccsah001012/module02/services:

/home/jccsah001012/module02/tmp:

/home/jccsah001012/module02/troubleshooting:
jccsah001012@localhost:~/module02$ ls -la
total 44
drwxrwxr-x 11 jccsah001012 jccsah001012 4096 Apr 30 10:20 .
drwx------ 11 jccsah001012 jccsah001012 4096 Apr 30 10:20 ..
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 architecture
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:35 baseline
drwxrwxr-x  3 jccsah001012 jccsah001012 4096 Apr 30 11:29 evidence
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 monitoring
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 notes
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 reports
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 services
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 tmp
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 Apr 30 10:20 troubleshooting
```

**Phase 1.2. Record Baseline Host and System Timebon Ubuntu**
```
whoami | tee ~/module02/evidence/baseline/whoami.txt 
hostname | tee ~/module02/evidence/baseline/hostname.txt 
date | tee ~/module02/evidence/baseline/date.txt 
uname -a | tee ~/module02/evidence/baseline/uname.txt
```

Command Function: Records host identity and system time as the initial context before network observations are performed.

Expected Output of `whoami.txt`, `hostname.txt`, `date.txt`, and `uname.txt` are available :
<img width="2036" height="290" alt="image" src="https://github.com/user-attachments/assets/9ed19b4f-3fd2-4e68-b417-9479af798b46" />

**Phase 1.3. Read Interfaces, Addresses, and Routes on Ubuntu**
```
ip addr | tee ~/module02/evidence/baseline/ip_addr.txt 
ip route | tee ~/module02/evidence/baseline/route.txt 
resolvectl status | tee ~/module02/evidence/baseline/dns_check.txt
```

Command Function: 
• Understand the host’s network stack from the perspective of interfaces, routing, and DNS resolution.
• Prepare baseline data for diagnosing potential connectivity failures later.

Expected Output of a list of interfaces, IP addresses, the default route, and the active resolver is displayed. Effective troubleshooting always starts from the host itself, not from assumptions about the target.

```
jccsah001012@localhost:~/module02$ ip addr | tee ~/module02/evidence/baseline/ip_addr.txt
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute 
       valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    link/ether 22:00:18:b1:fd:89 brd ff:ff:ff:ff:ff:ff
    inet 139.162.57.122/24 brd 139.162.57.255 scope global eth0
       valid_lft forever preferred_lft forever
    inet6 2400:8901::2000:18ff:feb1:fd89/64 scope global dynamic mngtmpaddr noprefixroute 
       valid_lft 5380sec preferred_lft 1780sec
    inet6 fe80::2000:18ff:feb1:fd89/64 scope link 
       valid_lft forever preferred_lft forever
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 62:cd:b2:a2:c2:09 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
    inet6 fe80::60cd:b2ff:fea2:c209/64 scope link 
       valid_lft forever preferred_lft forever
4: vetha8e0b21@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 6a:7f:9d:fa:ba:45 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet6 fe80::687f:9dff:fefa:ba45/64 scope link 
       valid_lft forever preferred_lft forever
5: veth60cffd4@if2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default 
    link/ether 82:96:d4:53:6c:f9 brd ff:ff:ff:ff:ff:ff link-netnsid 1
    inet6 fe80::8096:d4ff:fe53:6cf9/64 scope link 
       valid_lft forever preferred_lft forever
jccsah001012@localhost:~/module02$ ip route | tee ~/module02/evidence/baseline/route.txt
default via 139.162.57.1 dev eth0 proto static 
139.162.57.0/24 dev eth0 proto kernel scope link src 139.162.57.122 
172.17.0.0/16 dev docker0 proto kernel scope link src 172.17.0.1 
jccsah001012@localhost:~/module02$ resolvectl status | tee ~/module02/baseline/dns_check.txt
Global
         Protocols: -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
  resolv.conf mode: stub

Link 2 (eth0)
    Current Scopes: DNS
         Protocols: +DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
Current DNS Server: 139.162.3.6
       DNS Servers: 139.162.3.6 139.162.9.5 103.3.60.20
        DNS Domain: members.linode.com

Link 3 (docker0)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 4 (vetha8e0b21)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported

Link 5 (veth60cffd4)
    Current Scopes: none
         Protocols: -DefaultRoute -LLMNR -mDNS -DNSOverTLS DNSSEC=no/unsupported
```

**Module 02_Phase 2 — Service Exposure and Validation Perspective**
This phase distinguishes two perspectives that are often confused: what is visible from inside the host, and what is visible from the validation workstation. Ubuntu is used to observe the internal state, while Kali is used to validate exposure in a controlled manner.

**Phase 2.1. Read Listening Ports and Process Context from Ubuntu**
```
ss -tulpn | tee ~/module02/evidence/services/ss_output.txt
ps -ef | grep -E "nginx|ssh|docker|python" | grep -v grep | tee ~/module02/evidence/services/process_context.txt
```

Function of the Commands:
- View active listening ports and processes that are likely relevant to the lab.
- Correlate services with the host’s internal process context.

Expected Output:
- ss displays active listening ports.
- process_context.txt contains processes relevant to the lab services.

Meaning of the Output: Service exposure should not be interpreted merely as a list of open ports; the process context provides operational meaning behind those ports.
<img width="2862" height="932" alt="image" src="https://github.com/user-attachments/assets/646ed946-683e-4e7a-a507-c6c6e0cc7a3f" />

**Phase 2.2. Validate Local Targets from Kali as an Assessment Workstation**
```
~/lab-targets-info
nmap -Pn -sV -p 3000,8081 127.0.0.1 -oN ~/module02/evidence/services/nmap_local_targets.txt
curl -I http://127.0.0.1:3000 | tee ~/module02/evidence/services/http_headers_local.txt
curl -I http://127.0.0.1:8081 | tee -a ~/module02/evidence/services/http_headers_local.txt
```

Function of the Commands:
- Validate how local targets appear from the workstation perspective.
- Read HTTP headers and identify services without moving into more aggressive testing.

Expected Output:
- Ports 3000 and 8081 are visible according to the target’s current state.
- HTTP headers from both local targets are displayed.

Meaning of the Output: We learn that safe and targeted service validation already provides significant value before moving into heavier web testing or more advanced assessments.

```
┌──(jccsah001012㉿kali)-[~/module02]
└─$ ~/lab-targets-info
Kali local targets:
  Juice Shop : http://127.0.0.1:3000
  DVWA       : http://127.0.0.1:8081
  Metasploit : msfconsole

┌──(jccsah001012㉿kali)-[~/module02]
└─$ nmap -Pn -sV -p 3000,8081 127.0.0.1 -oN ~/module02/evidence/services/nmap_local_targets.txt
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-30 04:56 +0000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.00022s latency).

PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
8081/tcp open  http    Apache httpd 2.4.25 ((Debian))
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.99%I=7%D=4/30%Time=69F2E0F7%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,126B3,"HTTP/1\.1\x20200\x20OK\r\nAccess-Control-Allow-Origin:\
SF:x20\*\r\nX-Content-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20SAME
SF:ORIGIN\r\nFeature-Policy:\x20payment\x20'self'\r\nX-Recruiting:\x20/#/j
SF:obs\r\nAccept-Ranges:\x20bytes\r\nCache-Control:\x20public,\x20max-age=
SF:0\r\nLast-Modified:\x20Mon,\x2020\x20Apr\x202026\x2013:15:48\x20GMT\r\n
SF:ETag:\x20W/\"124fa-19dab08698f\"\r\nContent-Type:\x20text/html;\x20char
SF:set=UTF-8\r\nContent-Length:\x2075002\r\nVary:\x20Accept-Encoding\r\nDa
SF:te:\x20Thu,\x2030\x20Apr\x202026\x2004:56:23\x20GMT\r\nConnection:\x20c
SF:lose\r\n\r\n<!--\n\x20\x20~\x20Copyright\x20\(c\)\x202014-2026\x20Bjoer
SF:n\x20Kimminich\x20&\x20the\x20OWASP\x20Juice\x20Shop\x20contributors\.\
SF:n\x20\x20~\x20SPDX-License-Identifier:\x20MIT\n\x20\x20-->\n\n<!doctype
SF:\x20html>\n<html\x20lang=\"en\"\x20data-beasties-container>\n<head>\n\x
SF:20\x20<meta\x20charset=\"utf-8\">\n\x20\x20<title>OWASP\x20Juice\x20Sho
SF:p</title>\n\x20\x20<meta\x20name=\"description\"\x20content=\"Probably\
SF:x20the\x20most\x20modern\x20and\x20sophisticated\x20insecure\x20web\x20
SF:application\">\n\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=d
SF:evice-width,\x20initial-scale=1\">\n\x20\x20<link\x20id=\"favicon\"\x20
SF:rel=\"icon\"\x20")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nCon
SF:nection:\x20close\r\n\r\n")%r(NCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Reques
SF:t\r\nConnection:\x20close\r\n\r\n")%r(HTTPOptions,EA,"HTTP/1\.1\x20204\
SF:x20No\x20Content\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-Contro
SF:l-Allow-Methods:\x20GET,HEAD,PUT,PATCH,POST,DELETE\r\nVary:\x20Access-C
SF:ontrol-Request-Headers\r\nContent-Length:\x200\r\nDate:\x20Thu,\x2030\x
SF:20Apr\x202026\x2004:56:23\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RT
SF:SPRequest,EA,"HTTP/1\.1\x20204\x20No\x20Content\r\nAccess-Control-Allow
SF:-Origin:\x20\*\r\nAccess-Control-Allow-Methods:\x20GET,HEAD,PUT,PATCH,P
SF:OST,DELETE\r\nVary:\x20Access-Control-Request-Headers\r\nContent-Length
SF::\x200\r\nDate:\x20Thu,\x2030\x20Apr\x202026\x2004:56:23\x20GMT\r\nConn
SF:ection:\x20close\r\n\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.62 seconds

┌──(jccsah001012㉿kali)-[~/module02]
└─$ curl -I http://127.0.0.1:3000 | tee ~/module02/evidence/services/http_headers_local.txt 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0  75002   0      0   0      0      0      0                              0
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'
X-Recruiting: /#/jobs
Accept-Ranges: bytes
Cache-Control: public, max-age=0
Last-Modified: Mon, 20 Apr 2026 13:15:48 GMT
ETag: W/"124fa-19dab08698f"
Content-Type: text/html; charset=UTF-8
Content-Length: 75002
Vary: Accept-Encoding
Date: Thu, 30 Apr 2026 04:56:53 GMT
Connection: keep-alive
Keep-Alive: timeout=5


┌──(jccsah001012㉿kali)-[~/module02]
└─$ curl -I http://127.0.0.1:8081 | tee -a ~/module02/evidence/services/http_headers_local.txt 
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
HTTP/1.1 302 Found
Date: Thu, 30 Apr 2026 04:57:07 GMT
Server: Apache/2.4.25 (Debian)
Set-Cookie: PHPSESSID=q9inlqvgjuj4ff8dftd6m0lkp5; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: PHPSESSID=q9inlqvgjuj4ff8dftd6m0lkp5; path=/
Set-Cookie: security=low
Location: login.php
Content-Type: text/html; charset=UTF-8
```

**Phase 2.3. Write a Service Context Note**
```
cat > ~/module02/evidence/services/service_context_notes.md <<'EOF' 
# Service Context Notes 
## Internal host view - ss shows listening ports and local process context 
## Validation view - nmap and curl show how the target looks from the workstation 
## Why both matter - internal state and external validation can tell different stories
EOF
cat ~/module02/evidence/services/service_context_notes.md
```

Function of the Command:
Encourage students to summarize the differences between internal and external perspectives in written form.

Expected Output:
`service_context_notes.md` is successfully created and available.

Meaning of the Output: Notes like this will help us when moving into process analysis and service relationship analysis in Module 03.
<img width="1427" height="450" alt="image" src="https://github.com/user-attachments/assets/f8acf66c-4040-4a96-bbb7-fc7b7b94d1df" />


**Module 02_Phase 3 — Sequential Connectivity Troubleshooting**
This phase builds a structured troubleshooting methodology. The goal is not to memorize many commands, but to understand when to use `ping`, `traceroute`, `mtr`, and `netcat`, as well as how to determine the failure domain from the available results.

**Phase 3.1. Build a Ping Matrix on Ubuntu**
```
mkdir -p ~/module02/evidence/troubleshooting
ls -d ~/module02/evidence/troubleshooting
ls -R ~/module02
printf "=== LOCALHOST ===\n" | tee ~/module02/evidence/troubleshooting/ping_matrix.txt
ping -c 4 127.0.0.1 | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
printf "\n=== EXTERNAL IP ===\n" | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
ping -c 4 8.8.8.8 | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
printf "\n=== DNS NAME ===\n" | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
ping -c 4 example.com | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
```

Function of the Commands:
Systematically distinguish between local stack conditions, outbound connectivity, and name resolution.

Expected Output:
Ping results for localhost, a public IP, and a domain name are stored in a single structured file.

Meaning of the Output: Effective troubleshooting must separate problem layers from the beginning, rather than mixing all symptoms together.

```
jccsah001012@localhost:~/module02$ mkdir -p ~/module02/evidence/troubleshooting
jccsah001012@localhost:~/module02$ ls -d ~/module02/evidence/troubleshooting
/home/jccsah001012/module02/evidence/troubleshooting
jccsah001012@localhost:~/module02$ ls -R ~/module02
/home/jccsah001012/module02:
architecture  baseline  evidence  monitoring  notes  reports  services  tmp  troubleshooting

/home/jccsah001012/module02/architecture:

/home/jccsah001012/module02/baseline:
dns_check.txt

/home/jccsah001012/module02/evidence:
baseline  troubleshooting

/home/jccsah001012/module02/evidence/baseline:
date.txt  hostname.txt  ip_addr.txt  route.txt  uname.txt  whoami.txt

/home/jccsah001012/module02/evidence/troubleshooting:

/home/jccsah001012/module02/monitoring:

/home/jccsah001012/module02/notes:

/home/jccsah001012/module02/reports:

/home/jccsah001012/module02/services:

/home/jccsah001012/module02/tmp:

/home/jccsah001012/module02/troubleshooting:
jccsah001012@localhost:~/module02$ printf "=== LOCALHOST ===\n" | tee ~/module02/evidence/troubleshooting/ping_matrix.txt
=== LOCALHOST ===
jccsah001012@localhost:~/module02$ ping -c 4 127.0.0.1 | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.034 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.051 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.062 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.065 ms

--- 127.0.0.1 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3081ms
rtt min/avg/max/mdev = 0.034/0.053/0.065/0.012 ms
jccsah001012@localhost:~/module02$ printf "\n=== EXTERNAL IP ===\n" | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt

=== EXTERNAL IP ===
jccsah001012@localhost:~/module02$ ping -c 4 8.8.8.8 | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=117 time=0.631 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=117 time=0.744 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=117 time=0.764 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=117 time=0.731 ms

--- 8.8.8.8 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3053ms
rtt min/avg/max/mdev = 0.631/0.717/0.764/0.051 ms
jccsah001012@localhost:~/module02$ printf "\n=== DNS NAME ===\n" | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt

=== DNS NAME ===
jccsah001012@localhost:~/module02$ ping -c 4 example.com | tee -a ~/module02/evidence/troubleshooting/ping_matrix.txt 
PING example.com (2606:4700:10::6814:179a) 56 data bytes
64 bytes from 2606:4700:10::6814:179a: icmp_seq=1 ttl=56 time=1.13 ms
64 bytes from 2606:4700:10::6814:179a: icmp_seq=2 ttl=56 time=1.19 ms
64 bytes from 2606:4700:10::6814:179a: icmp_seq=3 ttl=56 time=1.15 ms
64 bytes from 2606:4700:10::6814:179a: icmp_seq=4 ttl=56 time=1.05 ms

--- example.com ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3003ms
rtt min/avg/max/mdev = 1.054/1.129/1.186/0.048 ms
```

**Phase 3.2. View the Route with Traceroute and Hop Quality with MTR**
```
traceroute 8.8.8.8 | tee ~/module02/evidence/troubleshooting/traceroute_test.txt
mtr -rw -c 10 8.8.8.8 | tee ~/module02/evidence/troubleshooting/mtr_test.txt
```

unction of the Commands:
Map the outbound route from the host and identify possible packet loss or latency on each hop.

Expected Output:
- traceroute displays the network hops.
- mtr displays packet loss and latency summaries.

Meaning of the Output: We learn that connectivity diagnosis does not stop at whether a ping succeeds or fails.
<img width="2732" height="866" alt="image" src="https://github.com/user-attachments/assets/bb0f5d17-da3b-4454-8b2a-39cb591a71ee" />

**Phase 3.3. Quickly Validate Ports with Netcat**
```
nc -vz 127.0.0.1 3000 | tee ~/module02/evidence/troubleshooting/nc_validation.txt
nc -vz 127.0.0.1 8081 | tee -a ~/module02/evidence/troubleshooting/nc_validation.txt
```

Function of the Commands:
Confirm whether the target ports are actually accepting TCP connections.

Expected Output:
Connection succeeded, refused, or timeout messages appear for each tested port.

Meaning of the Output: Netcat provides quick validation that is highly useful before using heavier tools.

<img width="1935" height="147" alt="image" src="https://github.com/user-attachments/assets/ba8cee54-d05f-4814-9716-6e68fdef30d5" />

**Phase 3.4. Write a Short Route Diagnosis**
```
cat > ~/module02/evidence/troubleshooting/route_diagnosis.md <<'EOF' 
# Route Diagnosis 
## Observations 
 - localhost reachability: 127.0.0.1 ping statistics with 0% packet loss, avg time 0.053 ms.
 - external IP reachability: 8.8.8.8 ping statistics with 0% packet loss, avg time 0.717 ms.
 - DNS name reachability: example.com ping statistics with 0% packet loss, avg ime 1.129 ms.
 - traceroute path to 8.8.8.8 with 8 hops success reach the destination.
 - mtr quality with 11 hops, 0% packet loss.
## Initial interpretation
 - determine whether the issue is local, routing, or name resolution related : There is no issue related to local connectivity, routing, or name resolution. All network layers are functioning optimally and no infrastructure issues were detected.
EOF
cat ~/module02/evidence/troubleshooting/route_diagnosis.md
```

Function of the Command: Transform command outputs into human-readable diagnostic notes.

Expected Output: `route_diagnosis.md` is available.
<img width="2859" height="364" alt="image" src="https://github.com/user-attachments/assets/2726cc25-8acc-4857-bf6d-340b94846d07" />
Meaning of the Output: In the real world, a value does not stop at the commands they run, but extends to the quality of their interpretation.


**Module 02_Phase 4 — Operational Monitoring and Packet View**
This phase uses Ubuntu as the analysis host. The focus is on reading operational signals from the system journal, authentication log samples, and packet captures, then connecting them as complementary context.

**Phase 4.1. Review the Journal and Authentication Log Sample on Ubuntu**
```
journalctl -n 50 --no-pager | tee ~/module02/evidence/monitoring/journal_review.txt
grep -i "failed\|accepted" ~/datasets/logs/linux/auth.log.sample | tee ~/module02/evidence/monitoring/auth_log_review.txt
```

Function of the Commands: Review recent system events and observe authentication patterns from the sample log.

Expected Output:
- A portion of the journal output is displayed.
- Lines containing failed and accepted entries from the authentication log sample are displayed.
```
~/module02/evidence/monitoring/journal_review.txt:
Apr 29 14:27:06 localhost systemd[255276]: Reached target default.target - Main User Target.
Apr 29 14:27:06 localhost systemd[255276]: Startup finished in 148ms.
Apr 29 14:32:49 localhost systemd[255276]: launchpadlib-cache-clean.service - Clean up old files in the Launchpadlib cache was skipped because of an unmet condition check (ConditionPathExists=/home/jccsah001012/.launchpadlib/api.launchpad.net/cache).
Apr 29 16:21:57 localhost sshd[255342]: Received disconnect from 180.252.49.211 port 19995:11: Normal Shutdown
Apr 29 16:21:57 localhost sshd[255342]: Disconnected from user jccsah001012 180.252.49.211 port 19995
Apr 30 00:04:17 localhost sshd[256705]: Received disconnect from 180.252.49.211 port 21130:11: Normal Shutdown
Apr 30 00:04:17 localhost sshd[256705]: Disconnected from user jccsah001012 180.252.49.211 port 21130
Apr 30 00:04:27 localhost systemd[255276]: Activating special unit exit.target...
Apr 30 00:04:27 localhost systemd[255276]: Stopped target default.target - Main User Target.
Apr 30 00:04:27 localhost systemd[255276]: Stopped target basic.target - Basic System.
Apr 30 00:04:27 localhost systemd[255276]: Stopped target paths.target - Paths.
Apr 30 00:04:27 localhost systemd[255276]: Stopped target sockets.target - Sockets.
Apr 30 00:04:27 localhost systemd[255276]: Stopped target timers.target - Timers.
Apr 30 00:04:27 localhost systemd[255276]: Stopped launchpadlib-cache-clean.timer - Clean up old files in the Launchpadlib cache.
Apr 30 00:04:27 localhost systemd[255276]: Closed dbus.socket - D-Bus User Message Bus Socket.
Apr 30 00:04:27 localhost systemd[255276]: Closed dirmngr.socket - GnuPG network certificate management daemon.
Apr 30 00:04:27 localhost systemd[255276]: Closed gpg-agent-browser.socket - GnuPG cryptographic agent and passphrase cache (access for web browsers).
Apr 30 00:04:27 localhost systemd[255276]: Closed gpg-agent-extra.socket - GnuPG cryptographic agent and passphrase cache (restricted).
Apr 30 00:04:27 localhost systemd[255276]: Stopping gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation)...
Apr 30 00:04:27 localhost systemd[255276]: Closed gpg-agent.socket - GnuPG cryptographic agent and passphrase cache.
Apr 30 00:04:27 localhost systemd[255276]: Closed keyboxd.socket - GnuPG public key management service.
Apr 30 00:04:27 localhost systemd[255276]: Closed pk-debconf-helper.socket - debconf communication socket.
Apr 30 00:04:27 localhost systemd[255276]: Closed snapd.session-agent.socket - REST API socket for snapd user session agent.
Apr 30 00:04:27 localhost systemd[255276]: Closed gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation).
Apr 30 00:04:27 localhost systemd[255276]: Removed slice app.slice - User Application Slice.
Apr 30 00:04:27 localhost systemd[255276]: Reached target shutdown.target - Shutdown.
Apr 30 00:04:27 localhost systemd[255276]: Finished systemd-exit.service - Exit the Session.
Apr 30 00:04:27 localhost systemd[255276]: Reached target exit.target - Exit the Session.
Apr 30 00:04:27 localhost (sd-pam)[255277]: pam_unix(systemd-user:session): session closed for user jccsah001012
Apr 30 10:06:35 localhost systemd[267714]: Queued start job for default target default.target.
Apr 30 10:06:35 localhost systemd[267714]: Created slice app.slice - User Application Slice.
Apr 30 10:06:35 localhost systemd[267714]: Started launchpadlib-cache-clean.timer - Clean up old files in the Launchpadlib cache.
Apr 30 10:06:35 localhost systemd[267714]: Reached target paths.target - Paths.
Apr 30 10:06:35 localhost systemd[267714]: Reached target timers.target - Timers.
Apr 30 10:06:35 localhost systemd[267714]: Starting dbus.socket - D-Bus User Message Bus Socket...
Apr 30 10:06:35 localhost systemd[267714]: Listening on dirmngr.socket - GnuPG network certificate management daemon.
Apr 30 10:06:35 localhost systemd[267714]: Listening on gpg-agent-browser.socket - GnuPG cryptographic agent and passphrase cache (access for web browsers).
Apr 30 10:06:35 localhost systemd[267714]: Listening on gpg-agent-extra.socket - GnuPG cryptographic agent and passphrase cache (restricted).
Apr 30 10:06:35 localhost systemd[267714]: Starting gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation)...
Apr 30 10:06:35 localhost systemd[267714]: Listening on gpg-agent.socket - GnuPG cryptographic agent and passphrase cache.
Apr 30 10:06:35 localhost systemd[267714]: Listening on keyboxd.socket - GnuPG public key management service.
Apr 30 10:06:35 localhost systemd[267714]: Listening on pk-debconf-helper.socket - debconf communication socket.
Apr 30 10:06:35 localhost systemd[267714]: Listening on snapd.session-agent.socket - REST API socket for snapd user session agent.
Apr 30 10:06:35 localhost systemd[267714]: Listening on dbus.socket - D-Bus User Message Bus Socket.
Apr 30 10:06:35 localhost systemd[267714]: Listening on gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation).
Apr 30 10:06:35 localhost systemd[267714]: Reached target sockets.target - Sockets.
Apr 30 10:06:35 localhost systemd[267714]: Reached target basic.target - Basic System.
Apr 30 10:06:35 localhost systemd[267714]: Reached target default.target - Main User Target.
Apr 30 10:06:35 localhost systemd[267714]: Startup finished in 129ms.
Apr 30 10:11:49 localhost systemd[267714]: launchpadlib-cache-clean.service - Clean up old files in the Launchpadlib cache was skipped because of an unmet condition check (ConditionPathExists=/home/jccsah001012/.launchpadlib/api.launchpad.net/cache).

~/module02/evidence/monitoring/auth_log_review.txt:
Apr 21 09:00:01 labhost sshd[1201]: Failed password for invalid user admin from 10.10.10.50 port 53122 ssh2
Apr 21 09:00:05 labhost sshd[1208]: Failed password for invalid user test from 10.10.10.50 port 53130 ssh2
Apr 21 09:00:11 labhost sshd[1215]: Failed password for jccsah001001 from 10.10.10.50 port 53144 ssh2
Apr 21 09:00:18 labhost sshd[1221]: Accepted password for jccsah001001 from 10.10.10.50 port 53155 ssh2
```
Meaning of the Output: Basic monitoring is not just about collecting logs, but about identifying relevant behavioral patterns within them.

**Phase 4.2. Review a PCAP Sample with Tshark on Ubuntu**
```
tshark -r ~/datasets/pcap/localhost_web_lab.pcap | head -n 30 | tee ~/module02/evidence/monitoring/pcap_summary.txt
```

Function of the Command:
- Review sample traffic without performing a live packet capture.
- Correlate local target activity with more concise packet-level evidence.

Expected Output: A packet summary is displayed in `pcap_summary.txt`.
<img width="2869" height="1155" alt="image" src="https://github.com/user-attachments/assets/fa487d74-8c53-4402-9d6b-0ad1d6d471bc" />
Meaning of the Output: We begin to understand that network observations can come from both logs and packet summaries.

**Phase 4.3. Create Packet Notes**
```
cat > ~/module02/evidence/monitoring/packet_notes.md <<'EOF' 
# Packet Notes
 - PCAP sample is used as a safe evidence source
 - Focus on destination port, protocol flow, and repeated request pattern
 - Packet view complements service and log view
EOF
cat ~/module02/evidence/monitoring/packet_notes.md
```

Function of the Command: Make packet analysis part of the reasoning process, rather than treating it as technical output that is quickly ignored.

Expected Output of `packet_notes.md` is available.
```
# Packet Notes
 - PCAP sample is used as a safe evidence source
 - Focus on destination port, protocol flow, and repeated request pattern
 - Packet view complements service and log view
```
Meaning of the Output: Small notes like this will be very helpful when we move into blue-team-oriented modules later on.


**Module 02_Phase 5 — Architecture Thinking and Cloud/Segmentation Notes**
This phase transforms network observations into architectural understanding. We begin documenting assets, trust boundaries, segmentation, and exposure notes. This is important so that networking does not stop at being just a collection of commands.

**Phase 5.1. Create a Simple Cloud Mapping Based on the Lab Environment**
```
cat > ~/module02/evidence/architecture/cloud_mapping.md <<'EOF' 
# Cloud and Virtual Network Mapping 
 
## Assets
 - Ubuntu Lab: analysis and evidence host
 - Kali Lab: validation workstation
 - Local targets: 127.0.0.1:3000 and 127.0.0.1:8081 
 
## Exposure
 - Student SSH access
 - Local web targets bound to localhost only 
 
## Operational note
 - Different hosts are used for different purposes
EOF
cat ~/module02/evidence/architecture/cloud_mapping.md
```

Function of the Command: Train us to map assets, exposure, and trust boundaries using concise and professional language.

Expected Output of `cloud_mapping.md` is available.
```
# Cloud and Virtual Network Mapping 
 
## Assets
 - Ubuntu Lab: analysis and evidence host
 - Kali Lab: validation workstation
 - Local targets: 127.0.0.1:3000 and 127.0.0.1:8081 
 
## Exposure
 - Student SSH access
 - Local web targets bound to localhost only 
 
## Operational note
 - Different hosts are used for different purposes
```
Meaning of the Output: A networking foundation should lead toward architecture thinking, not stop at the level of interfaces and ports.

**Phase 5.2. Write a Segmentation Note Based on the Real Lab Environment**
```
cat > ~/module02/evidence/architecture/network_segmentation_notes.md <<'EOF' 
# Network Segmentation Notes 
 
## Current state
 - Student accounts are isolated at home-directory level
 - Shared server resources still exist
 - Local targets are intentionally kept on localhost 
 
## Recommendation
 - Keep role separation between Ubuntu and Kali
 - Keep privileged actions under instructor control
 - Keep local targets away from public exposure
EOF
cat ~/module02/evidence/architecture/network_segmentation_notes.md
```
Function of the Command: Connect segmentation theory with the realities of the environment currently being used.

Expected Output of `network_segmentation_notes.md` is available.
```
# Network Segmentation Notes 
 
## Current state
 - Student accounts are isolated at home-directory level
 - Shared server resources still exist
 - Local targets are intentionally kept on localhost 
 
## Recommendation
 - Keep role separation between Ubuntu and Kali
 - Keep privileged actions under instructor control
 - Keep local targets away from public exposure
```
Meaning of the Output: We learn that a good architecture note must be based on real conditions, not generic statements.


**Module 02_Phase 6 — Network Case Assessment and Bridge to Module 03**
This closing phase brings all outputs together into a short assessment. We must be able to demonstrate that they are not only running tools, but also understanding the meaning of the results and how they connect to host analysis in the next module.

**Phase 6.1. Write a Network Case Assessment**
```
cat > ~/module02/reports/network_case_assessment.md <<'EOF' 
# Network Case Assessment 
 
## Summary 
Environment shows clear role separation between Ubuntu and Kali, with safe localhost targets 
for controlled validation. 
 
## Key observations
 - Baseline route and DNS available
 - Local targets respond as expected
 - Troubleshooting evidence collected
 - Logs and PCAP reviewed 
 
## Risks
 - Shared host means noisy activity can affect others
 - Service interpretation must always include host context 
 
## Recommendation
 - Keep evidence-first workflow
 - Maintain role separation
 - Connect network observations with host-level analysis in the next module
EOF
cat ~/module02/reports/network_case_assessment.md
```

Function of the Command: Conclude the lab with a deliverable that is more professional than simply having separate output files.

Expected Output of `network_case_assessment.md` is available.
```
# Network Case Assessment 
 
## Summary 
Environment shows clear role separation between Ubuntu and Kali, with safe localhost targets 
for controlled validation. 
 
## Key observations
 - Baseline route and DNS available
 - Local targets respond as expected
 - Troubleshooting evidence collected
 - Logs and PCAP reviewed 
 
## Risks
 - Shared host means noisy activity can affect others
 - Service interpretation must always include host context 
 
## Recommendation
 - Keep evidence-first workflow
 - Maintain role separation
 - Connect network observations with host-level analysis in the next module
```
Meaning of the Output: We begin practicing how to produce assessments that can be understood by both technical supervisors and other operators.

**Phase 6.2. Write a Final Summary that Bridges to Module 03**
```
cat > ~/module02/reports/final_summary.md <<'EOF' 
# Final Summary 
 
Modul 02 memperjelas bagaimana host, route, resolver, service exposure, troubleshooting, log, 
dan packet view saling berhubungan. 
 
Bridge to Modul 03:
 - service exposure will connect to process and port relationship
 - network observations will connect to host baseline and process state
 - evidence discipline will continue into logs, sessions, permissions, and automation
EOF
cat ~/module02/reports/final_summary.md
```

Function of the Command: Explicitly connect the results of this module to the focus of Module 03.

Expected Output of `final_summary.md` is available.
```
# Final Summary 
 
Modul 02 memperjelas bagaimana host, route, resolver, service exposure, troubleshooting, log, 
dan packet view saling berhubungan. 
 
Bridge to Modul 03:
 - service exposure will connect to process and port relationship
 - network observations will connect to host baseline and process state
 - evidence discipline will continue into logs, sessions, permissions, and automation
```
Meaning of the Output: We do not begin Module 03 from zero; they enter with a mature networking context already established.

**Module 02_Phase 7 — Closing**

Module 02 should leave a clear impact: We are able to read host baselines, validate services from two perspectives, perform structured troubleshooting, utilize logs and packet summaries, and translate technical findings into architecture notes and short assessments.

With these outcomes, Module 03 can immediately move deeper into file systems, process and port relationships, sessions, logs, and automation without needing to re-explain networking fundamentals from the beginning.
