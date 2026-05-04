# Module 05 — Security Principles, Threats, IAM, Cryptography, Web Security, and Incident Thinking
The focus of this module is to transform core cybersecurity concepts into actionable workflows: identifying risks from assets and services, linking controls to the attack surface, validating trust and access, reviewing cryptographic evidence, examining web exposure, and concluding with a relevant incident-oriented assessment.

## 1. Lab Objectives
- Understand cybersecurity as a combination of risk, control, architecture, identity, cryptography, application security, and incident response—not merely a collection of tools.
- Use the Ubuntu Lab as the primary analysis host to review evidence, validate basic controls, check hashes and certificates, write notes, and produce assessments.
- Use the Kali Lab as a validation workstation to examine service exposure, application headers, local target behavior, and an attacker-aware perspective within a controlled scope.
- Train students to evaluate an environment from a control effectiveness perspective: what is protected, what is exposed, what is lacking, and what should be prioritized.
- Establish a direct bridge to defensive security, blue team operations, incident response, and offensive validation in subsequent modules.

## 2. Environtment Roles
<img width="1421" height="830" alt="image" src="https://github.com/user-attachments/assets/ddfd32c9-0224-4f63-9634-a5987f2034c4" />

## 3. Practical Workflow
- Prepare the workspace and establish a security reasoning baseline on both Ubuntu and Kali.
- Create a mapping of assets, trust boundaries, CIA (Confidentiality, Integrity, Availability), and relevant controls within the lab environment.
- Review the threat landscape and attack surface using internal datasets and local targets.
- Examine identity context, sessions, and least privilege within the existing lab environment.
- Review hashes, certificates, and basic cryptographic evidence that are practical and operational.
- Validate web application behavior, security headers, and application exposure from both Ubuntu and Kali according to their respective roles.
- Develop an incident-oriented case assessment that serves as a bridge to blue team and red team modules.

## Module 05_Phase 1 — Preparing the Workspace and Security Reasoning Baseline
This phase ensures that we do not jump straight into tools without a proper framework. Before analyzing threats and controls, students must understand what assets are being used, which host serves as the analysis environment, which acts as the validation environment, and what exactly needs to be protected.

**Phase 1.1. Create the Module 5 Working Directory on Ubuntu**
```
mkdir -p ~/module05/{evidence,reports}
mkdir -p ~/module05/evidence/{notes,principles,threats,iam,crypto,web,incident,tmp}
cd ~/module05
pwd
ls -la ~/module05
ls -la ~/module05/evidence
```

Function of the command:
- Create a clear folder structure to separate different practicum domains.
- Prepare locations for evidence and reports from the beginning.

Expected output:
- The active path is set to `~/module05`.
- The folders `evidence`, `reports`, `notes`, `principles`, `threats`, `iam`, `crypto`, `web`, `incident`, and `tmp` are available.
<img width="1768" height="839" alt="image" src="https://github.com/user-attachments/assets/a5758009-2cfb-4931-811e-45ad6c303765" />


**Phase 1.2. Write a Short Asset and Control Map**
```
cat > ~/module05/evidence/principles/asset_control_map.md <<'EOF' 
# Asset and Control Map 

## Assets
 - Ubuntu Lab: primary analysis host
 - Kali Lab: validation workstation
 - Local targets: 127.0.0.1:3000 and 127.0.0.1:8081
 - Datasets: logs, pcap, phishing, osint, starter 

## High-value areas
 - Student credentials
 - Shared venv and tooling
 - Local target access path
 - Evidence and reports 

## Existing controls
 - Non-sudo student account
 - Localhost binding for web targets
 - Shared datasets in controlled location
EOF
cat ~/module05/evidence/principles/asset_control_map.md
```
Function of the command:
- Train students to map assets and controls before analyzing threats.
- Connect the lab environment with a more formal security mindset.

Expected output:
- The `asset_control_map.md` file is available and contains assets, high-value areas, and existing controls.
<img width="1478" height="677" alt="image" src="https://github.com/user-attachments/assets/7d0efcb4-c7d0-48e4-9b8d-d30eb03d24f0" />


**Phase 1.3. Document CIA and Defense-in-Depth in the Lab Context**
```
cat > ~/module05/evidence/principles/cia_notes.md <<'EOF' 
# CIA Notes
 - Confidentiality: student home directory, credentials, and sensitive datasets should not be 
exposed broadly.
 - Integrity: evidence files, logs, and reports should remain unchanged unless intentionally 
updated.
 - Availability: shared lab resources must remain usable for all students. 
EOF
cat ~/module05/evidence/principles/cia_notes.md
```
Function of the command:
- Translate CIA and defense-in-depth principles into the actual lab context used by students.

