# Module 06 — SOC, Log Analysis, SIEM Thinking, IDS/IPS Awareness, Forensics, and Incident-Oriented Defensive Workflow
Focus of this module is to build a practical defensive security workflow: collecting evidence, analyzing logs, correlating alerts with context, validating exposure, reviewing PCAP and forensic artifacts, and producing assessments relevant to security operations.

## 1. Lab Objectives
- Understand defensive security operations as a combination of monitoring, analysis, validation, triage, and response thinking.
- Use Ubuntu Lab as the primary analysis host for log review, evidence handling, YARA, timeline building, file-based forensics, and assessment writing.
- Use Kali Lab as a validation workstation for service visibility, packet summary inspection, web/app validation, and an analyst perspective that may require additional tools.
- Build the discipline of interpreting events—not just running commands: what happened, what are the signals, what is the context, and what is the priority.
- Establish a foundation for SOC workflows, SIEM/EDR thinking, IDS/IPS reasoning, DFIR basics, and incident handling.

## 2. Environtment Roles
<img width="1783" height="790" alt="image" src="https://github.com/user-attachments/assets/37d07ede-d6bd-419b-8654-14918a1997ec" />

## 3. Practical Workflow
- Prepare the workspace and define the monitoring scope along with available evidence sources.
- Perform structured triage on Linux logs, Apache logs, and other sample events.
- Review PCAP data and service exposure to correlate logs with network context.
- Apply IOC review, enrichment, and YARA-based helpers to relevant artifacts.
- Build a timeline and basic forensic notes from the available samples.
- Write an incident-oriented assessment and response notes that are professionally readable.
- Conclude the module with bridge notes toward deeper blue team specialization and the next offensive module.

## Module 06_Phase 1 — Preparing the Workspace and Monitoring Scope
This phase ensures that students understand where signals originate, what evidence is available, and what observation boundaries are realistic within a shared lab environment. The goal is not just to create folders, but to define a proper monitoring scope.

**Phase 1.1. Create the Module 6 Working Directory on Ubuntu**
```
mkdir -p ~/module06/{evidence,reports}
mkdir -p ~/module06/evidence/{notes,monitoring,logs,network,threatintel,forensics,incident,tmp}
cd ~/module06
pwd
ls -la
ls -la ~/module06/evidence
```
Command function:
- Establish a clear working structure to separate domains such as logs, network, threat intelligence, forensics, and incident notes.
Prepare locations for evidence and reports from the beginning.

Expected output:
- The active path is ~/module06.
- Folders such as `monitoring`, `logs`, `network`, `threatintel`, `forensics`, `incident`, and `reports` are available.
<img width="1152" height="649" alt="image" src="https://github.com/user-attachments/assets/5defe3b5-54ae-4b86-8c1f-2dd8cf87ff80" />

**Phase 1.2. Write Log Source Map and Detection Scope**
```
cat > ~/module06/evidence/monitoring/log_source_map.md <<'EOF' 
# Log Source Map 
 
## Sources Available
 - Linux auth log sample
 - Apache access log sample - Windows/SIEM sample
 - Localhost web lab PCAP - Phishing and OSINT sample 
 
## Monitoring Scope
 - User-space analysis only
 - Localhost validation only
 - Shared environment awareness 
EOF
cat ~/module06/evidence/monitoring/log_source_map.md

cat > ~/module06/evidence/monitoring/detection_scope_notes.md <<'EOF' 
# Detection Scope Notes 
 - Student observes existing evidence and safe local targets.
 - Student does not deploy live defensive controls to the whole host.
 - Focus is on reading signals, not forcing intrusive collection. 
EOF
cat ~/module06/evidence/monitoring/detection_scope_notes.md
```
Command function:
- Encourage us to understand from the start what evidence sources are actually available.
- Set expectations that monitoring in a shared lab differs from a production SOC environment.

Expected output:
- Files `log_source_map.md` and `detection_scope_notes.md` are created.
<img width="1709" height="1299" alt="image" src="https://github.com/user-attachments/assets/b4f05138-aa97-4528-9e24-6f1a86620a71" />

**Phase 1.3. Validate Relevant Service Context**
```
ss -tulpn | tee ~/module06/evidence/monitoring/service_context.txt
~/lab-targets-info | tee -a ~/module06/evidence/monitoring/service_context.txt 
```
Command function:
- Identify listening ports and relate them to local target context.
- Establish a service baseline before starting triage.

