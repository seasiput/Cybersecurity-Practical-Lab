This guide is intended for students to carry out practical exercises independently or with guidance. The module does not focus on overly basic exercises, but rather on developing a professional workflow: Understanding the host context, Analyzing services and networks, Managing evidence, Using the Linux CLI efficiently, Concluding the practice with notes that are ready to be used in subsequent modules.

### Module 01 — Foundations of Technical Work, the Internet, Linux CLI, and Security Awareness

**1. Lab Objectives**
• Understand computers and the internet as real working systems that can be proven through technical observation, not merely definitions.
• Develop organized work habits: a structured workspace, properly stored evidence, generated checksums, and analysis results concluded with a brief summary.
• Use Ubuntu Lab as the primary learning server for Linux CLI, parsing, file handling, permission review, and evidence processing.
• Use Kali Lab as a validation workstation for DNS lookup, service verification, header inspection, and safe enumeration within the localhost scope.
• Build a solid foundation required for Modules 2 and 3: network context, data filtering, service observation, and discipline in working from the terminal.

**2. Environtment Overview and Host Roles**
<img width="1797" height="914" alt="Image" src="https://github.com/user-attachments/assets/81f5347f-6140-48ad-ac28-a832b70e66b3" />


**3. Practical Workflow**
• Build the module workspace and establish the host context on Ubuntu and Kali.
• Validate the basic internet chain: domain names, IP addresses, HTTP headers, and localhost services.
• Apply proper evidence handling so that work results can be transferred, reviewed, and their integrity verified.
• Perform simple log filtering and parsing as a foundation for automation and analysis.
• Use the Linux CLI for navigation, metadata review, ownership, permissions, and searching for relevant files.
• Conduct an initial security awareness review based on the artifacts available in the lab.
• Conclude the practicum with a recap connecting Module 01 to Module 02 and Module 03.

**Module 01_Phase 1 — Establishing the Workspace and Host Context**
This phase is brief but essential. The objective is not to excessively repeat basic checks, but to establish a working baseline that will be used throughout the module and serve as a reference when comparing results from Ubuntu and Kali.

**Phase 1.1. Build the workspace on the Ubuntu Lab**

```
mkdir -p ~/module01/{notes,evidence,reports,tmp} 
cd ~/module01 
hostname | tee ~/module01/evidence/ubuntu_identity.txt 
hostname -I | tee -a ~/module01/evidence/ubuntu_identity.txt 
whoami | tee -a ~/module01/evidence/ubuntu_identity.txt
echo $SHELL | tee -a ~/module01/evidence/ubuntu_identity.txt 
pwd | tee -a ~/module01/evidence/ubuntu_identity.txt 
```

Function of the command :
• Create a clear workspace structure from the beginning.
• Record the Ubuntu host identity, active user, shell, and working path.

Expected Output of `ubuntu_identity.txt` :
```
localhost
139.162.57.122 172.17.0.1 2400:8901::2000:18ff:feb1:fd89
jccsah001012
/bin/bash
/home/jccsah001012/module01
```

**Phase 1.2. Record the Host Context on the Kali Lab**
```
mkdir -p ~/module01/{notes,evidence,reports,tmp} 
cd ~/module01 
hostname | tee ~/module01/evidence/kali_identity.txt 
hostname -I | tee -a ~/module01/evidence/kali_identity.txt 
whoami | tee -a ~/module01/evidence/kali_identity.txt 
echo $SHELL | tee -a ~/module01/evidence/kali_identity.txt 
pwd | tee -a ~/module01/evidence/kali_identity.txt
```
Function of the command :
• Store the host identity baseline from the assessment workstation side.
• Familiarize students with working consistently across two different hosts.

Expected Output of `kali_identity.txt `:
```
kali
139.162.57.242 172.17.0.1 2400:8901::2000:3fff:fe06:4369
jccsah001012
/bin/bash
/home/jccsah001012/module01
```