Expected output:
- The `cia_notes.md`is created and available.
```
# CIA Notes
 - Confidentiality: student home directory, credentials, and sensitive datasets should not be 
exposed broadly.
 - Integrity: evidence files, logs, and reports should remain unchanged unless intentionally 
updated.
 - Availability: shared lab resources must remain usable for all students.
```
- The `defense_in_depth_notes.md` is created and available.
```
# Defense in Depth Notes
 - Account separation reduces blast radius.
 - Localhost-only targets reduce exposure.
 - Shared venv improves consistency.
 - Dataset-driven practice reduces unnecessary live probing.
```


## Module 05_Phase 2 — Threat Landscape, Attack Surface, and Risk Context
This phase encourages us to view threats as a relationship between assets, weaknesses, and consequences. The focus is not on listing scary attacks, but on understanding why a threat is relevant in a specific environment.

**Phase 2.1. Review Phishing Artifacts and Initial Threat Indicators**
```
cat ~/datasets/phishing/sample_email_1.txt | tee ~/module05/evidence/threats/phishing_findings.txt
grep -RinE 'urgent|verify|vpn|password|login' ~/datasets/phishing ~/datasets/osint 2>/dev/null | tee ~/module05/evidence/threats/threat_review.txt
```
Function of the command:
- Read artifacts that resemble social engineering attempts and extract relevant early signals.
- Train us to identify threat-related keywords within real artifacts, not just theory.

Expected output:
- The sample email content is displayed.
- Relevant keywords such as urgent, verify, vpn, password, or login appear if present.
```
From: it-support@redbridge-it-helpdesk.example
To: finance.staff@redbridge-logistics.example
Subject: URGENT - VPN Revalidation Required

/home/jccsah001012/datasets/phishing/sample_email_1.txt:3:Subject: URGENT - VPN Revalidation Required
/home/jccsah001012/datasets/osint/company_profile.txt:4:- vpn.redbridge-logistics.example
```

**Phase 2.2. Write Attack Surface Notes for the Current Lab Environment**
```
cat > ~/module05/evidence/threats/attack_surface_notes.md <<'EOF' 
# Attack Surface Notes 
 
## Observable Surface
 - SSH access for student accounts
 - Local web targets on localhost
 - Shared datasets and reports
 - User-space tooling on Ubuntu and Kali 
 
## Key Risks
 - Credential misuse
 - Weak review of phishing-like content
 - Over-broad trust in local tools or data
 - Shared resource exhaustion on the server 
EOF
cat ~/module05/evidence/threats/attack_surface_notes.md
```

Function of the command:
- Convert technical observations into more operational risk notes.
- Train us to define the attack surface based on the actual environment.

Expected output:
- The file `attack_surface_notes.md` is created and available.
```
# Attack Surface Notes 
 
## Observable Surface
 - SSH access for student accounts
 - Local web targets on localhost
 - Shared datasets and reports
 - User-space tooling on Ubuntu and Kali 
 
## Key Risks
 - Credential misuse
 - Weak review of phishing-like content
 - Over-broad trust in local tools or data
 - Shared resource exhaustion on the server
```


## Module 05_Phase 3 — Identity, Session, and Least Privilege Thinking
Identity and access control are at the core of many modern incidents. This phase helps us understand that identity, sessions, groups, and privileges are not just administrative details, but key parts of the security boundary.

**Phase 3.1. Review Identity Context on Ubuntu**
```
whoami | tee ~/module05/evidence/iam/iam_context.txt
id | tee -a ~/module05/evidence/iam/iam_context.txt
w | tee ~/module05/evidence/iam/session_access_review.txt
who | tee -a ~/module05/evidence/iam/session_access_review.txt
```
Function of the command:
- Examine user context, group membership, and session information of the current account.
- Show that access review starts with understanding who is working and from where.

Expected output:
- Outputs from `whoami`, `id, groups`, `w`, and `who` are recorded.
<img width="1584" height="433" alt="image" src="https://github.com/user-attachments/assets/3c4ef8a3-8d6c-452f-a985-21393ec716b4" />

