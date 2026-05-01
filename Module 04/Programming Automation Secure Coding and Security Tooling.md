# Module 04 — Building Python Workflows for Analysis, Validation, and Security Operations

This module guide us to carry out practicum activities independently or under supervision on the Ubuntu Lab and Kali Lab. The focus of this module is not merely writing scripts, but building useful workflows: collecting data, validating targets, processing evidence, writing more secure code, and producing outputs that can be used in security operations.

**1. Lab Objectives**
- Build the mindset that scripting is used to accelerate technical work, not to add unnecessary complexity.
- Use the Ubuntu Lab as the primary development and analysis host for Python, parsing, log processing, secure coding, data handling, and report generation.
- Use the Kali Lab as the validation workstation to test automation results against services, endpoints, and local targets from an assessment perspective.
- Train students to write reasonably organized scripts: clear input, useful output, basic error handling, and properly stored evidence.
- Establish a direct foundation for cyber core, blue team, threat hunting, and red team modules in a balanced manner.

**2. Environtment Overview and Host Roles**
<img width="1443" height="961" alt="image" src="https://github.com/user-attachments/assets/3716254e-ddd0-4bf4-bae5-6f536b62a7cb" />

**3. Practical Workflow**

- Prepare the workspace and verify the Python environment on Ubuntu, as well as validation helpers on Kali.
- Review Python foundations that are genuinely useful for security operations: file I/O, parsing, structured data, and argument handling.
- Build simple automation for endpoint validation, data collection, and summarization.
- Apply a secure coding mindset to the scripts being written.
- Use internal datasets to create IOC parsers, hash reviews, and useful rule-based helpers.
- Validate automation results from the Kali side as an assessment workstation.
- Conclude the practicum with a clear assessment and a bridge to the next module.


## Module 04_Phase 1 — Preparing the Workspace and Python Environment

This phase ensures that we do not start from an empty editor without structure. All scripts, outputs, and notes must be stored properly because this module will generate more artifacts than previous modules.

**Phase 1.1. Create the Module 4 Workspace on Ubuntu**
```
mkdir -p ~/module04/evidence/{automation,foundations,secure_coding,analysis,baseline,notes,tmp}
mkdir -p ~/module04/reports
cd ~/module04
pwd
ls -la
ls -la ~/module04/evidence
```

Function of the command:
- Create a folder structure so that source code, evidence, and reports do not become mixed together.
- Prepare a workspace ready to be used throughout the module.

Expected output:
- The active path is ~/module04.
- The folders are available.
```
/home/jccsah001012/module04
/home/jccsah001012/module04:
total 16
drwxrwxr-x  4 jccsah001012 jccsah001012 4096 May  1 14:47 .
drwx------ 13 jccsah001012 jccsah001012 4096 May  1 14:41 ..
drwxrwxr-x  9 jccsah001012 jccsah001012 4096 May  1 14:46 evidence
drwxrwxr-x  2 jccsah001012 jccsah001012 4096 May  1 14:47 reports

/home/jccsah001012/module04/evidence:
total 36
drwxrwxr-x 9 jccsah001012 jccsah001012 4096 May  1 14:46 .
drwxrwxr-x 4 jccsah001012 jccsah001012 4096 May  1 14:47 ..
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 14:41 analysis
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 14:41 automation
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 14:53 baseline
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 15:22 foundations
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 14:41 notes
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 14:41 secure_coding
drwxrwxr-x 2 jccsah001012 jccsah001012 4096 May  1 15:22 tmp
```
Meaning of the output: Good automation requires organized file management from the beginning.

**Phase 1.2. Activate the Shared Environment and Save the Python Baseline**
```
source ~/activate-pylab
python --version | tee ~/module04/evidence/baseline/python_version.txt
which python | tee ~/module04/evidence/baseline/venv_paths.txt
which pip | tee -a ~/module04/evidence/baseline/venv_paths.txt
~/jccsah-env-info
```

Function of the command:
- Activate the Python environment prepared by the administrator.
- Record the active interpreter and pip paths.