**Phase 1.3. Write a Brief Context Note**
```
cat > ~/module01/evidence/host_context_notes.md <<'EOF' 
# Host Context Notes 
## Ubuntu Lab - Primary learning server - Main place for evidence, file handling, parsing, and notes 
## Kali Lab - Validation workstation - Main place for DNS check, HTTP header check, and localhost assessment 
EOF 
cat ~/module01/evidence/host_context_notes.md
```
Function of the command:
• Encourage students to rewrite their understanding of each host’s role, rather than simply looking at the table and forgetting it.

Expected Output of `host_context_notes.md` : 
```
# Host Context Notes 
## Ubuntu Lab 
- Primary learning server 
- Main place for evidence, file handling, parsing, and notes 
## Kali Lab 
- Validation workstation 
- Main place for DNS check, HTTP header check, and localhost assessment
```


**Module 01_Phase 2 — Internet Basics, DNS, HTTP, dan Validasi Service **
This phase is not intended as an introduction to internet theory. The purpose is to demonstrate that domains, IP addresses, HTTP headers, and local services can actually be read and validated quickly from the appropriate workstation. In this module, Kali is used as the validation workstation, while Ubuntu serves as the place where evidence is collected in an organized manner.

**Phase 2.1. DNS Lookup and Name Resolution on Kali**
```
cd ~/module01 
printf "=== DNS LOOKUP ===\n" | tee ~/module01/evidence/dns_check.txt 
dig openai.com +short | tee -a ~/module01/evidence/dns_check.txt 
dig example.com +short | tee -a ~/module01/evidence/dns_check.txt 
getent hosts example.com | tee -a ~/module01/evidence/dns_check.txt
```
Function of the command:
• Demonstrate domain name resolution to IP addresses using more than one approach.
• Familiarize students with not relying on only one tool when validating DNS.

Expected Output of `dns_check.txt` :
• IP addresses returned by `dig `are displayed.
• `getent hosts` returns resolution results for the same hostname.

```
=== DNS LOOKUP ===
104.18.33.45
172.64.154.211
104.20.23.154
172.66.147.243
2606:4700:10::6814:179a example.com
2606:4700:10::ac42:93f3 example.com
```

**Phase 2.2. HTTP Header Inspection of External and Local Targets from Kali**
```
printf "=== HTTP HEADERS EXTERNAL ===\n" | tee ~/module01/evidence/http_headers.txt 
curl -I https://example.com | tee -a ~/module01/evidence/http_headers.txt 
printf "\n=== HTTP HEADERS LOCAL TARGETS ===\n" | tee -a ~/module01/evidence/http_headers.txt 
~/lab-targets-info | tee -a ~/module01/evidence/http_headers.txt 
curl -I http://127.0.0.1:3000 | tee -a ~/module01/evidence/http_headers.txt 
curl -I http://127.0.0.1:8081 | tee -a ~/module01/evidence/http_headers.txt
```

Function of the command:
• Read status codes, redirects, server responses, and header characteristics from both external and local targets.
• Compare different response patterns between common public services and lab targets.

Expected Output of `http_headers.txt` :
• HTTP headers such as HTTP/1.1 200, 301, or 302 are displayed.
• Local target addresses are clearly visible in the output.

```
=== HTTP HEADERS EXTERNAL ===
% Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
HTTP/2 200 
date: Wed, 29 Apr 2026 07:37:54 GMT
content-type: text/html
server: cloudflare
last-modified: Sat, 18 Apr 2026 00:51:00 GMT
allow: GET, HEAD
accept-ranges: bytes
age: 11399
cf-cache-status: HIT
cf-ray: 9f3ca4e22a31561c-SIN

=== HTTP HEADERS LOCAL TARGETS ===
Kali local targets:
  Juice Shop : http://127.0.0.1:3000
  DVWA       : http://127.0.0.1:8081
  Metasploit : msfconsole

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
Date: Wed, 29 Apr 2026 07:38:20 GMT
Connection: keep-alive
Keep-Alive: timeout=5

 % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
HTTP/1.1 302 Found
Date: Wed, 29 Apr 2026 07:38:27 GMT
Server: Apache/2.4.25 (Debian)
Set-Cookie: PHPSESSID=rerdvkfc0au0miidna0hrv9ij4; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: PHPSESSID=rerdvkfc0au0miidna0hrv9ij4; path=/
Set-Cookie: security=low
Location: login.php
Content-Type: text/html; charset=UTF-8
```