**Phase 3.2. Write Least Privilege Notes Based on the Lab Model**
```
cat > ~/module05/evidence/iam/least_privilege_notes.md <<'EOF' 
# Least Privilege Notes 
 - Student accounts are non-sudo and limited to user-space practice.
 - Privileged changes remain under instructor or administrator control.
 - Localhost-only targets reduce accidental exposure.
 - Separation between Ubuntu and Kali reduces role confusion and encourages controlled workflow.
EOF
cat ~/module05/evidence/iam/least_privilege_notes.md
```
Function of the command:
- Translate the concept of least privilege into the current lab model.
- Show that controls like non-sudo access are not just technical limitations, but part of security design.

Expected output:
- The file `least_privilege_notes.md` is created and available.
```
# Least Privilege Notes 
 - Student accounts are non-sudo and limited to user-space practice.
 - Privileged changes remain under instructor or administrator control.
 - Localhost-only targets reduce accidental exposure.
 - Separation between Ubuntu and Kali reduces role confusion and encourages controlled workflow.
```
Meaning of the output:
Least privilege is easier to understand when tied to real lab experience rather than abstract theory.


## Module 05_Phase 4 — Cryptography, Hash, and Certificate Review
This phase places cryptography in a real-world operational context: verifying file integrity, examining certificate fingerprints, and understanding what is actually proven by hash values or Transport Layer Security (TLS) evidence.

**Phase 4.1. Review Hashes of Key Artifacts**
```
sha256sum ~/datasets/phishing/sample_email_1.txt | tee ~/module05/evidence/crypto/hash_verification.txt
sha256sum ~/datasets/logs/linux/auth.log.sample | tee -a ~/module05/evidence/crypto/hash_verification.txt
sha256sum ~/datasets/pcap/localhost_web_lab.pcap | tee -a ~/module05/evidence/crypto/hash_verification.txt || true
```
Function of the command:
- Train us to use hashes for file identity and evidence integrity.
- Compare multiple file types using hashes, not just a single example.

Expected output:
- SHA256 values for the reviewed files are displayed.
```
710a3222cf86bb6aa98ad6d7bcabae120ac5c371006380b5167a2074ee5c6c83  /home/jccsah001012/datasets/phishing/sample_email_1.txt
392487dcdc5575c4924fd25963299edd6db49da85c6ba0d2a21e430689e2a365  /home/jccsah001012/datasets/logs/linux/auth.log.sample
65f77099517924c82d023fc3782e3c5bd7dc100ff3aef2654aca6f437cebd533  /home/jccsah001012/datasets/pcap/localhost_web_lab.pcap
```
Meaning of the output:
Hashing is not just a “cryptography theory” concept—it is a practical tool for ensuring evidence integrity.

**Phase 4.2. Review Certificate and TLS Metadata Using OpenSSL (from Kali)**
```
echo | openssl s_client -connect example.com:443 -servername example.com 2>/dev/null | openssl x509 -noout -subject -issuer -dates -fingerprint | tee ~/module05/evidence/crypto/certificate_review.txt
```
Function of the command:
- Extract subject, issuer, validity dates, and fingerprint from a certificate.
- Connect HTTPS usage with real trust chain and certificate metadata.

Expected output:
- Certificate subject, issuer, validity dates, and fingerprint are displayed.
<img width="2875" height="300" alt="image" src="https://github.com/user-attachments/assets/117bb034-c390-4f95-8329-fe8232b501de" />

**Phase 4.3. Document What Hash and Certificate Reviews Prove**
```
cat > ~/module05/evidence/crypto/openssl_notes.txt <<'EOF' 
# Crypto Notes
 - Hash proves file identity and supports integrity checking.
 - A certificate review shows trust metadata, issuer, validity period, and fingerprint.
 - HTTPS presence alone does not prove the whole application is secure.
EOF
cat ~/module05/evidence/crypto/openssl_notes.txt
```
Function of the command:
- Encourage students to write operational conclusions from cryptographic evidence outputs.

Expected output:
- The file openssl_notes.txt is created and available.
```
# Crypto Notes
 - Hash proves file identity and supports integrity checking.
 - A certificate review shows trust metadata, issuer, validity period, and fingerprint.
 - HTTPS presence alone does not prove the whole application is secure.
```


## Module 05_Phase 5 — Web & Application Security Validation
This phase uses local targets to help us understand that application security goes beyond OWASP terminology. The focus is on reading headers, routes, and control indicators from both the analysis host and the validation workstation.