Expected output:
- Listening ports and local target information are displayed.
<img width="1831" height="611" alt="image" src="https://github.com/user-attachments/assets/f8516b01-5643-4e0b-894f-2b0b10f1932c" />


## Module 06_Phase 2 — Log Triage and SIEM Thinking
This phase teaches us that SIEM thinking is not about fancy dashboards, but about the ability to read events, identify patterns, correlate fields, and set priorities. Ubuntu is used as the primary analysis host, as this is where evidence is reviewed and summarized.

**Phase 2.1. Triage Linux Auth Log**
```
grep -i 'failed\|accepted' ~/datasets/logs/linux/auth.log.sample | tee ~/module06/evidence/logs/auth_triage.txt
awk '/Failed password/ {print $(NF-3), $(NF-1)}' ~/datasets/logs/linux/auth.log.sample | sort | uniq -c | tee -a ~/module06/evidence/logs/auth_
triage.txt
```
Command function:
- Identify failed and successful login attempts from the sample log.
- Summarize source IP patterns for initial triage.

Expected output:
- Lines showing failed and accepted logins are displayed.
- A summary of source IP addresses and their occurrence counts is available.
<img width="2865" height="354" alt="image" src="https://github.com/user-attachments/assets/e6992a27-bcde-4877-8a21-e6d76e2963c9" />

**Phase 2.2. Triage Apache Access Log**
```
cat ~/datasets/logs/apache/access.log.sample | tee ~/module06/evidence/logs/apache_triage.txt
grep -E 'script|passwd|download|admin' ~/datasets/logs/apache/access.log.sample | tee -a ~/module06/evidence/logs/apache_triage.txt || true
```
Command function:
- Review interesting requests from application logs.
- Correlate request paths with potentially higher-risk behavior.

Expected output:
- Access log entries are displayed.
- Requests targeting sensitive paths or suspicious payloads appear if present.
<img width="2826" height="253" alt="image" src="https://github.com/user-attachments/assets/cd6e5f7b-6e5d-468b-a3cd-2bb55f2abe23" />

**Phase 2.3. Review Windows/SIEM Event Sample and Create Correlation Notes**
```
cat ~/datasets/logs/windows/sysmon_sample.jsonl | tee ~/module06/evidence/logs/windows_event_notes.txt
cat > ~/module06/evidence/logs/correlation_notes.md <<'EOF' 
# Correlation Notes
 - Process creation event may matter more when followed by outbound network event.
 - Registry persistence clue becomes stronger when paired with suspicious process or connection 
context.
 - Triage should correlate identity, process, network, and timing rather than review each line 
in isolation. 
EOF
cat ~/module06/evidence/logs/correlation_notes.md
```
Command function:
- Introduce a correlation mindset across different event sources.
- Encourage students to explicitly document relationships between events.

Expected output:
- Sample Windows event logs are displayed.
- The file correlation_notes.md is created.
```
{"EventID":1,"TimeCreated":"2026-04-21T09:30:11Z","Image":"C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe","CommandLine":"powershell.exe -enc ZQBjAGgAbwAgAEgAZQBsAGwAbwA=","ParentImage":"C:\\Windows\\explorer.exe","User":"LAB\\student1"}
{"EventID":3,"TimeCreated":"2026-04-21T09:30:15Z","Image":"C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe","DestinationIp":"45.33.2.1","DestinationPort":443,"Protocol":"tcp","User":"LAB\\student1"}

# Correlation Notes
 - Process creation event may matter more when followed by outbound network event.
 - Registry persistence clue becomes stronger when paired with suspicious process or connection 
context.
 - Triage should correlate identity, process, network, and timing rather than review each line 
in isolation.
```


## Module 06_Phase 3 — PCAP, Beaconing, and IDS/IPS Awareness
This phase moves us from log analysis to network context. The focus is on reading traffic summaries, recognizing patterns that may be suspicious, and understanding what can—and cannot—be concluded from PCAP samples and lightweight validation.

**Phase 3.1. Review PCAP Sample with tshark on Ubuntu**
```
tshark -r ~/datasets/pcap/localhost_web_lab.pcap | head -n 30 | tee ~/module06/evidence/network/pcap_summary.txt 
```
Command function:
- Read traffic summaries from a PCAP sample without performing invasive live capture.
- Train us to quickly interpret source, destination, protocol, and request patterns.