**Phase 2.3. Localhost Service Validation from the Assessment Side**
```
printf "=== LOCALHOST SERVICE VALIDATION ===\n" | tee ~/module01/evidence/localhost_service_validation.txt 
ss -tulpn | head -n 30 | tee -a ~/module01/evidence/localhost_service_validation.txt 
nmap -Pn -sV -p 22,3000,8081 127.0.0.1 | tee -a ~/module01/evidence/localhost_service_validation.txt 
```

Function of the command:
• Compare the internal host perspective with the assessment perspective.
• Introduce the idea that safe enumeration within localhost scope can still provide useful insights.

Expected Output of `localhost_service_validation.txt` :
• Ports 22, 3000, or 8081 may appear depending on the host condition.
• Nmap displays port states and service identification.

```
=== LOCALHOST SERVICE VALIDATION ===
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
udp   UNCONN 0      0          127.0.0.1:323        0.0.0.0:*          
udp   UNCONN 0      0              [::1]:323           [::]:*          
tcp   LISTEN 0      128          0.0.0.0:2222       0.0.0.0:*          
tcp   LISTEN 0      200        127.0.0.1:5432       0.0.0.0:*          
tcp   LISTEN 0      4096       127.0.0.1:43391      0.0.0.0:*          
tcp   LISTEN 0      128          0.0.0.0:22         0.0.0.0:*          
tcp   LISTEN 0      4096       127.0.0.1:3000       0.0.0.0:*          
tcp   LISTEN 0      4096       127.0.0.1:8081       0.0.0.0:*          
tcp   LISTEN 0      128             [::]:2222          [::]:*          
tcp   LISTEN 0      200            [::1]:5432          [::]:*          
tcp   LISTEN 0      128             [::]:22            [::]:* 

Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-29 09:08 +0000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000040s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 10.2p1 Debian 6 (protocol 2.0)
3000/tcp open  ppp?
8081/tcp open  http    Apache httpd 2.4.25 ((Debian))
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.99%I=7%D=4/29%Time=69F1CAA2%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,101B9,"HTTP/1\.1\x20200\x20OK\r\nAccess-Control-Allow-Origin:\
SF:x20\*\r\nX-Content-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20SAME
SF:ORIGIN\r\nFeature-Policy:\x20payment\x20'self'\r\nX-Recruiting:\x20/#/j
SF:obs\r\nAccept-Ranges:\x20bytes\r\nCache-Control:\x20public,\x20max-age=
SF:0\r\nLast-Modified:\x20Mon,\x2020\x20Apr\x202026\x2013:15:48\x20GMT\r\n
SF:ETag:\x20W/\"124fa-19dab08698f\"\r\nContent-Type:\x20text/html;\x20char
SF:set=UTF-8\r\nContent-Length:\x2075002\r\nVary:\x20Accept-Encoding\r\nDa
SF:te:\x20Wed,\x2029\x20Apr\x202026\x2009:08:50\x20GMT\r\nConnection:\x20c
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
SF:ontrol-Request-Headers\r\nContent-Length:\x200\r\nDate:\x20Wed,\x2029\x
SF:20Apr\x202026\x2009:08:50\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RT
SF:SPRequest,EA,"HTTP/1\.1\x20204\x20No\x20Content\r\nAccess-Control-Allow
SF:-Origin:\x20\*\r\nAccess-Control-Allow-Methods:\x20GET,HEAD,PUT,PATCH,P
SF:OST,DELETE\r\nVary:\x20Access-Control-Request-Headers\r\nContent-Length
SF::\x200\r\nDate:\x20Wed,\x2029\x20Apr\x202026\x2009:08:50\x20GMT\r\nConn
SF:ection:\x20close\r\n\r\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.58 seconds
```

**Module 01_Phase 3 — Evidence Handling, Archiving, and File Integrity **
In this phase, discipline is required, it's not only about following commands, but also being able to provide evidence of work or failing to articulate progress in advanced technical modules.