Expected output:
- The Python version is displayed normally.
- The python and pip paths point to `/opt/jccsah/venvs/pylab`.
```
[+] pylab aktif
Python 3.12.3

Python 3.12.3

/opt/jccsah/venvs/pylab/bin/pip

=== JCCSAH Ubuntu Lab ===
LAB_ROOT=/opt/jccsah
Python venv=/opt/jccsah/venvs/pylab

python3         : /opt/jccsah/venvs/pylab/bin/python3
pip3            : /opt/jccsah/venvs/pylab/bin/pip3
nmap            : /usr/bin/nmap
ffuf            : /usr/bin/ffuf
gobuster        : /usr/bin/gobuster
feroxbuster     :
sqlmap          : /usr/bin/sqlmap
hydra           : /usr/bin/hydra
john            : /usr/sbin/john
hashcat         : /usr/bin/hashcat
yara            : /usr/bin/yara
tshark          : /usr/bin/tshark
amass           :
theHarvester    :
volatility3     :
```
Meaning of the output: We work within a consistent environment that is ready to be used across scripts.

**Phase 1.3. Inventory the Workspace and Resources to Be Used**
```
find ~/module04 -maxdepth 2 -type d | sort | tee ~/module04/evidence/baseline/module04_workspace_tree.txt
ls -la ~/datasets ~/starter ~/wordlists
```

Function of the command:
- Record the module workspace structure.
- Remind students that the primary resources still come from the prepared datasets, starter files, and wordlists.

Expected output:
- The module04_workspace_tree.txt file is available.
- The `datasets`, `starter`, and `wordlists` folders are accessible.
<img width="2391" height="541" alt="image" src="https://github.com/user-attachments/assets/4824c2d0-3326-4f11-aeb0-700cac7c3f9b" />

Meaning of the output:
This module does not start from zero; it is built on top of existing lab resources.


## Module 04_Phase 2 — Python Foundations Relevant to Security Operations

This phase does not discuss Python syntax in general. The focus is on Python components most commonly used when working with evidence, APIs, files, and structured data.

**Phase 2.1. File I/O and Simple Structured Data**
```
cat > ~/module04/evidence/tmp/json_yaml_demo.py <<'EOF' 
import json 
from pathlib import Path 
 
sample = { 
    'host': 'ubuntu-lab', 
    'service': 'local-target', 
    'ports': [3000, 8081], 
    'status': 'ready' 
} 
out = Path.home() / 'module04' / 'evidence' / 'foundations' / 'json_yaml_notes.txt' 
out.write_text(json.dumps(sample, indent=2)) 
print(out.read_text()) 
EOF
python3 ~/module04/evidence/tmp/json_yaml_demo.py
```

Function of the command:
- Demonstrate how to save structured data to a file and read it back.
- Familiarize students with using Path and JSON for cleaner output formatting.

Expected output:
- The `json_yaml_notes.txt` file is created and contains formatted JSON.
```
{
  "host": "ubuntu-lab",
  "service": "local-target",
  "ports": [
    3000,
    8081
  ],
  "status": "ready"
}
```
Meaning of the output: Structured output is far easier to reuse than random plain-text output.

**Phase 2.2. Simple Argument Handling with Argparse**
```
cat > ~/module04/evidence/tmp/argparse_demo.py <<'EOF' 
import argparse 
parser = argparse.ArgumentParser(description='Simple endpoint checker') 
parser.add_argument('--target', required=True) 
args = parser.parse_args() 
print(f'Target received: {args.target}') 
EOF
python3 ~/module04/evidence/tmp/argparse_demo.py --target http://127.0.0.1:3000 | tee ~/module04/evidence/foundations/argparse_demo.txt
```

Function of the command:
- Introduce clear and explicit script input handling.
- Encourage students to avoid hardcoded values as early as possible.

Expected output:
- The script displays the target received from the argument.
```
Target received: http://127.0.0.1:3000
```
Meaning of the output: A production-ready script should be easy to rerun with different parameters.

**Phase 2.3. Review File Input and Output in a Controlled Way**
```
cat > ~/module04/evidence/tmp/file_io_demo.py <<'EOF' 
from pathlib import Path 
src = Path.home() / 'datasets' / 'phishing' / 'sample_email_1.txt' 
out = Path.home() / 'module04' / 'evidence' / 'foundations' / 'file_io_review.txt' 
text = src.read_text(errors='ignore') 
out.write_text(text[:400]) 
print(out.read_text()) 
EOF
python3 ~/module04/evidence/tmp/file_io_demo.py
```