Expected output:
- Frame summaries and connection details are displayed.
<img width="2863" height="1147" alt="image" src="https://github.com/user-attachments/assets/2c7631dd-aa45-4e9b-b893-2607d98e3b3d" />

**Phase 3.2. Write Beaconing Notes from Sample Network Logs**
```
cat ~/datasets/logs/network/firewall_logs.txt | tee ~/module06/evidence/network/beaconing_notes.md
cat > ~/module06/evidence/network/beaconing_notes.md <<'EOF' 
# Observation 
Repeated connections with similar interval and destination may suggest beacon-like behavior, but context and volume still matter before escalation. 
EOF
cat ~/module06/evidence/network/beaconing_notes.md
```
Command function:
- Connect network log samples with the concept of beaconing or repetitive call-out behavior.
- Encourage cautious analysis before labeling activity as malicious.

Expected output:
- Firewall log content is displayed and observation notes are appended.
```
02:00:00 ALLOW SRC=10.10.4.55 DST=45.33.2.1 PORT=443 PROTO=TCP LEN=450
02:05:01 ALLOW SRC=10.10.4.55 DST=45.33.2.1 PORT=443 PROTO=TCP LEN=450
02:10:02 ALLOW SRC=10.10.4.55 DST=45.33.2.1 PORT=443 PROTO=TCP LEN=452

# Observation 
Repeated connections with similar interval and destination may suggest beacon-like behavior, but context and volume still matter before escalation.
```

**Phase 3.3. Write IDS/IPS Observations Based on Context**
```
cat > ~/module06/evidence/network/ids_ips_observations.md <<'EOF' 
# IDS/IPS Observations
 - Signatures and rules help detect known or suspicious patterns, but context is still required.
 - Blocking decisions should consider false positives, business impact, and evidence quality.
 - Shared lab analysis emphasizes reasoning about detection and prevention, not deploying intrusive controls globally. 
EOF
cat ~/module06/evidence/network/ids_ips_observations.md
```
Command function:
- Teach us to differentiate between detection logic and prevention decisions.
- Position IDS/IPS as part of a broader decision-making process, not just a list of tools.

Expected output:
- The file ids_ips_observations.md is created.
<img width="1921" height="185" alt="image" src="https://github.com/user-attachments/assets/eebf0bf3-f6ac-4d5f-b952-e8e545003dca" />



## Module 06_Phase 4 — Threat Intelligence, Enrichment, and Rule-Based Review
This phase demonstrates that useful threat intelligence is not just about feeds or IOC lists, but about the ability to connect artifacts with the right context. Ubuntu is used for lightweight enrichment and YARA-based review of the available samples.

**Phase 4.1. IOC Review and Enrichment Notes**
```
grep -RinE 'http|@|vpn|verify|urgent' ~/datasets/phishing ~/datasets/osint 2>/dev/null | tee ~/module6/evidence/threatintel/ioc_review.txt
cat > ~/module06/evidence/threatintel/enrichment_notes.md <<'EOF' 
# Enrichment Notes
 - An IOC becomes more useful when tied to host, timing, and user context.
 - Email sender, destination domain, suspicious wording, and timing should be read together.
 - Enrichment should reduce ambiguity, not add noise. 
EOF
cat ~/module06/evidence/threatintel/enrichment_notes.md
```
Command function:
- Train us to extract potential IOC artifacts from phishing or OSINT samples.
- Encourage adding context through enrichment notes.

Expected output:
- Files ioc_review.txt and enrichment_notes.md are created.
<img width="2813" height="612" alt="image" src="https://github.com/user-attachments/assets/65ae028c-9a8f-475a-8ae8-528626a59bda" />

**Phase 4.2. Run YARA Helper on Text Artifacts**
```
cat > ~/module06/evidence/tmp/blue_keywords.yar <<'EOF' 
rule blue_keywords { 
  strings: 
    $a = "urgent" nocase 
    $b = "verify" nocase 
    $c = "password" nocase 
    $d = "vpn" nocase 
  condition: 
    any of them 
} 
EOF
yara ~/module06/evidence/tmp/blue_keywords.yar ~/datasets/phishing/sample_email_1.txt | tee ~/module06/evidence/threatintel/yara_findings.txt
```
Command function:
- Demonstrate rule-based helpers as a fast initial review tool.
- Connect this module to a detection mindset and simple content scanning.