**Phase 3.1. Organize the Workspace and Inventory Notes on Ubuntu**
```
cd ~/module01 
mkdir -p evidence/raw evidence/processed reports/archive 
printf "# Module 01 Notes\n\n" > notes/session_notes.md 
printf "Analyst: %s\nDate: %s\n" "$USER" "$(date)" >> notes/session_notes.md 
find ~/module01 -maxdepth 2 -type d | sort | tee ~/module01/evidence/workspace_tree.txt 
find ~/module01/notes -maxdepth 2 -type f | sort | tee ~/module01/evidence/notes_inventory.txt
```

Function of the command :
• Organize the workspace into raw, processed, reports, and archive areas.
• Create a notes inventory so students know which files have already been created.

<img width="1626" height="79" alt="Image" src="https://github.com/user-attachments/assets/42e4a7b3-d375-4d1b-8bfa-091d4d5a67fb" />

Expected Output of `workspace_tree.txt` is available :
```
/home/jccsah001012/module01
/home/jccsah001012/module01/evidence
/home/jccsah001012/module01/evidence/processed
/home/jccsah001012/module01/evidence/raw
/home/jccsah001012/module01/notes
/home/jccsah001012/module01/reports
/home/jccsah001012/module01/reports/archive
/home/jccsah001012/module01/tmp
```

Expected Output of `notes_inventory.txt` is available :
`/home/jccsah001012/module01/notes/session_notes.md`

**Phase 3.2. Archive the Initial Evidence and Create a Checksum**
```
tar -czf ~/module01/reports/module01_workspace_initial.tar.gz ~/module01/notes ~/module01/evidence 
sha256sum ~/module01/reports/module01_workspace_initial.tar.gz | tee ~/module01/evidence/archive_sha256.txt 
ls -lh ~/module01/reports/module01_workspace_initial.tar.gz
```
Function of the command:
• Create an initial snapshot of the work results.
• Add a checksum as proof of archive integrity.

Expected Output :
• The tar.gz archive file is successfully created.
• A SHA256 checksum is available.

```
tar: Removing leading `/' from member names
tar: Removing leading `/' from hard link targets
9a0ffa72129a3f13060cd0a0648d09216b519611498308b1bc10a8bf6aff8b0b  /home/jccsah001012/module01/reports/module01_workspace_initial.tar.gz
-rw-rw-r-- 1 jccsah001012 jccsah001012 696 Apr 29 17:19 /home/jccsah001012/module01/reports/module01_workspace_initial.tar.gz
```

**Module 01_Phase 4 — Programming Logic, Filtering, and Data Summarization **
This phase serves as a bridge to scripting and operating systems modules. We are begin to see that technical logic is not only about writing code, but also about reading data, selecting important signals, and producing useful summaries.

**Phase 4.1. Build a Simple Log Artifact for Parsing Practice on Ubuntu**
```
cat > ~/module01/tmp/activity.log <<'EOF' 
INFO login user=alice src=10.10.10.5 
WARN failed_login user=bob src=10.10.10.7 
WARN failed_login user=bob src=10.10.10.7 
INFO file_upload user=alice path=/uploads/q1.xlsx 
WARN failed_login user=charlie src=10.10.10.9 
EOF
```

Function of the command :
• Provide a small yet realistic log sample for filtering and summarization practice.

Expected Output is small and controlled datasets are very useful for building the correct analytical mindset before moving on to larger datasets. The `activity.log` file is created in the tmp folder.
```
INFO login user=alice src=10.10.10.5
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
INFO file_upload user=alice path=/uploads/q1.xlsx
WARN failed_login user=charlie src=10.10.10.9
```

**Phase 4.2. Filter, Count, and Summarize Patterns Using the Shell**
```
grep "failed_login" ~/module01/tmp/activity.log | tee ~/module01/evidence/activity_filtering.txt 
awk '{print $3,$4}' ~/module01/tmp/activity.log | sort | uniq -c | tee -a ~/module01/evidence/activity_filtering.txt 
cat > ~/module01/tmp/check_failed_logins.sh <<'EOF' 
#!/usr/bin/env bash 
COUNT=$(grep -c "failed_login" ~/module01/tmp/activity.log) 
if [ "$COUNT" -ge 2 ]; then 
echo "Review needed: repeated failed_login events detected" 
else 
echo "No repeated failed_login pattern detected" 
fi 
EOF 
chmod +x ~/module01/tmp/check_failed_logins.sh ~/module01/tmp/check_failed_logins.sh | tee ~/module01/evidence/shell_logic_output.txt
```