Function of the command:
- Familiarize students with reading dataset files from known locations.
- Demonstrate how partial data output can be stored as evidence.

Expected output:
- A portion of the phishing sample file content is displayed and saved.
```
From: it-support@redbridge-it-helpdesk.example
To: finance.staff@redbridge-logistics.example
Subject: URGENT - VPN Revalidation Required
```
Meaning of the output: Safe and organized file handling is the foundation for larger-scale parsing tasks.


## Module 04_Phase 3 — Endpoint Validation and Automation Workflow

This phase takes us from Python foundations into a more operational workflow: connecting to endpoints, reading status information, saving outputs, and evaluating results in a structured way. Ubuntu is used to build the scripts, while Kali is used to validate the results from an assessment perspective.

**Phase 3.1. Create an Endpoint Checker on Ubuntu**
```
cat > ~/module04/evidence/tmp/endpoint_checker.py <<'EOF'
import requests
from pathlib import Path

targets = ['http://127.0.0.1:3000', 'http://127.0.0.1:8081']
out = Path.home() / 'module04' / 'evidence' / 'automation' / 'requests_output.txt'
lines = []

for target in targets:
    try:
        r = requests.get(target, timeout=5)
        lines.append(f'{target} | status={r.status_code} | len={len(r.text)}')
    except Exception as e:
        lines.append(f'{target} | error={e}')

out.write_text('\n'.join(lines))
print(out.read_text())
EOF
python3 ~/module04/evidence/tmp/endpoint_checker.py
```

Function of the command:
- Build a script that performs endpoint validation and stores readable output.
- Introduce the use of timeouts and basic exception handling.

Expected output:
- Status codes or errors for each endpoint are displayed and saved.
```
http://127.0.0.1:3000 | status=200 | len=75002
http://127.0.0.1:8081 | status=200 | len=1523
```
Meaning of the output: Useful automation should produce results that can be reviewed later, not just temporary terminal output.

**Phase 3.2. Validate Endpoint Results from Kali**
```
curl -I http://127.0.0.1:3000 | tee ~/module04/evidence/automation/endpoint_validation.txt
curl -I http://127.0.0.1:8081 | tee -a ~/module04/evidence/automation/endpoint_validation.txt
nmap -Pn -sV -p 3000,8081 127.0.0.1 | tee -a ~/module04/evidence/automation/endpoint_validation.txt 
```

Function of the command:
- Compare Ubuntu script results with validation results from the Kali workstation.
- Train us to verify whether automation output aligns with relevant manual observations.