**Phase 5.1. Validate Security Headers and Application Response from Kali**
```
curl -I http://127.0.0.1:3000 | tee ~/module05/evidence/web/http_security_headers.txt
curl -I http://127.0.0.1:8081 | tee -a ~/module05/evidence/web/http_security_headers.txt
whatweb http://127.0.0.1:3000 | tee ~/module05/evidence/web/local_target_review.txt || true
whatweb http://127.0.0.1:8081 | tee -a ~/module05/evidence/web/local_target_review.txt || true
```
Function of the command:
- Read HTTP headers and perform lightweight fingerprinting on local targets.
- Train us to observe whether an application response reflects certain security controls.

Expected output:
- HTTP headers are displayed.
```
http://127.0.0.1:3000 | http_security_headers.txt:
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
Date: Mon, 04 May 2026 06:12:36 GMT
Connection: keep-alive
Keep-Alive: timeout=5

http://127.0.0.1:8081 | http_security_headers.txt:
  % Total    % Received % Xferd  Average Speed  Time    Time    Time   Current
                                 Dload  Upload  Total   Spent   Left   Speed
  0      0   0      0   0      0      0      0                              0
HTTP/1.1 302 Found
Date: Mon, 04 May 2026 06:12:47 GMT
Server: Apache/2.4.25 (Debian)
Set-Cookie: PHPSESSID=m1qgq9sjob4l4vofhjhv1kpen4; path=/
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Set-Cookie: PHPSESSID=m1qgq9sjob4l4vofhjhv1kpen4; path=/
Set-Cookie: security=low
Location: login.php
Content-Type: text/html; charset=UTF-8
```
- WhatWeb or lightweight fingerprinting results appear (if available).
<img width="2865" height="404" alt="image" src="https://github.com/user-attachments/assets/96de2272-d786-41e0-a612-a351ce8192a2" />

Meaning of the output: Application security begins with systematic observation, not assumptions or immediate exploitation.

**Phase 5.2. Write AppSec Observations Based on Evidence**
```
cat > ~/module05/evidence/web/appsec_observations.md <<'EOF' 
# AppSec Observations 

## Observed points
 - Application responses are reachable on localhost-only targets.
 - Headers and fingerprinting provide clues about exposure and control visibility.
 - Presence of HTTP headers alone does not guarantee secure authorization, input handling, or secure design. 

## Immediate questions
 - What controls are visible?
 - What controls are not visible from headers alone?
 - Which parts require code review or deeper testing rather than assumptions? 
EOF
cat ~/module05/evidence/web/appsec_observations.md
```
Function of the command:
- Teach us to write balanced observations without jumping to conclusions about application security.

Expected output:
- The file `appsec_observations.md` is created and available.
<img width="1756" height="429" alt="image" src="https://github.com/user-attachments/assets/403ff1c3-6300-4398-90de-142f53b547e1" />

Meaning of the output: Maturity in application security means understanding the limits of observation and knowing when deeper analysis is required.


## Module 05_Phase 6 — Incident Thinking and Case Assessment
This phase concludes the module with an incident response mindset. We are expected to connect artifacts, timelines, and control gaps into a more complete assessment. The goal is to prepare a natural transition to blue team and incident response modules.

**Phase 6.1. Build a Simple Timeline from Available Artifacts**
```
cat > ~/module05/evidence/incident/incident_timeline.md <<'EOF' 
# Incident Timeline 
1. Suspicious phishing-style email observed in dataset. 
2. Access and identity context reviewed on host. 
3. Local target exposure validated from Kali. 
4. Logs and hashes reviewed to preserve evidence context. 
5. Control notes prepared for follow-up analysis.
EOF
cat ~/module05/evidence/incident/incident_timeline.md
```
Function of the command:
- Train us to clearly structure sequences of events or observations.
- Turn timelines into a practical working tool, not just a report section.

Expected output:
- The file `incident_timeline.md` is created and available.
<img width="1434" height="245" alt="image" src="https://github.com/user-attachments/assets/13aad79e-9194-4ba4-914f-b0d5c3802457" />

**Phase 6.2. Write Response Notes Based on Control and Risk**
```
cat > ~/module05/evidence/incident/response_notes.md <<'EOF' 
# Response Notes 

## What Should Happen First
 - Preserve evidence and notes
 - Confirm host and user context
 - Validate whether the observed issue is local, application-level, or identity-related 

## What Should Not Happen First
 - Blindly change configuration
 - Assume compromise without evidence
 - Overclaim security based on one header or one artifact 
EOF
cat ~/module05/evidence/incident/response_notes.md
```
Function of the command:
- Encourage us to think about response priorities, not just artifact collection.