Function of the command :
• Demonstrate that filtering, counting, and simple decision-making can be built using very simple shell scripting.
• Build the foundation for threat hunting, triage, and automation in future modules.

Expected Output :
• Failed_login events are displayed.
• A summary count result appears.
• The script returns a simple decision message.

```
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=bob src=10.10.10.7
WARN failed_login user=charlie src=10.10.10.9
      1 user=alice path=/uploads/q1.xlsx
      1 user=alice src=10.10.10.5
      2 user=bob src=10.10.10.7
      1 user=charlie src=10.10.10.9
Review needed: repeated failed_login events detected
```

**Phase 4.3. Create a Simple Parser with Python on Ubuntu**
```
cat > ~/module01/tmp/activity_parser.py <<'EOF'
from collections import Counter
from pathlib import Path

log = Path.home() / 'module01' / 'tmp' / 'activity.log'
users = Counter()
failed = 0
for line in log.read_text().splitlines():
    parts = line.split()
    for part in parts:
        if part.startswith('user='):
            users[part.split('=',1)[1]] += 1
    if 'failed_login' in line:
        failed += 1
print('failed_login_count=', failed)
print('user_activity=', dict(users))
EOF
```
Function of the command:
• Demonstrate that Python can be used to create more structured summaries from the same log data.
• Serve as a direct foundation for scripting and automation modules.

Expected output:
• The number of failed_login events and a summary of user activity are displayed.
```
failed_login_count= 3
user_activity= {'alice': 2, 'bob': 2, 'charlie': 1}
```
Meaning of the output:
Shell and Python are not positioned as competitors, but as two complementary tools.


**Module 01_Phase 5 — Linux CLI, Navigation, Metadata, and Permission Review **
This phase is more than just using cd and ls. The focus is on building the habit of reading paths, metadata, ownership, and permissions as part of real technical work.

**Phase 5.1. Navigation and Directory Structure Review on Ubuntu**
```
printf "=== HOME ===\n" | tee ~/module01/evidence/filesystem_navigation.txt 
pwd | tee -a ~/module01/evidence/filesystem_navigation.txt 
ls -la ~ | tee -a ~/module01/evidence/filesystem_navigation.txt 
printf "\n=== DATASETS ===\n" | tee -a ~/module01/evidence/filesystem_navigation.txt 
find -L ~/datasets -maxdepth 2 -type d | sort | tee -a ~/module01/evidence/filesystem_navigation.txt 
printf "\n=== STARTER ===\n" | tee -a ~/module01/evidence/filesystem_navigation.txt 
find -L ~/starter -maxdepth 2 -type f | sort | tee -a ~/module01/evidence/filesystem_navigation.txt
```

Function of the command:
• Familiarize students with reading workspace structures and available resources instead of working blindly.

Expected Output:
• Lists of the home, datasets, and starter directories are displayed.