Expected output:
- HTTP headers and port status information are displayed from the Kali perspective.
```
http://127.0.0.1:3000 | endpoint_validation.txt:
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
Date: Fri, 01 May 2026 09:22:54 GMT
Connection: keep-alive
Keep-Alive: timeout=5

http://127.0.0.1:8081 | endpoint_validation.txt:
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
HTTP/1.1 302 Found
Date: Fri, 01 May 2026 09:23:05 GMT
Server: Apache/2.4.25 (Debian)
Set-Cookie: PHPSESSID=em97rvnurctcqcsf7u1mm30sh1; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: PHPSESSID=em97rvnurctcqcsf7u1mm30sh1; path=/
Set-Cookie: security=low
Location: login.php
Content-Type: text/html; charset=UTF-8

Starting Nmap 7.99 ( https://nmap.org ) at 2026-05-01 09:23 +0000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000043s latency).

PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
8081/tcp open  http    Apache httpd 2.4.25 ((Debian))
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.99%I=7%D=5/1%Time=69F47115%P=x86_64-pc-linux-gnu%r(Get
SF:Request,126B3,"HTTP/1\.1\x20200\x20OK\r\nAccess-Control-Allow-Origin:\x
SF:20\*\r\nX-Content-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20SAMEO
SF:RIGIN\r\nFeature-Policy:\x20payment\x20'self'\r\nX-Recruiting:\x20/#/jo
SF:bs\r\nAccept-Ranges:\x20bytes\r\nCache-Control:\x20public,\x20max-age=0
SF:\r\nLast-Modified:\x20Mon,\x2020\x20Apr\x202026\x2013:15:48\x20GMT\r\nE
SF:Tag:\x20W/\"124fa-19dab08698f\"\r\nContent-Type:\x20text/html;\x20chars
SF:et=UTF-8\r\nContent-Length:\x2075002\r\nVary:\x20Accept-Encoding\r\nDat
SF:e:\x20Fri,\x2001\x20May\x202026\x2009:23:33\x20GMT\r\nConnection:\x20cl
SF:ose\r\n\r\n<!--\n\x20\x20~\x20Copyright\x20\(c\)\x202014-2026\x20Bjoern
SF:\x20Kimminich\x20&\x20the\x20OWASP\x20Juice\x20Shop\x20contributors\.\n
SF:\x20\x20~\x20SPDX-License-Identifier:\x20MIT\n\x20\x20-->\n\n<!doctype\
SF:x20html>\n<html\x20lang=\"en\"\x20data-beasties-container>\n<head>\n\x2
SF:0\x20<meta\x20charset=\"utf-8\">\n\x20\x20<title>OWASP\x20Juice\x20Shop
SF:</title>\n\x20\x20<meta\x20name=\"description\"\x20content=\"Probably\x
SF:20the\x20most\x20modern\x20and\x20sophisticated\x20insecure\x20web\x20a
SF:pplication\">\n\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=de
SF:vice-width,\x20initial-scale=1\">\n\x20\x20<link\x20id=\"favicon\"\x20r
SF:el=\"icon\"\x20")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConn
SF:ection:\x20close\r\n\r\n")%r(NCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request
SF:\r\nConnection:\x20close\r\n\r\n")%r(HTTPOptions,EA,"HTTP/1\.1\x20204\x
SF:20No\x20Content\r\nAccess-Control-Allow-Origin:\x20\*\r\nAccess-Control
SF:-Allow-Methods:\x20GET,HEAD,PUT,PATCH,POST,DELETE\r\nVary:\x20Access-Co
SF:ntrol-Request-Headers\r\nContent-Length:\x200\r\nDate:\x20Fri,\x2001\x2
SF:0May\x202026\x2009:23:33\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RTS
SF:PRequest,EA,"HTTP/1\.1\x20204\x20No\x20Content\r\nAccess-Control-Allow-
SF:Origin:\x20\*\r\nAccess-Control-Allow-Methods:\x20GET,HEAD,PUT,PATCH,PO
SF:ST,DELETE\r\nVary:\x20Access-Control-Request-Headers\r\nContent-Length:
SF:\x200\r\nDate:\x20Fri,\x2001\x20May\x202026\x2009:23:33\x20GMT\r\nConne
SF:ction:\x20close\r\n\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.60 seconds
```
Meaning of the output: Good automation should still be validated from a different perspective.

**Phase 3.3. Create Automation Workflow Notes**
```
cat > ~/module04/evidence/automation/automation_notes.md <<'EOF' 
# Automation Notes
 - Ubuntu digunakan untuk menulis dan menjalankan script.
 - Kali digunakan untuk memvalidasi hasil dari sudut pandang workstation assessment.
 - Script harus menghasilkan output yang tersimpan, bukan hanya tampilan terminal.
EOF
```

Function of the command:
- Encourage us to document workflow patterns, not just save commands.

Expected output:
- The `automation_notes.md` file is available.
```
# Automation Notes
 - Ubuntu digunakan untuk menulis dan menjalankan script.
 - Kali digunakan untuk memvalidasi hasil dari sudut pandang workstation assessment.
 - Script harus menghasilkan output yang tersimpan, bukan hanya tampilan terminal.
```
Meaning of the output: Workflow notes make automation easier to explain and reproduce.


## Module 04_Phase 4 — Relevant and Practical Secure Coding

This phase does not turn the module into a full web programming class. The focus is on applying secure coding principles that are directly relevant to the scripts and workflows being built: input validation, safe queries, and error handling that does not expose excessive details.