Expected output:
- YARA results are displayed if matching strings are found.
<img width="2850" height="72" alt="image" src="https://github.com/user-attachments/assets/522523f6-c126-4528-9520-8c77ea6ca91d" />


## Module 06_Phase 5 — Forensics-on-Files and Timeline Building
This phase does not encourage heavy live acquisition. The focus is on realistic DFIR basics for a lab environment: preserving evidence integrity, reviewing artifacts, and building a timeline from available data.

**Phase 5.1. Review Artifact Samples and Store Integrity Hashes**
```
yara ~/module06/evidence/tmp/blue_keywords.yar ~/datasets/phishing/sample_email_1.txt | tee ~/module06/evidence/threatintel/yara_findings.txt
cat ~/datasets/forensics/registry_runkeys_sample.txt | tee ~/module06/evidence/forensics/artifact_review.txt
cat ~/datasets/forensics/usb_artifacts_sample.txt | tee -a ~/module06/evidence/forensics/artifact_review.txt
sha256sum ~/datasets/forensics/registry_runkeys_sample.txt | tee ~/module06/evidence/forensics/evidence_integrity.txt
sha256sum ~/datasets/forensics/usb_artifacts_sample.txt | tee -a ~/module06/evidence/forensics/evidence_integrity.txt
```
Command function:
- Review artifact samples relevant to persistence mechanisms and removable media context.
- Demonstrate the use of hashing to preserve evidence identity.

Expected output:
- Contents of artifact samples are displayed.
- Hash values for evidence integrity are recorded.
<img width="2864" height="359" alt="image" src="https://github.com/user-attachments/assets/247468fe-3f06-49dc-83cb-9c34bfae4298" />
Notes:
```
cat: /home/jccsah001012/datasets/forensics/usb_artifacts_sample.txt: No such file or directory
sha256sum: /home/jccsah001012/datasets/forensics/usb_artifacts_sample.txt: No such file or directory
```

**Phase 5.2. Build Timeline Notes from Available Artifacts**
```
cat > ~/module06/evidence/forensics/timeline_notes.md <<'EOF' 

# Timeline Notes 
1. Phishing-style artifact identified. 
2. Auth and service context reviewed. 
3. Web and network evidence reviewed. 
4. Registry/USB artifact sample observed. 
5. Evidence integrity recorded. 
6. Incident assessment prepared. 
EOF
cat ~/module06/evidence/forensics/timeline_notes.md
```
Command function:
- Train us to organize observations into a structured sequence for incident narratives.
- Show that timelines are useful even without large-scale incidents.

Expected output:
- The file timeline_notes.md is created.
<img width="1407" height="324" alt="image" src="https://github.com/user-attachments/assets/5b8fdb0e-4c27-4d8a-9dee-c3adf2f3cc11" />



## Module 06_Phase 6 — Triage Decision and Incident Workflow
This phase concludes the analysis with decision-making. Students are expected to define what should be prioritized, what still requires additional data, and how an initial response should be structured.

**Phase 6.1. Write Triage Decision Notes**
```
cat > ~/module06/evidence/incident/triage_decision_notes.md <<'EOF' 
# Triage Decision Notes 
 
## Priority Questions
 - Is the issue identity-related, application-related, or host-related?
 - Which signal is strongest: log, network, phishing content, or artifact evidence?
 - What needs validation before escalation? 
 
## Immediate Actions
 - Preserve evidence
 - Validate context
 - Correlate across sources
 - Avoid premature conclusions 
EOF
cat /module06/evidence/incident/triage_decision_notes.md
```
Command function:
- Encourage us to think in terms of priorities and decisions, not just raw observations.

Expected output:
- The file triage_decision_notes.md is created.
```
# Triage Decision Notes 
 
## Priority Questions
 - Is the issue identity-related, application-related, or host-related?
 - Which signal is strongest: log, network, phishing content, or artifact evidence?
 - What needs validation before escalation? 
 
## Immediate Actions
 - Preserve evidence
 - Validate context
 - Correlate across sources
 - Avoid premature conclusions 
```