```
=== HOME ===
/home/jccsah001012/module01
total 68
drwx------ 10 jccsah001012 jccsah001012 4096 Apr 29 14:28 .
drwxr-xr-x 16 root         root         4096 Apr 20 16:11 ..
lrwxrwxrwx  1 jccsah001012 jccsah001012   30 Apr 20 20:20 activate-pylab -> /opt/jccsah/bin/activate-pylab
-rw-------  1 jccsah001012 jccsah001012 9112 Apr 29 16:21 .bash_history
-rw-r--r--  1 jccsah001012 jccsah001012  220 Mar 31  2024 .bash_logout
-rw-r--r--  1 jccsah001012 jccsah001012 3826 Apr 20 20:20 .bashrc
drwx------  2 jccsah001012 jccsah001012 4096 Apr 20 17:59 .cache
lrwxrwxrwx  1 jccsah001012 jccsah001012   20 Apr 20 20:20 datasets -> /opt/jccsah/datasets
lrwxrwxrwx  1 jccsah001012 jccsah001012   31 Apr 20 20:20 jccsah-env-info -> /opt/jccsah/bin/jccsah-env-info
drwxrwxr-x  8 jccsah001012 jccsah001012 4096 Apr 20 21:11 lab501
drwxr-xr-x  2 jccsah001012 jccsah001012 4096 Apr 20 20:20 labs
lrwxrwxrwx  1 jccsah001012 jccsah001012   36 Apr 20 20:21 lab-targets-info -> /opt/jccsah/scripts/lab-targets-info
drwxrwxr-x  6 jccsah001012 jccsah001012 4096 Apr 29 11:57 lap-prep
drwxrwxr-x  6 jccsah001012 jccsah001012 4096 Apr 29 14:28 module01
drwxrwxr-x  6 jccsah001012 jccsah001012 4096 Apr 29 12:00 module501
drwxr-xr-x  2 jccsah001012 jccsah001012 4096 Apr 20 20:20 notes
-rw-r--r--  1 jccsah001012 jccsah001012  807 Mar 31  2024 .profile
drwxr-xr-x  2 jccsah001012 jccsah001012 4096 Apr 20 20:20 projects
lrwxrwxrwx  1 jccsah001012 jccsah001012   19 Apr 20 20:20 samples -> /opt/jccsah/samples
lrwxrwxrwx  1 jccsah001012 jccsah001012   19 Apr 20 20:21 starter -> /opt/jccsah/starter
lrwxrwxrwx  1 jccsah001012 jccsah001012   17 Apr 20 20:20 tools -> /opt/jccsah/tools
-rw-------  1 jccsah001012 jccsah001012  532 Apr 20 23:21 .viminfo
lrwxrwxrwx  1 jccsah001012 jccsah001012   21 Apr 20 20:20 wordlists -> /opt/jccsah/wordlists

=== DATASETS ===
/home/jccsah001012/datasets
/home/jccsah001012/datasets/forensics
/home/jccsah001012/datasets/logs
/home/jccsah001012/datasets/logs/apache
/home/jccsah001012/datasets/logs/linux
/home/jccsah001012/datasets/logs/network
/home/jccsah001012/datasets/logs/siem
/home/jccsah001012/datasets/logs/windows
/home/jccsah001012/datasets/osint
/home/jccsah001012/datasets/pcap
/home/jccsah001012/datasets/phishing

=== STARTER ===
/home/jccsah001012/starter/bash/log_parser.sh
/home/jccsah001012/starter/python/auth_log_monitor.py
/home/jccsah001012/starter/reports/report_template.md
/home/jccsah001012/starter/secure-coding/flask_safe_query_example.py
```

**Phase 5.2. Review File Metadata and Permissions**
```
mkdir -p ~/module01/filesystem/labdata 
cd ~/module01/filesystem/labdata 
touch note.txt report.txt script.sh 
chmod 600 note.txt 
chmod 640 report.txt 
chmod 750 script.sh 
ls -l | tee ~/module01/evidence/permission_review.txt 
stat note.txt report.txt script.sh | tee ~/module01/evidence/metadata_review.txt 
namei -l ~/module01/filesystem/labdata | tee -a ~/module01/evidence/metadata_review.txt
```

Function of the command:
• Demonstrate the differences between file permission modes and file metadata.
• Familiarize students with reading not only filenames, but also owners, modes, timestamps, and path permissions.

Expected Output of `permission_review.txt` :
```
total 0
-rw------- 1 jccsah001012 jccsah001012 0 Apr 29 18:46 note.txt
-rw-r----- 1 jccsah001012 jccsah001012 0 Apr 29 18:46 report.txt
-rwxr-x--- 1 jccsah001012 jccsah001012 0 Apr 29 18:46 script.sh
```