**Phase 4.1. Basic Input Validation for Scripts**
```
cat > ~/module04/evidence/tmp/input_validation_demo.py <<'EOF'
from urllib.parse import urlparse

def validate_target(url: str) -> bool:
    parsed = urlparse(url)
    return parsed.scheme in {'http', 'https'} and bool(parsed.netloc)

samples = ['http://127.0.0.1:3000', '127.0.0.1:8081', 'ftp://example.com']

for s in samples:
    print(s, validate_target(s))
EOF
python3 ~/module04/evidence/tmp/input_validation_demo.py | tee ~/module04/evidence/secure_coding/input_validation_notes.txt
```

Function of the command:
- Demonstrate that input should not be accepted blindly without basic format validation.
- Encourage students to think about validation before executing core logic.

Expected output:
- Examples of valid and invalid input are displayed with True or False results.
```
http://127.0.0.1:3000 True
127.0.0.1:8081 False
ftp://example.com False
```
Meaning of the output: Secure coding begins with the habit of consciously validating input.

**Phase 4.2. Safe Queries with Parameterized Access**
```
cat > ~/module04/evidence/tmp/safe_query_demo.py <<'EOF' 
import sqlite3 
from pathlib import Path 
 
db = Path.home() / 'module04' / 'evidence' / 'tmp' / 'users.db' 
conn = sqlite3.connect(db) 
cur = conn.cursor() 
cur.execute('CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, username TEXT)') 
cur.execute('DELETE FROM users') 
cur.executemany('INSERT INTO users (username) VALUES (?)', [('alice',), ('bob',)]) 
user_id = '1' 
cur.execute('SELECT * FROM users WHERE id = ?', (user_id,)) 
print(cur.fetchall()) 
conn.commit() 
conn.close() 
EOF
python3 ~/module04/evidence/tmp/safe_query_demo.py | tee ~/module04/evidence/secure_coding/safe_query_demo.txt
```

Function of the command:
- Introduce parameterized queries as a fundamental best practice.
- Demonstrate that secure examples can be built without making the lab unnecessarily complex.

Expected output:
- The results of the safe query are displayed.

```
[(1, 'alice')]
```
Meaning of the output: Small habits such as parameterization are important foundations of secure coding.

**Phase 4.3. Useful but Controlled Error Handling**
```
cat > ~/module04/evidence/tmp/error_handling_demo.py <<'EOF' 
from pathlib import Path 
try: 
    data = Path.home() / 'datasets' / 'nonexistent' / 'missing.txt' 
    print(data.read_text()) 
except FileNotFoundError: 
    print('Input file not found. Review dataset path before rerunning the script.') 
EOF
python3 ~/module04/evidence/tmp/error_handling_demo.py | tee ~/module04/evidence/secure_coding/error_handling_notes.txt
```

Function of the command:
- Demonstrate that error messages should be helpful without becoming noisy or exposing unnecessary details.

Expected output:
- A clear error message is displayed when the file is not found.
```
Input file not found. Review dataset path before rerunning the script.
```
Meaning of the output:
A good script fails clearly, rather than silently or with excessive and unfocused verbosity.


## Module 04_Phase 5 — Threat-Aware Scripting and Basic Data Analysis

This phase uses internal datasets so we can begin to see how scripting supports security operations: reading simple IOCs, reviewing hashes, and performing rule-based helper analysis on available artifacts.

**Phase 5.1. Simple IOC Parser from Text Artifacts**
```
cat > ~/module04/evidence/tmp/ioc_parser.py <<'EOF' 
import re 
from pathlib import Path 
 
src = Path.home() / 'datasets' / 'phishing' / 'sample_email_1.txt' 
text = src.read_text(errors='ignore') 
emails = re.findall(r'[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+', text) 
words = [w for w in text.split() if 'vpn' in w.lower() or 'urgent' in w.lower()] 
print('emails=', emails) 
print('keywords=', words) 
EOF
python3 ~/module04/evidence/tmp/ioc_parser.py | tee ~/module04/evidence/analysis/ioc_parser_output.txt
```

Function of the command:
- Introduce lightweight IOC extraction from text artifacts.
- Demonstrate that small scripts can save time compared to manual review.

Expected output:
- A list of email addresses and relevant keywords is displayed.
```
emails= ['it-support@redbridge-it-helpdesk.example', 'finance.staff@redbridge-logistics.example']
keywords= ['URGENT', 'VPN']
```
Meaning of the output: Threat-aware scripting means building scripts that identify signals that truly matter.