Expected output:
- The file `response_notes.md` is created and available.
<img width="1423" height="429" alt="image" src="https://github.com/user-attachments/assets/b385c3db-1ee4-4d6a-ac1d-5b759f4e1c0e" />

**Phase 6.3. Write the Final Case Assessment**
```
cat > ~/module05/evidence/incident/final_case_assessment.md <<'EOF' 
# Final Case Assessment 
 
## Summary 
Environment shows reasonable separation of roles between Ubuntu and Kali, but control effectiveness still depends on disciplined use of identity, evidence handling, localhost-only targets, and validation workflow. 
 
## Key Control Themes
 - Non-sudo accounts reduce unnecessary privilege
 - Localhost binding reduces exposure
 - Evidence handling supports integrity and review
 - Security headers and certificates provide clues, not full assurance 
 
## Risk Themes
 - Credential misuse
 - Misreading application exposure
 - Weak evidence discipline
 - Overconfidence in partial signals 
EOF
cat ~/module05/evidence/incident/final_case_assessment.md
```
Function of the command:
- Transform all observations from this module into a comprehensive, readable assessment.

Expected output:
- The file `final_case_assessment.md` is created and available.
<img width="2865" height="644" alt="image" src="https://github.com/user-attachments/assets/ff149410-45da-44a4-8756-df7205d06fdc" />



## Module 05_Phase 7 — Final Outputs and Bridge to the Next Module
This closing phase ensures that we do not stop at a collection of evidence files. We must conclude the module with a usable assessment and bridge notes that explain how this workflow will carry forward.

**Phase 6.1. Write the Module 05 Assessment**
```
cat > ~/module05/reports/module05_assessment.md <<'EOF' 
# Module 05 Assessment 
 
## Summary 
Cybersecurity should be understood as the relationship between assets, controls, identity, application exposure, cryptographic evidence, and incident-oriented reasoning.
 
## Practical Takeaway
- Ubuntu remains the primary analysis host
- Kali remains the validation workstation
- Controls must be evaluated in context
- Evidence must be preserved and interpreted carefully 
EOF
cat ~/module05/reports/module05_assessment.md
```
Function of the command:
- Summarize the module outcomes into a reusable assessment.

Expected output:
- The file `module05_assessment.md` is created and available.
 ```
# Module 05 Assessment 
 
## Summary 
Cybersecurity should be understood as the relationship between assets, controls, identity, application exposure, cryptographic evidence, and incident-oriented reasoning.
 
## Practical Takeaway
- Ubuntu remains the primary analysis host
- Kali remains the validation workstation
- Controls must be evaluated in context
- Evidence must be preserved and interpreted carefully
 ```

**Phase 6.2. Create Bridge Notes to the Next Module**
```
cat > ~/module05/reports/module05_bridge_notes.md <<'EOF' 
# Module 05 Bridge Notes 

## What Carries Forward
 - Control thinking matters
 - Identity and trust boundaries matter
 - Evidence must be preserved before conclusions are made
 - Headers, hashes, certificates, and logs are signals, not final truth by themselves 

## Bridge to next modules
 - Modul 06 will operationalize this reasoning into blue team, SIEM, EDR, IDS/IPS, and forensics workflows
 - Modul 07 will use the same reasoning to understand offensive pathways, validation, and red team perspective in a controlled way 
EOF
cat ~/module05/reports/module05_bridge_notes.md
```
Function of the command:
- Reinforce continuity from cyber core concepts to defensive and offensive specialization.

Expected output:
- The file `module05_bridge_notes.md` is created and available.
```
# Module 05 Bridge Notes 

## What Carries Forward
 - Control thinking matters
 - Identity and trust boundaries matter
 - Evidence must be preserved before conclusions are made
 - Headers, hashes, certificates, and logs are signals, not final truth by themselves 

## Bridge to next modules
 - Modul 06 will operationalize this reasoning into blue team, SIEM, EDR, IDS/IPS, and forensics workflows
 - Modul 07 will use the same reasoning to understand offensive pathways, validation, and red team perspective in a controlled way
```
Meaning of the output: We recognize that this module is the core of security reasoning, not just a theoretical pause in the learning path.



## Module 05_Closing
Module 05 serves as the core of cybersecurity reasoning within the practicum series. We not only become familiar with terms such as CIA, least privilege, hash, TLS, OWASP, and incident response, but also learn to connect them to real assets, controls, identity, exposure, and evidence. With this foundation, the following modules can move more effectively into blue team practices, forensics, SOC operations, and red team activities in a more structured and purposeful way.