Expected Output of `metadata_review.txt` :
```
File: note.txt
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 8,0     Inode: 680185      Links: 1
Access: (0600/-rw-------)  Uid: ( 1011/jccsah001012)   Gid: ( 1011/jccsah001012)
Access: 2026-04-29 18:46:25.887932466 +0700
Modify: 2026-04-29 18:46:25.887932466 +0700
Change: 2026-04-29 18:46:33.158165757 +0700
 Birth: 2026-04-29 18:46:25.887932466 +0700
  File: report.txt
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 8,0     Inode: 680186      Links: 1
Access: (0640/-rw-r-----)  Uid: ( 1011/jccsah001012)   Gid: ( 1011/jccsah001012)
Access: 2026-04-29 18:46:25.887932466 +0700
Modify: 2026-04-29 18:46:25.887932466 +0700
Change: 2026-04-29 18:46:40.300394886 +0700
 Birth: 2026-04-29 18:46:25.887932466 +0700
  File: script.sh
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 8,0     Inode: 680187      Links: 1
Access: (0750/-rwxr-x---)  Uid: ( 1011/jccsah001012)   Gid: ( 1011/jccsah001012)
Access: 2026-04-29 18:46:25.887932466 +0700
Modify: 2026-04-29 18:46:25.887932466 +0700
Change: 2026-04-29 18:46:45.173551189 +0700
 Birth: 2026-04-29 18:46:25.887932466 +0700

f: /home/jccsah001012/module01/filesystem/labdata
drwxr-xr-x root         root         /
drwxr-xr-x root         root         home
drwx------ jccsah001012 jccsah001012 jccsah001012
drwxrwxr-x jccsah001012 jccsah001012 module01
drwxrwxr-x jccsah001012 jccsah001012 filesystem
drwxrwxr-x jccsah001012 jccsah001012 labdata
```

**Phase 5.3. Search for Files Potentially Relevant for Analysis**
```
find -L ~/datasets ~/samples -maxdepth 3 -type f \( -iname "*.log" -o -iname "*.json" -o -iname "*.pcap" -o -iname "*.csv" -o -iname "*.txt" \) | sort | tee ~/module01/evidence/risky_strings_findings.txt 
```

Function of the command:
• Train students to search for files based on practical value and evidence type, rather than simply browsing manually.

Expected Output of `risky_strings_findings.txt` :
```
/home/jccsah001012/datasets/forensics/registry_runkeys_sample.txt
/home/jccsah001012/datasets/logs/network/firewall_logs.txt
/home/jccsah001012/datasets/osint/company_profile.txt
/home/jccsah001012/datasets/pcap/localhost_web_lab.pcap
/home/jccsah001012/datasets/phishing/sample_email_1.txt
```
Meaning of the output:
This habit will become the foundation for log review, evidence triage, and case analysis.


**Module 01_Phase 6 — Security Awareness Based on Artifacts and Observation **
Security awareness in this module is designed to be practical and closely related to real-world work. We are asked to examine artifacts, identify signs of risk, and clearly document their findings, rather than simply memorizing security slogans.

**Phase 6.1. Review a Phishing Email Sample Artifact on Ubuntu**
```
find -L ~/datasets -type f | grep -i 'phish\|email' || true
cat ~/datasets/phishing/sample_email_1.txt | tee ~/module01/evidence/phishing_review.txt
```

Function of the command:
• Familiarize students with carefully reading messages and identifying indicators of manipulation, urgency, or suspicious requests.

Expected Output of `phishing_review.txt` :
```
/home/jccsah001012/datasets/phishing/sample_email_1.txt
From: it-support@redbridge-it-helpdesk.example
To: finance.staff@redbridge-logistics.example
Subject: URGENT - VPN Revalidation Required
```
Meaning of the output:
Good security awareness begins with the ability to read details, not from memorizing posters.

**Phase 6.2. Write Relevant Password Hygiene Notes**
```
cat > ~/module01/evidence/password_strength_notes.txt <<'EOF'
Password hygiene notes:
- Jangan gunakan password yang sama lintas sistem.
- Panjang password dan uniqueness lebih penting daripada variasi simbol yang dangkal.
- Credential lab tidak boleh dipakai di sistem lain.
- Password manager lebih baik daripada pola manual yang mudah ditebak.
EOF
cat ~/module01/evidence/password_strength_notes.txt
```

Function of the command:
• Encourage students to write concise and correct working principles.

Expected Output of `password_strength_notes.txt` file is available :
```
Password hygiene notes:
- Jangan gunakan password yang sama lintas sistem.
- Panjang password dan uniqueness lebih penting daripada variasi simbol yang dangkal.
- Credential lab tidak boleh dipakai di sistem lain.
- Password manager lebih baik daripada pola manual yang mudah ditebak.
```