**Phase 6.2. Write Response Playbook Notes and Final Case Assessment**
```
cat > ~/module06/evidence/incident/response_playbook_notes.md <<'EOF' 
# Response Playbook Notes 
 - Confirm scope and preserve evidence first.
 - Separate validation from remediation. - Escalate when multiple signals align.
 - Document what is known, unknown, and assumed. 
EOF
cat ~/module06/evidence/incident/response_playbook_notes.md

cat > ~/module06/evidence/incident/final_case_assessment.md <<'EOF' 
# Final Case Assessment 
 
## Summary 
The environment provides multiple defensive signals: auth events, web logs, network sample, phishing-style artifact, and forensics sample. Stronger conclusions emerge when these are correlated instead of reviewed in isolation. 
 
## Key Themes
 - Identity context matters
 - Service context matters
 - Evidence integrity matters
 - Correlation matters more than any single line of output 
EOF
cat ~/module06/evidence/incident/final_case_assessment.md
```
Command function:
- Position response thinking as a practical conclusion, not just theoretical incident response lifecycle.

Expected output:
- Files response_playbook_notes.md and final_case_assessment.md are created.
```
# Response Playbook Notes 
 - Confirm scope and preserve evidence first.
 - Separate validation from remediation. - Escalate when multiple signals align.
 - Document what is known, unknown, and assumed.


# Final Case Assessment 
 
## Summary 
The environment provides multiple defensive signals: auth events, web logs, network sample, phishing-style artifact, and forensics sample. Stronger conclusions emerge when these are correlated instead of reviewed in isolation. 
 
## Key Themes
 - Identity context matters
 - Service context matters
 - Evidence integrity matters
 - Correlation matters more than any single line of output
```


## Module 06_Phase 7 — Final Outputs and Bridge to the Next Module
This closing phase ensures that students leave the module with a solid assessment and a clear bridge to the next offensive module. This is important so that students understand that mastering defensive concepts will significantly strengthen and better guide their offensive perspective.

**Phase 7.1. Write the Module 06 Assessment**
```
cat > ~/module06/reports/module06_assessment.md <<'EOF' 
# Module 06 Assessment 
 
## Summary 
Defensive security requires evidence-driven thinking across host, log, network, application, and artifact layers. 
 
## Practical Takeaway
 - Ubuntu remains the primary analysis host
 - Kali remains the validation workstation
 - Log review, PCAP, YARA, and artifact analysis should be correlated
 - Triage and response notes are part of operational quality, not optional extras 
EOF
cat ~/module06/reports/module06_assessment.md
```

Command function:
- Summarizes the entire module into a clear, professional assessment format.

Expected output:
- The file module06_assessment.md is created and available.
```
# Module 06 Assessment 
 
## Summary 
Defensive security requires evidence-driven thinking across host, log, network, application, and artifact layers. 
 
## Practical Takeaway
 - Ubuntu remains the primary analysis host
 - Kali remains the validation workstation
 - Log review, PCAP, YARA, and artifact analysis should be correlated
 - Triage and response notes are part of operational quality, not optional extras
```

**Phase 7.2. Create Bridge Notes to the Next Offensive Module**
```
cat > ~/module06/reports/module06_bridge_notes.md <<'EOF' 
# Module 06 Bridge Notes 
 
## What carries forward
 - Correlation matters more than isolated signals
 - Validation from multiple viewpoints is essential 
 - Evidence integrity must be preserved
 - Response thinking begins with scope and context 

## Bridge to next module
 - Modul 07 will examine offensive pathways, but the same context, evidence, and validation discipline still applies 
EOF
cat ~/module06/reports/module06_bridge_notes.md
```
Command function:
- Reinforces that transitioning to offensive security must still carry forward discipline in evidence handling and contextual analysis.

Expected output:
- The file module06_bridge_notes.md is created and available.
```
# Module 06 Bridge Notes 
 
## What carries forward
 - Correlation matters more than isolated signals
 - Validation from multiple viewpoints is essential 
 - Evidence integrity must be preserved
 - Response thinking begins with scope and context 

## Bridge to next module
 - Modul 07 will examine offensive pathways, but the same context, evidence, and validation discipline still applies
```


## Module 06_Closing
Module 06 is designed to help us understand that blue team work is not just about reading logs or waiting for alerts, but about building context, correlating evidence, assessing priorities, and writing responses that can be justified and defended. With this foundation, we will move into the next module with a more mature defensive perspective and be better prepared to compare it with the offensive point of view in a more balanced way.