**Phase 5.2. Review File Hashes and Integrity**
```
sha256sum ~/datasets/phishing/sample_email_1.txt | tee ~/module04/evidence/analysis/hash_review.txt
sha256sum ~/datasets/logs/linux/auth.log.sample | tee -a ~/module04/evidence/analysis/hash_review.txt
```

Function of the command:
- Familiarize students with using hashes as file identifiers and as the basis for evidence integrity.

Expected output:
- SHA256 hash values for the reviewed files are displayed.
```
710a3222cf86bb6aa98ad6d7bcabae120ac5c371006380b5167a2074ee5c6c83  /home/jccsah001012/datasets/phishing/sample_email_1.txt
392487dcdc5575c4924fd25963299edd6db49da85c6ba0d2a21e430689e2a365  /home/jccsah001012/datasets/logs/linux/auth.log.sample
```
Meaning of the output: Hashes are not just theoretical material; they are operational working tools.

**Phase 5.3. Simple Rule-Based Helper for Local Artifacts**
```
cat > ~/module04/evidence/tmp/basic_keywords.yar <<'EOF' 
rule basic_keywords { 
  strings: 
    $a = "urgent" nocase 
    $b = "verify" nocase 
    $c = "password" nocase 
  condition: 
    any of them 
} 
EOF
yara ~/module04/evidence/tmp/basic_keywords.yar ~/datasets/phishing/sample_email_1.txt | tee ~/module04/evidence/analysis/yara_helper_output.txt
```

Function of the command:
- Demonstrate how rule-based helpers can accelerate initial review activities.
- Connect this module to more advanced blue team analysis in future modules.

Expected output:
- YARA results are displayed if the targeted strings are found.
```
basic_keywords /home/jccsah001012/datasets/phishing/sample_email_1.txt
```
Meaning of the output: Rule-based thinking helps us understand the relationship between automation and a detection mindset.


## Module 04_Phase 6 — Mini Tool Building and Reusable Output

This phase summarizes the previous practices into a more complete mini tool. We are expected to produce a small tool that accepts input, processes data, stores output, and generates reusable results.

**Phase 6.1. Build a Mini Tool for Endpoint Summary and Evidence Collection**
```
cat > ~/module04/evidence/tmp/mini_tool.py <<'EOF'
import requests
from pathlib import Path

TARGETS = ['http://127.0.0.1:3000', 'http://127.0.0.1:8081']
out = Path.home() / 'module04' / 'reports' / 'mini_tool_report.txt'
results = []

for t in TARGETS:
    try:
        r = requests.get(t, timeout=5)
        results.append(f'{t}\tstatus={r.status_code}\tlen={len(r.text)}')
    except Exception as e:
        results.append(f'{t}\terror={e}')

# Memastikan folder reports sudah ada sebelum menulis file
out.parent.mkdir(parents=True, exist_ok=True)
out.write_text('\n'.join(results))
print(out.read_text())
EOF
python3 ~/module04/evidence/tmp/mini_tool.py
```

Function of the command:
- Combine input handling, requests, error handling, and file output into a more complete mini tool.
- Demonstrate that a small and well-organized tool is more useful than a large script that is difficult to explain.

Expected output:
- The mini_tool_report.txt file is created and contains an endpoint summary.
```
http://127.0.0.1:3000   status=200      len=75002
http://127.0.0.1:8081   status=200      len=1523
```
Meaning of the output: We begin thinking like tool builders, not just command users.

**Phase 6.2. Validate the Mini Tool from Kali**
```
cat ~/module04/reports/mini_tool_report.txt
curl -I http://127.0.0.1:3000 | head -n 5
curl -I http://127.0.0.1:8081 | head -n 5
```

Function of the command:
- Compare the Ubuntu-generated report with quick validation results from Kali.
- Teach students that tool output should still be verified from the assessment perspective.