**Phase 6.3. Search for Risky Strings in Available Samples**
```
grep -RinE 'password|secret|token|urgent|verify|login' ~/datasets ~/samples 2>/dev/null | tee -a ~/module01/evidence/risky_strings_findings.txt | head -n 50
```

Function of the command:
• Train students to identify strings that are commonly relevant during initial data and artifact reviews.

Expected Output of `risky_strings_findings.txt` are displayed : 
```
/home/jccsah001012/datasets/phishing/sample_email_1.txt:3:Subject: URGENT - VPN Revalidation Required
/home/jccsah001012/datasets/logs/linux/auth.log.sample:1:Apr 21 09:00:01 labhost sshd[1201]: Failed password for invalid user admin from 10.10.10.50 port 53122 ssh2
/home/jccsah001012/datasets/logs/linux/auth.log.sample:2:Apr 21 09:00:05 labhost sshd[1208]: Failed password for invalid user test from 10.10.10.50 port 53130 ssh2
/home/jccsah001012/datasets/logs/linux/auth.log.sample:3:Apr 21 09:00:11 labhost sshd[1215]: Failed password for jccsah001001 from 10.10.10.50 port 53144 ssh2
/home/jccsah001012/datasets/logs/linux/auth.log.sample:4:Apr 21 09:00:18 labhost sshd[1221]: Accepted password for jccsah001001 from 10.10.10.50 port 53155 ssh2
```

**Module 01_Phase 7 — Final Recap and Bridge to the Next Module **
This phase concludes the module with the same discipline used in professional work: 
work results are summarized, readiness gaps are noted, and the connection to the next module is clearly emphasized.

**Phase 7.1. Create a Final Readiness Checklist**
```
cat > ~/module01/reports/module01_readiness_checklist.txt <<'EOF'
[OK] Ubuntu context collected
[OK] Kali context collected
[OK] DNS and HTTP validation completed
[OK] Evidence archive created and hashed
[OK] Shell filtering completed
[OK] Python parser completed
[OK] Permission and metadata review completed
[OK] Security awareness evidence recorded
EOF
cat ~/module01/reports/module01_readiness_checklist.txt
```

Function of the command:
• Provide proof that all major phases have been completed.

Expected Output of `readiness_checklist.txt` is available :
```
[OK] Ubuntu context collected
[OK] Kali context collected
[OK] DNS and HTTP validation completed
[OK] Evidence archive created and hashed
[OK] Shell filtering completed
[OK] Python parser completed
[OK] Permission and metadata review completed
[OK] Security awareness evidence recorded
```

**Phase 7.2. Write Reflection Notes Connecting to Module 02 and Module 03**
```
cat > ~/module01/reports/reflection_notes.md <<'EOF'
# Reflection Notes

## What I learned
- Ubuntu and Kali have different roles
- Evidence handling matters from the first module
- DNS, headers, and localhost services can be validated quickly
- Shell and Python both help build technical reasoning
- Permission and metadata review are operationally useful

## Bridge to next modules
- Modul 02 will deepen service exposure, troubleshooting, and monitoring
- Modul 03 will deepen file system, process, logs, and automation
EOF
cat ~/module01/reports/reflection_notes.md
```

Function of the command:
• Conclude the module with a narrative that bridges current practices to the next modules.

Expected Output of `reflection_notes.md` is available in the reports folder :
```
# Reflection Notes

## What I learned
- Ubuntu and Kali have different roles
- Evidence handling matters from the first module
- DNS, headers, and localhost services can be validated quickly
- Shell and Python both help build technical reasoning
- Permission and metadata review are operationally useful

## Bridge to next modules
- Modul 02 will deepen service exposure, troubleshooting, and monitoring
- Modul 03 will deepen file system, process, logs, and automation
```

Meaning of the output:
We are positioned to move into the next modules with a more mature context, rather than starting from zero again.


**Module 01_Phase 8 — Closing**
This module establishes a proper and structured working foundation:
We know which host we are working on, understand how to validate services and the internet chain, are accustomed to preserving evidence, are capable of performing simple filtering, and are becoming comfortable reading metadata and permissions.