Expected output:
- The mini tool summary matches the header validation results from Kali.
```
mini_tool_report.txt:
http://127.0.0.1:3000   status=200      len=75002
http://127.0.0.1:8081   status=200      len=1523

  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0  75002   0      0   0      0      0      0                              0
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
X-Content-Type-Options: nosniff
X-Frame-Options: SAMEORIGIN
Feature-Policy: payment 'self'

  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
HTTP/1.1 302 Found
Date: Fri, 01 May 2026 12:03:00 GMT
Server: Apache/2.4.25 (Debian)
Set-Cookie: PHPSESSID=khoiujja5fll1mur3hmqm6i4s7; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
```
Meaning of the output: A good tool should produce output that is consistent with other technical observations.


## Module 04_Phase 7 — Final Assessment and Bridge to the Next Module

This closing phase is important so that we do not stop at merely collecting scripts. We must conclude the practicum with assessments, operational notes, and a clear bridge to the following modules.

**Phase 7.1. Write a Brief Automation Readiness Assessment**
```
cat > ~/module04/reports/automation_assessment.md <<'EOF' 
# Automation Assessment 
 
## Summary 
Ubuntu digunakan untuk membangun script, memproses evidence, dan menghasilkan output 
terstruktur. Kali digunakan untuk memvalidasi service dan endpoint dari perspektif workstation 
assessment. 
 
## Key observations
 - Shared Python environment ready
 - Endpoint automation completed
 - Basic secure coding patterns applied
 - IOC helper and hash review completed
 - Mini tool produced reusable report output 
 
## Operational takeaway 
Automation harus jelas tujuannya, rapi outputnya, dan bisa divalidasi ulang. Script yang 
sederhana tetapi terstruktur lebih bernilai daripada script yang besar namun sulit dijelaskan. 
EOF
cat ~/module04/reports/automation_assessment.md
```

Function of the command:
- Build the habit of concluding modules with assessments that can be read professionally.

Expected output:
- The `automation_assessment.md` file is available.
```
# Automation Assessment 
 
## Summary 
Ubuntu digunakan untuk membangun script, memproses evidence, dan menghasilkan output 
terstruktur. Kali digunakan untuk memvalidasi service dan endpoint dari perspektif workstation 
assessment. 
 
## Key observations
 - Shared Python environment ready
 - Endpoint automation completed
 - Basic secure coding patterns applied
 - IOC helper and hash review completed
 - Mini tool produced reusable report output 
 
## Operational takeaway 
Automation harus jelas tujuannya, rapi outputnya, dan bisa divalidasi ulang. Script yang 
sederhana tetapi terstruktur lebih bernilai daripada script yang besar namun sulit dijelaskan.
```
Meaning of the output: In real-world work, the quality of the assessment is just as important as the quality of the code.

**Phase 7.2. Create Bridge Notes for the Next Module**
```
at > ~/module04/reports/module04_bridge_notes.md <<'EOF' 
# Module 04 Bridge Notes 
 
## What carries forward
 - Structured output matters
 - Validation from Kali remains important
 - Secure coding starts from small habits
 - Automation should work with datasets and local targets safely 
 
## Bridge to next modules
 - Modul 05 akan memakai reasoning dan output discipline ini untuk cyber core labs
 - Modul 06 dan 07 akan memakai automation mindset untuk analysis, validation, dan assessment 
yang lebih serius 
EOF
cat ~/module04/reports/module04_bridge_notes.md
```

Function of the command: Reinforce continuity between modules so students see their learning path as a connected progression.
  
Expected output: The `module04_bridge_notes.md` file is available.
```
# Module 04 Bridge Notes 
 
## What carries forward
 - Structured output matters
 - Validation from Kali remains important
 - Secure coding starts from small habits
 - Automation should work with datasets and local targets safely 
 
## Bridge to next modules
 - Modul 05 akan memakai reasoning dan output discipline ini untuk cyber core labs
 - Modul 06 dan 07 akan memakai automation mindset untuk analysis, validation, dan assessment 
yang lebih serius
```
Meaning of the output: Modules do not stand alone; the workflows developed here will be reused in future modules.


## Module 04_Phase 8 — Closing

Module 04 is intended to make us comfortable using Python and automation as real operational tools. The focus is not on becoming a generic programmer, but on becoming a security practitioner who can collect data, process it, store organized output, validate results from the appropriate host, and write more secure scripts. With this foundation, the following modules can move further into cyber core concepts, secure coding, blue team analysis, threat hunting, and selected offensive validation without losing operational discipline.
