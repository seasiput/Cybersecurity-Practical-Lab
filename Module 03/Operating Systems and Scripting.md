# Module 03 — Host Analysis, Permission Review, Process Observation, Logging, and Automation

This guide is intended for us to carry out practicum activities independently or under supervision on the Ubuntu Lab and Kali Lab. The focus of this module is to build working patterns relevant to junior security analysts, SOC analysts, system support personnel, and security operations teams: understanding host context, reviewing metadata and permissions, correlating processes with services, analyzing logs and PCAPs, and concluding the practice with organized automation and assessment.

**1. Lab Objectives**
- Understand operating systems as sources of observation, control, and evidence within cybersecurity work environments.
- Read user context, groups, file systems, metadata, permissions, processes, sessions, ports, and logs in a structured manner without relying on full administrative privileges.
- Use the Ubuntu Lab as the primary analysis host for file review, metadata analysis, process correlation, log parsing, YARA scanning, and automation.
- Use the Kali Lab as a validation workstation to observe targets, services, and evidence from a different assessment perspective.
- Build working habits suitable for advanced modules: organized evidence, documented output, and concise assessments that can be understood by technical teams.

**2. Environtment Overview and Host Roles**
<img width="1427" height="936" alt="image" src="https://github.com/user-attachments/assets/bd0fb1e8-a07a-4787-96d8-03560f4f28ca" />

**3. Practical Workflow**
- Prepare the workspace and verify tool readiness on Ubuntu and Kali.
- Review the host baseline, user context, storage, memory, and process state.
- Analyze the file system, ownership, metadata, permissions, and file patterns relevant to security operations.
- Safely observe processes, sessions, ports, and service relationships.
- Analyze log and PCAP samples to build host monitoring context.
- Build simple automation using Bash and Python for evidence parsing.
- Conclude the practicum with a concise, clear, and professionally readable assessment.

## Module 03_Phase 1 — Preparing the Workspace and Tool Readiness

This phase ensures that we work with a well-organized structure and clearly understand which resources are available for use. After this phase is completed, there should no longer be confusion regarding helpers, datasets, or evidence locations.

**Phase 1.1. Create the Module 3 Workspace on Ubuntu**
```
mkdir -p ~/module03/evidence/{automation,filesystem,monitoring,notes,processes,baseline,reports} 
cd ~/module03 
ls -R ~/module03/evidence
```

Function of the command:
- Create a folder structure so that evidence, notes, and reports are stored separately.
- Reduce the risk of losing output as the practicum becomes more extensive.

Expected output:
- The active path is located at ~/module03.
- The folders evidence, reports, notes, baseline, filesystem, processes, monitoring, and automation are visible.

<img width="1178" height="610" alt="image" src="https://github.com/user-attachments/assets/45f7ad01-8dd2-48da-995e-1945c56f5db7" />



**Phase 1.2. Activate the Shared Environment and Check Lab Helpers**
```
source ~/activate-pylab 
~/jccsah-env-info 
ls -la ~/datasets ~/samples ~/wordlists ~/starter
```

Function of the command:
- Activate the Python environment prepared by the lab administrator.
- Ensure that symlinks to datasets, samples, wordlists, and starter are available.

Expected output:
- The Python environment is active.
- Important commands such as `python3`, `yara`, `tshark`, `nmap`, and other helpers appear in `jccsah-env-info`.

<img width="1757" height="901" alt="image" src="https://github.com/user-attachments/assets/057f1bb5-8c9b-496b-b092-342e268c52f6" />


## Module 03_Phase 2 — Host Baseline, User Context, and Resource State

Before analyzing risks, we must understand the context of the host they are using. In real-world environments, many misinterpretations occur because host context and resource state are not checked from the beginning.

**Phase 2.1. Host Identity and User Context**
```
whoami | tee ~/module03/evidence/baseline/whoami.txt 
id | tee ~/module03/evidence/baseline/id.txt 
hostname | tee ~/module03/evidence/baseline/hostname.txt 
uname -a | tee ~/module03/evidence/baseline/uname.txt 
date | tee ~/module03/evidence/baseline/date.txt
```

Function of the command:
- Record the active user, assigned groups, hostname, kernel, and system time.
- Provide context when evidence is later compared with output from other hosts.

Expected output:
- The active user matches the student account.
- id displays the relevant UID, GID, and groups.
- The kernel version and system time are clearly displayed.
```
jccsah001012
uid=1011(jccsah001012) gid=1011(jccsah001012) groups=1011(jccsah001012)
localhost
Linux localhost 6.8.0-110-generic #110-Ubuntu SMP PREEMPT_DYNAMIC Thu Mar 19 15:09:20 UTC 2026 x86_64 x86_64 x86_64 GNU/Linux
Thu Apr 30 07:59:11 PM WIB 2026
```

Meaning of the output: Context is the foundation of all analysis. Without the correct context, evidence interpretation can easily become inaccurate.

**Phase 2.2. Resource Baseline: Disk, Memory, and Process Snapshot**
```
df -h | tee ~/module03/evidence/baseline/disk_usage.txt
free -h | tee ~/module03/evidence/baseline/memory_usage.txt
ps -eo pid,ppid,user,%cpu,%mem,tty,stat,start,time,command --sort=-%mem | head -n 25 | tee ~/module03/evidence/baseline/mem_process_snapshot.txt
```

Function of the command:
- Quickly review storage and memory conditions.
- Identify processes consuming the most resources.

Expected output:
- df -h displays mount points and storage usage.
```
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           795M  1.3M  793M   1% /run
/dev/sda        157G   11G  139G   7% /
tmpfs           3.9G     0  3.9G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           795M   16K  795M   1% /run/user/1010
tmpfs           795M   16K  795M   1% /run/user/1011
```
- free -h displays memory and swap usage.
```
               total        used        free      shared  buff/cache   available
Mem:           7.8Gi       846Mi       983Mi       9.7Mi       6.3Gi       6.9Gi
Swap:          495Mi       768Ki       495Mi
```
- ps displays the top processes sorted by memory usage.
```
    PID    PPID USER     %CPU %MEM TT       STAT  STARTED     TIME COMMAND
  25952   25929 65532     0.4  2.4 ?        Ssl    Apr 20 01:07:51 /nodejs/bin/node /juice-shop/build/app.js
  23551       1 root      0.0  1.0 ?        Ssl    Apr 20 00:01:26 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
  26319   26173 message+  0.0  0.9 ?        Sl     Apr 20 00:10:49 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/x86_64-linux-gnu/mariadb18/plugin --user=mysql --skip-log-error --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306
  54969       1 root      0.0  0.9 ?        S<s    Apr 22 00:03:12 /usr/lib/systemd/systemd-journald
  23047       1 root      0.0  0.6 ?        Ssl    Apr 20 00:12:33 /usr/bin/containerd
  26426   26088 root      0.0  0.3 ?        Ss     Apr 20 00:00:39 /usr/sbin/apache2 -k start
    814       1 root      0.0  0.2 ?        Ssl    Apr 20 00:00:00 /usr/bin/python3 /usr/share/unattended-upgrades/unattended-upgrade-shutdown --wait-for-signal
 229054       1 clamav    0.0  0.2 ?        Ss     Apr 29 00:00:00 /usr/bin/freshclam -d --foreground=true
  26436   26426 www-data  0.0  0.1 ?        S      Apr 20 00:00:00 /usr/sbin/apache2 -k start
  26435   26426 www-data  0.0  0.1 ?        S      Apr 20 00:00:00 /usr/sbin/apache2 -k start
  26438   26426 www-data  0.0  0.1 ?        S      Apr 20 00:00:00 /usr/sbin/apache2 -k start
  26439   26426 www-data  0.0  0.1 ?        S      Apr 20 00:00:00 /usr/sbin/apache2 -k start
  26440   26426 www-data  0.0  0.1 ?        S      Apr 20 00:00:00 /usr/sbin/apache2 -k start
      1       0 root      0.0  0.1 ?        Ss     Apr 20 00:00:55 /usr/lib/systemd/systemd --system --deserialize=86
  54961       1 root      0.0  0.1 ?        Ssl    Apr 22 00:00:13 /usr/libexec/udisks2/udisksd
  54952       1 systemd+  0.0  0.1 ?        Ss     Apr 22 00:00:01 /usr/lib/systemd/systemd-resolved
 272404       1 jccsah0+  0.0  0.1 ?        Ss   19:12:49 00:00:00 /usr/lib/systemd/systemd --user
 271932       1 jccsah0+  0.0  0.1 ?        Ss   17:28:45 00:00:00 /usr/lib/systemd/systemd --user
  25929       1 root      0.0  0.1 ?        Sl     Apr 20 00:00:53 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 9664db1b259cc11408c1509431b3c9cc79ed9d1b62462ed34eacb9d5a36c0a9a -address /run/containerd/containerd.sock
  26064       1 root      0.0  0.1 ?        Sl     Apr 20 00:00:54 /usr/bin/containerd-shim-runc-v2 -namespace moby -id a77ea8ae734cd3fc6b88b72b6a633658396e29aec726731d5e692fbb16c3bf30 -address /run/containerd/containerd.sock
 272884  266422 root      0.0  0.1 ?        Ss   20:02:28 00:00:00 sshd: jccsah001011 [priv]
 272400  266422 root      0.0  0.1 ?        Ss   19:12:48 00:00:00 sshd: jccsah001012 [priv]
 272633  266422 root      0.0  0.1 ?        Ss   19:43:00 00:00:00 sshd: jccsah001011 [priv]
  55006       1 systemd+  0.0  0.1 ?        Ss     Apr 22 00:00:01 /usr/lib/systemd/systemd-networkd
```

Meaning of the output: Security symptoms are often mixed with performance symptoms. A resource baseline helps distinguish between the two.


## Module 03_Phase 3 — File System, Metadata, Ownership, and Permission Review

This phase emphasizes that many host security issues originate from files stored in the wrong location, overly permissive permissions, or incorrect ownership.The focus is not only on running ls, but on fully understanding metadata.

**Phase 3.1. Review the Home Directory, Hidden Files, and Path Permissions**
```
echo $HOME 
ls -la $HOME | tee ~/module03/evidence/filesystem/home_listing.txt 
namei -l $HOME | tee ~/module03/evidence/filesystem/home_path_permissions.txt
```

Function of the command:
- Display the contents of the home directory, including hidden files that are often relevant to user configuration.
- Read the permissions of each path component using namei -l.

Expected output:
- The full contents of the home directory are displayed.
- The namei -l output shows the owner and permission mode for each part of the path.

Meaning of the output: Permissions should not only be reviewed on the target file itself; the path leading to the file must also be understood.
```
/home/jccsah001012

total 100
drwx------ 12 jccsah001012 jccsah001012  4096 Apr 30 19:27 .
drwxr-xr-x 16 root         root          4096 Apr 20 16:11 ..
lrwxrwxrwx  1 jccsah001012 jccsah001012    30 Apr 20 20:20 activate-pylab -> /opt/jccsah/bin/activate-pylab
-rw-------  1 jccsah001012 jccsah001012 29620 Apr 30 15:34 .bash_history
-rw-r--r--  1 jccsah001012 jccsah001012   220 Mar 31  2024 .bash_logout
-rw-r--r--  1 jccsah001012 jccsah001012  3826 Apr 20 20:20 .bashrc
drwx------  2 jccsah001012 jccsah001012  4096 Apr 20 17:59 .cache
lrwxrwxrwx  1 jccsah001012 jccsah001012    20 Apr 20 20:20 datasets -> /opt/jccsah/datasets
lrwxrwxrwx  1 jccsah001012 jccsah001012    31 Apr 20 20:20 jccsah-env-info -> /opt/jccsah/bin/jccsah-env-info
drwxrwxr-x  8 jccsah001012 jccsah001012  4096 Apr 20 21:11 lab501
drwxr-xr-x  2 jccsah001012 jccsah001012  4096 Apr 20 20:20 labs
lrwxrwxrwx  1 jccsah001012 jccsah001012    36 Apr 20 20:21 lab-targets-info -> /opt/jccsah/scripts/lab-targets-info
drwxrwxr-x  6 jccsah001012 jccsah001012  4096 Apr 29 11:57 lap-prep
drwxrwxr-x  7 jccsah001012 jccsah001012  4096 Apr 29 18:46 module01
drwxrwxr-x 11 jccsah001012 jccsah001012  4096 Apr 30 10:20 module02
drwxrwxr-x  3 jccsah001012 jccsah001012  4096 Apr 30 20:30 module03
drwxrwxr-x  6 jccsah001012 jccsah001012  4096 Apr 29 12:00 module501
drwxr-xr-x  2 jccsah001012 jccsah001012  4096 Apr 20 20:20 notes
-rw-r--r--  1 jccsah001012 jccsah001012   807 Mar 31  2024 .profile
drwxr-xr-x  2 jccsah001012 jccsah001012  4096 Apr 20 20:20 projects
lrwxrwxrwx  1 jccsah001012 jccsah001012    19 Apr 20 20:20 samples -> /opt/jccsah/samples
lrwxrwxrwx  1 jccsah001012 jccsah001012    19 Apr 20 20:21 starter -> /opt/jccsah/starter
lrwxrwxrwx  1 jccsah001012 jccsah001012    17 Apr 20 20:20 tools -> /opt/jccsah/tools
-rw-------  1 jccsah001012 jccsah001012   532 Apr 20 23:21 .viminfo
lrwxrwxrwx  1 jccsah001012 jccsah001012    21 Apr 20 20:20 wordlists -> /opt/jccsah/wordlists

f: /home/jccsah001012
drwxr-xr-x root         root         /
drwxr-xr-x root         root         home
drwx------ jccsah001012 jccsah001012 jccsah001012
```

**Phase 3.2. Compare Ownership and Permission Modes on Practice Files**
```
mkdir -p ~/module03/filesystem/labdata
cd ~/module03/filesystem/labdata
touch note.txt report.txt script.sh
chmod 600 note.txt
chmod 640 report.txt
chmod 750 script.sh
ls -l | tee ~/module03/evidence/filesystem/ownership_review.txt
stat note.txt report.txt script.sh | tee ~/module03/evidence/filesystem/stat_review.txt
```

Function of the command:
- Observe differences in permission modes between regular files and files considered executable.
- Train students to read ls -l and stat outputs as sources of evidence.

Expected output:
- The files note.txt, report.txt, and script.sh appear with different permission modes.
- stat displays the owner, group, mode, and timestamps of the files.

Meaning of the output: File system security reviews always require metadata analysis, not just reading filenames.
```
ownership_review.txt:
total 0
-rw------- 1 jccsah001012 jccsah001012 0 Apr 30 20:57 note.txt
-rw-r----- 1 jccsah001012 jccsah001012 0 Apr 30 20:57 report.txt
-rwxr-x--- 1 jccsah001012 jccsah001012 0 Apr 30 20:57 script.sh

stat_review.txt:
  File: note.txt
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 8,0     Inode: 680269      Links: 1
Access: (0600/-rw-------)  Uid: ( 1011/jccsah001012)   Gid: ( 1011/jccsah001012)
Access: 2026-04-30 20:57:25.170657750 +0700
Modify: 2026-04-30 20:57:25.170657750 +0700
Change: 2026-04-30 20:57:31.448849285 +0700
 Birth: 2026-04-30 20:57:25.170657750 +0700
  File: report.txt
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 8,0     Inode: 680270      Links: 1
Access: (0640/-rw-r-----)  Uid: ( 1011/jccsah001012)   Gid: ( 1011/jccsah001012)
Access: 2026-04-30 20:57:25.170657750 +0700
Modify: 2026-04-30 20:57:25.170657750 +0700
Change: 2026-04-30 20:57:37.567035958 +0700
 Birth: 2026-04-30 20:57:25.170657750 +0700
  File: script.sh
  Size: 0               Blocks: 0          IO Block: 4096   regular empty file
Device: 8,0     Inode: 680271      Links: 1
Access: (0750/-rwxr-x---)  Uid: ( 1011/jccsah001012)   Gid: ( 1011/jccsah001012)
Access: 2026-04-30 20:57:25.170657750 +0700
Modify: 2026-04-30 20:57:25.170657750 +0700
Change: 2026-04-30 20:57:42.989201410 +0700
 Birth: 2026-04-30 20:57:25.170657750 +0700
```

**Phase 3.3. Search for Files Resembling Sensitive Data or Important Artifacts**
```
find -L ~/datasets ~/samples -maxdepth 3 -type f \( -iname "*.log" -o -iname "*.json" -o -iname "*.pcap" -o -iname "*.csv" -o -iname "*.txt" \) | sort | tee  ~/module03/evidence/filesystem/find_sensitive_like.txt
```

Function of the command:
- Build the habit of searching for evidence based on file type and practical value.
- Train students to distinguish operational data, logs, artifacts, and analysis support files.

Expected output:
- A list of log, JSON, PCAP, CSV, and TXT files from datasets or samples is displayed.

Meaning of the output: In real-world work, analysts must quickly locate relevant evidence without manually browsing the entire file system.
<img width="2862" height="249" alt="image" src="https://github.com/user-attachments/assets/e0740d25-6dac-4ee3-b6ec-f03066beb1a8" />


## Module 03_Phase 4 — Process, Session, Port, and Service Relationship

This phase takes students from the file system level to the system activity level. The focus is on understanding running processes, logged-in users, listening ports, and how services appear both from the host perspective and from the workstation perspective.

**Phase 4.1. Review Processes Based on CPU, Memory, and Parent-Child Context**
```
ps -eo pid,ppid,user,%cpu,%mem,tty,stat,start,time,command --sort=-%cpu | head -n 25 | tee ~/module03/evidence/processes/ps_review.txt
pstree -ap | head -n 80 | tee ~/module03/evidence/processes/pstree_review.txt
```

Function of the command:
- Quickly identify active processes and parent-child relationships.
- Help students understand how processes are created, invoked, and maintained by the system.

Expected output:
- A list of active processes with CPU and memory metrics is displayed.
```
    PID    PPID USER     %CPU %MEM TT       STAT  STARTED     TIME COMMAND
  25952   25929 65532     0.4  2.4 ?        Ssl    Apr 20 01:08:06 /nodejs/bin/node /juice-shop/build/app.js
  23047       1 root      0.0  0.6 ?        Ssl    Apr 20 00:12:36 /usr/bin/containerd
  26319   26173 message+  0.0  0.9 ?        Sl     Apr 20 00:10:51 /usr/sbin/mysqld --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/x86_64-linux-gnu/mariadb18/plugin --user=mysql --skip-log-error --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/run/mysqld/mysqld.sock --port=3306
  54969       1 root      0.0  0.9 ?        S<s    Apr 22 00:03:13 /usr/lib/systemd/systemd-journald
  23551       1 root      0.0  1.0 ?        Ssl    Apr 20 00:01:26 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
 273216       2 root      0.0  0.0 ?        I    21:10:16 00:00:00 [kworker/u8:5-events_unbound]
     50       2 root      0.0  0.0 ?        SN     Apr 20 00:01:22 [khugepaged]
 273128       2 root      0.0  0.0 ?        I    20:43:52 00:00:00 [kworker/u8:2-events_unbound]
  26446   26088 root      0.0  0.0 ?        S      Apr 20 00:00:57 tail -f /var/log/apache2/access.log /var/log/apache2/error.log /var/log/apache2/other_vhosts_access.log
  26064       1 root      0.0  0.1 ?        Sl     Apr 20 00:00:55 /usr/bin/containerd-shim-runc-v2 -namespace moby -id a77ea8ae734cd3fc6b88b72b6a633658396e29aec726731d5e692fbb16c3bf30 -address /run/containerd/containerd.sock
      1       0 root      0.0  0.1 ?        Ss     Apr 20 00:00:55 /usr/lib/systemd/systemd --system --deserialize=86
  25929       1 root      0.0  0.1 ?        Sl     Apr 20 00:00:53 /usr/bin/containerd-shim-runc-v2 -namespace moby -id 9664db1b259cc11408c1509431b3c9cc79ed9d1b62462ed34eacb9d5a36c0a9a -address /run/containerd/containerd.sock
 272314       2 root      0.0  0.0 ?        I    18:43:38 00:00:00 [kworker/u8:0-flush-8:0]
  54991       1 syslog    0.0  0.0 ?        Ssl    Apr 22 00:00:43 /usr/sbin/rsyslogd -n -iNONE
 271444       2 root      0.0  0.0 ?        I    15:34:16 00:00:01 [kworker/0:0-events]
 272495  272400 jccsah0+  0.0  0.0 ?        S    19:12:50 00:00:00 sshd: jccsah001012@pts/1
  26426   26088 root      0.0  0.3 ?        Ss     Apr 20 00:00:39 /usr/sbin/apache2 -k start
     17       2 root      0.0  0.0 ?        I      Apr 20 00:00:35 [rcu_preempt]
     47       2 root      0.0  0.0 ?        S      Apr 20 00:00:33 [kcompactd0]
 266422       1 root      0.0  0.1 ?        Ss   06:30:15 00:00:02 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
  54955       1 root      0.0  0.0 ?        S<sl   Apr 22 00:00:22 /sbin/auditd
 272496  272495 jccsah0+  0.0  0.0 pts/1    Ss   19:12:50 00:00:00 -bash
 272537       2 root      0.0  0.0 ?        I    19:15:48 00:00:00 [kworker/1:1-events]
 272334       2 root      0.0  0.0 ?        I    18:50:28 00:00:00 [kworker/3:1-events]
```
- pstree shows hierarchical process relationships.
```
systemd,1 --system --deserialize=86
  |-agetty,812 -o -p -- \\u --keep-baud 115200,57600,38400,9600 - vt220
  |-agetty,823 -o -p -- \\u --noclear - linux
  |-atd,22587 -f
  |-auditd,54955
  |   `-{auditd},54956
  |-containerd,23047
  |   |-{containerd},23049
  |   |-{containerd},23050
  |   |-{containerd},23051
  |   |-{containerd},23052
  |   |-{containerd},23053
  |   |-{containerd},23054
  |   |-{containerd},23055
  |   |-{containerd},23069
  |   |-{containerd},25735
  |   `-{containerd},25736
  |-containerd-shim,25929 -namespace moby -id 9664db1b259cc11408c1509431b3c9cc79ed9d1b62462ed34eacb9d5a36c0a9a -address/run/
  |   |-MainThread,25952 /juice-shop/build/app.js
  |   |   |-{MainThread},26002
  |   |   |-{MainThread},26003
  |   |   |-{MainThread},26004
  |   |   |-{MainThread},26005
  |   |   |-{MainThread},26006
  |   |   |-{MainThread},26015
  |   |   |-{MainThread},26017
  |   |   |-{MainThread},26018
  |   |   |-{MainThread},26019
  |   |   `-{MainThread},26020
  |   |-{containerd-shim},25930
  |   |-{containerd-shim},25931
  |   |-{containerd-shim},25932
  |   |-{containerd-shim},25933
  |   |-{containerd-shim},25934
  |   |-{containerd-shim},25935
  |   |-{containerd-shim},25936
  |   |-{containerd-shim},25937
  |   |-{containerd-shim},25958
  |   `-{containerd-shim},25959
  |-containerd-shim,26064 -namespace moby -id a77ea8ae734cd3fc6b88b72b6a633658396e29aec726731d5e692fbb16c3bf30 -address/run/
  |   |-main.sh,26088 /main.sh
  |   |   |-apache2,26426 -k start
  |   |   |   |-apache2,26435 -k start
  |   |   |   |-apache2,26436 -k start
  |   |   |   |-apache2,26438 -k start
  |   |   |   |-apache2,26439 -k start
  |   |   |   `-apache2,26440 -k start
  |   |   |-mysqld_safe,26173 /usr/bin/mysqld_safe
  |   |   |   |-logger,26320 -t mysqld -p daemon error
  |   |   |   `-mysqld,26319 --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib/x86_64-linux-gnu/mariadb18/plugin ...
  |   |   |       |-{mysqld},26321
  |   |   |       |-{mysqld},26322
  |   |   |       |-{mysqld},26324
  |   |   |       |-{mysqld},26325
  |   |   |       |-{mysqld},26326
  |   |   |       |-{mysqld},26327
  |   |   |       |-{mysqld},26328
  |   |   |       |-{mysqld},26329
  |   |   |       |-{mysqld},26330
  |   |   |       |-{mysqld},26331
  |   |   |       |-{mysqld},26332
  |   |   |       |-{mysqld},26333
  |   |   |       |-{mysqld},26334
  |   |   |       |-{mysqld},26337
  |   |   |       |-{mysqld},26338
  |   |   |       |-{mysqld},26339
  |   |   |       |-{mysqld},26340
  |   |   |       |-{mysqld},26341
  |   |   |       |-{mysqld},26342
  |   |   |       |-{mysqld},26343
  |   |   |       |-{mysqld},26344
  |   |   |       |-{mysqld},26345
  |   |   |       |-{mysqld},26346
  |   |   |       |-{mysqld},26347
  |   |   |       |-{mysqld},26348
  |   |   |       |-{mysqld},26351
  |   |   |       `-{mysqld},166492
  |   |   `-tail,26446 -f /var/log/apache2/access.log /var/log/apache2/error.log /var/log/apache2/other_vhosts_access.log
  |   |-{containerd-shim},26065
  |   |-{containerd-shim},26066
```

Meaning of the output: Good process analysis does not stop at process names; relationships between processes are often far more valuable.


**Phase 4.2. Review User Sessions and Active Terminals**
```
w | tee ~/module03/evidence/processes/w_output.txt
who | tee ~/module03/evidence/processes/who_output.txt
last -n 10 | tee ~/module03/evidence/processes/last_output.txt 
```

Function of the command:
- View who is currently logged in, from where, and which terminal is being used.
- Build the habit of reviewing session information before drawing conclusions about user activity.

Expected output:
- The outputs of w and who display active sessions.
- last displays a brief login history.
```
w_output.txt:
 21:37:03 up 10 days,  5:59,  3 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU  WHAT
jccsah00          180.243.11.87    20:02    7:44   0.00s  0.05s sshd: jccsah001011 [priv]
jccsah00          180.243.0.32     19:43    7:44   0.00s  0.05s sshd: jccsah001011 [priv]
jccsah00          180.252.52.208   19:12    7:44   0.00s  0.05s sshd: jccsah001012 [priv]

who_output.txt:
jccsah001012 pts/1        2026-04-30 19:12 (180.252.52.208)
jccsah001011 pts/2        2026-04-30 19:43 (180.243.0.32)
jccsah001011 pts/3        2026-04-30 20:02 (180.243.11.87)

last_output.txt:
jccsah00 pts/3        180.243.11.87    Thu Apr 30 20:02   still logged in
jccsah00 pts/2        180.243.0.32     Thu Apr 30 19:43   still logged in
jccsah00 pts/1        180.252.52.208   Thu Apr 30 19:12   still logged in
jccsah00 pts/0        101.255.110.198  Thu Apr 30 17:28 - 20:12  (02:43)
jccsah00 pts/1        101.255.110.198  Thu Apr 30 14:39 - 16:51  (02:11)
jccsah00 pts/0        180.252.49.211   Thu Apr 30 10:06 - 15:34  (05:27)
jccsah00 pts/2        101.255.110.198  Thu Apr 30 08:30 - 12:51  (04:20)
jccsah00 pts/1        180.243.0.32     Thu Apr 30 07:33 - 09:50  (02:17)
jccsah00 pts/0        180.243.0.32     Thu Apr 30 06:41 - 08:55  (02:14)
jccsah00 pts/2        180.243.0.32     Thu Apr 30 04:27 - 06:40  (02:12)
```
Meaning of the output: Session review is important for access validation, incident investigation, and activity auditing.

**Phase 4.3. Correlate Processes with Listening Ports on Ubuntu, Then Validate from Kali Ubuntu Lab**
```
# Ubuntu Lab 
ss -tulpn | tee ~/module03/evidence/processes/ss_review.txt

# Kali Lab 
nmap -sV -Pn -p 3000,8081 127.0.0.1 -oN ~/module03/evidence/processes/nmap_targets.txt 
```

Function of the command:
- Review listening ports from the Ubuntu host perspective.
- Validate from the assessment perspective how local services appear.

Expected output:
- ss displays relevant listening ports.
- nmap displays active services on the localhost targets.
```
ss_review.txt: 
Netid State  Recv-Q Send-Q Local Address:Port  Peer Address:PortProcess
udp   UNCONN 0      0         127.0.0.54:53         0.0.0.0:*          
udp   UNCONN 0      0      127.0.0.53%lo:53         0.0.0.0:*          
tcp   LISTEN 0      4096         0.0.0.0:22         0.0.0.0:*          
tcp   LISTEN 0      4096   127.0.0.53%lo:53         0.0.0.0:*          
tcp   LISTEN 0      4096       127.0.0.1:8081       0.0.0.0:*          
tcp   LISTEN 0      4096       127.0.0.1:3000       0.0.0.0:*          
tcp   LISTEN 0      4096       127.0.0.1:39559      0.0.0.0:*          
tcp   LISTEN 0      4096         0.0.0.0:2222       0.0.0.0:*          
tcp   LISTEN 0      4096      127.0.0.54:53         0.0.0.0:*          
tcp   LISTEN 0      4096            [::]:22            [::]:*          
tcp   LISTEN 0      4096            [::]:2222          [::]:*

nmap_targets.txt:
Starting Nmap 7.99 ( https://nmap.org ) at 2026-04-30 14:39 +0000
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000047s latency).

PORT     STATE SERVICE VERSION
3000/tcp open  ppp?
8081/tcp open  http    Apache httpd 2.4.25 ((Debian))
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port3000-TCP:V=7.99%I=7%D=4/30%Time=69F369CA%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,101B9,"HTTP/1\.1\x20200\x20OK\r\nAccess-Control-Allow-Origin:\
SF:x20\*\r\nX-Content-Type-Options:\x20nosniff\r\nX-Frame-Options:\x20SAME
SF:ORIGIN\r\nFeature-Policy:\x20payment\x20'self'\r\nX-Recruiting:\x20/#/j
SF:obs\r\nAccept-Ranges:\x20bytes\r\nCache-Control:\x20public,\x20max-age=
SF:0\r\nLast-Modified:\x20Mon,\x2020\x20Apr\x202026\x2013:15:48\x20GMT\r\n
SF:ETag:\x20W/\"124fa-19dab08698f\"\r\nContent-Type:\x20text/html;\x20char
SF:set=UTF-8\r\nContent-Length:\x2075002\r\nVary:\x20Accept-Encoding\r\nDa
SF:te:\x20Thu,\x2030\x20Apr\x202026\x2014:40:10\x20GMT\r\nConnection:\x20c
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
SF:20Apr\x202026\x2014:40:10\x20GMT\r\nConnection:\x20close\r\n\r\n")%r(RT
SF:SPRequest,EA,"HTTP/1\.1\x20204\x20No\x20Content\r\nAccess-Control-Allow
SF:-Origin:\x20\*\r\nAccess-Control-Allow-Methods:\x20GET,HEAD,PUT,PATCH,P
SF:OST,DELETE\r\nVary:\x20Access-Control-Request-Headers\r\nContent-Length
SF::\x200\r\nDate:\x20Thu,\x2030\x20Apr\x202026\x2014:40:10\x20GMT\r\nConn
SF:ection:\x20close\r\n\r\n");

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.62 seconds
```

Meaning of the output: Good service discovery always compares the internal host perspective with the assessment perspective.


## Module 03_Phase 5 — Log Management, Monitoring, and Evidence Correlation

This phase maximizes the use of Ubuntu and Kali resources: journals, authentication log samples, PCAP samples, and YARA. The primary focus is on building context, not merely running standalone commands.

**Phase 5.1. Review Recent System Events and Authentication Patterns**
```
journalctl -n 50 --no-pager | tee ~/module03/evidence/monitoring/journal_review.txt
grep -i "failed\|accepted" ~/datasets/logs/linux/auth.log.sample | tee ~/module03/evidence/monitoring/auth_log_review.txt
```

Function of the command:
- Review recent system events and observe authentication patterns.
- Train students to recognize successful events, failed logins, and event timing context.

Expected output:
- Snippets of journal events are displayed.
- Lines containing failed and accepted login events from the authentication log dataset are displayed.
```
journal_review.txt:
Hint: You are currently not seeing messages from other users and the system.
      Users in groups 'adm', 'systemd-journal' can see all messages.
      Pass -q to turn off this notice.
May 01 12:14:43 localhost systemd[279234]: Listening on snapd.session-agent.socket - REST API socket for snapd user session agent.
May 01 12:14:43 localhost systemd[279234]: Listening on dbus.socket - D-Bus User Message Bus Socket.
May 01 12:14:43 localhost systemd[279234]: Listening on gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation).
May 01 12:14:43 localhost systemd[279234]: Reached target sockets.target - Sockets.
May 01 12:14:43 localhost systemd[279234]: Reached target basic.target - Basic System.
May 01 12:14:43 localhost systemd[279234]: Reached target default.target - Main User Target.
May 01 12:14:43 localhost systemd[279234]: Startup finished in 131ms.
May 01 12:19:49 localhost systemd[279234]: launchpadlib-cache-clean.service - Clean up old files in the Launchpadlib cache was skipped because of an unmet condition check (ConditionPathExists=/home/jccsah001012/.launchpadlib/api.launchpad.net/cache).
May 01 12:28:27 localhost systemd[279234]: Activating special unit exit.target...
May 01 12:28:27 localhost systemd[279234]: Stopped target default.target - Main User Target.
May 01 12:28:27 localhost systemd[279234]: Stopped target basic.target - Basic System.
May 01 12:28:27 localhost systemd[279234]: Stopped target paths.target - Paths.
May 01 12:28:27 localhost systemd[279234]: Stopped target sockets.target - Sockets.
May 01 12:28:27 localhost systemd[279234]: Stopped target timers.target - Timers.
May 01 12:28:27 localhost systemd[279234]: Stopped launchpadlib-cache-clean.timer - Clean up old files in the Launchpadlib cache.
May 01 12:28:27 localhost systemd[279234]: Closed dbus.socket - D-Bus User Message Bus Socket.
May 01 12:28:27 localhost systemd[279234]: Closed dirmngr.socket - GnuPG network certificate management daemon.
May 01 12:28:27 localhost systemd[279234]: Closed gpg-agent-browser.socket - GnuPG cryptographic agent and passphrase cache (access for web browsers).
May 01 12:28:27 localhost systemd[279234]: Closed gpg-agent-extra.socket - GnuPG cryptographic agent and passphrase cache (restricted).
May 01 12:28:27 localhost systemd[279234]: Stopping gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation)...
May 01 12:28:27 localhost systemd[279234]: Closed gpg-agent.socket - GnuPG cryptographic agent and passphrase cache.
May 01 12:28:27 localhost systemd[279234]: Closed keyboxd.socket - GnuPG public key management service.
May 01 12:28:27 localhost systemd[279234]: Closed pk-debconf-helper.socket - debconf communication socket.
May 01 12:28:27 localhost systemd[279234]: Closed snapd.session-agent.socket - REST API socket for snapd user session agent.
May 01 12:28:27 localhost systemd[279234]: Closed gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation).
May 01 12:28:27 localhost systemd[279234]: Removed slice app.slice - User Application Slice.
May 01 12:28:27 localhost systemd[279234]: Reached target shutdown.target - Shutdown.
May 01 12:28:27 localhost systemd[279234]: Finished systemd-exit.service - Exit the Session.
May 01 12:28:27 localhost systemd[279234]: Reached target exit.target - Exit the Session.
May 01 12:28:27 localhost (sd-pam)[279235]: pam_unix(systemd-user:session): session closed for user jccsah001012
May 01 12:29:51 localhost systemd[279397]: Queued start job for default target default.target.
May 01 12:29:51 localhost systemd[279397]: Created slice app.slice - User Application Slice.
May 01 12:29:51 localhost systemd[279397]: Started launchpadlib-cache-clean.timer - Clean up old files in the Launchpadlib cache.
May 01 12:29:51 localhost systemd[279397]: Reached target paths.target - Paths.
May 01 12:29:51 localhost systemd[279397]: Reached target timers.target - Timers.
May 01 12:29:51 localhost systemd[279397]: Starting dbus.socket - D-Bus User Message Bus Socket...
May 01 12:29:51 localhost systemd[279397]: Listening on dirmngr.socket - GnuPG network certificate management daemon.
May 01 12:29:51 localhost systemd[279397]: Listening on gpg-agent-browser.socket - GnuPG cryptographic agent and passphrase cache (access for web browsers).
May 01 12:29:51 localhost systemd[279397]: Listening on gpg-agent-extra.socket - GnuPG cryptographic agent and passphrase cache (restricted).
May 01 12:29:51 localhost systemd[279397]: Starting gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation)...
May 01 12:29:51 localhost systemd[279397]: Listening on gpg-agent.socket - GnuPG cryptographic agent and passphrase cache.
May 01 12:29:51 localhost systemd[279397]: Listening on keyboxd.socket - GnuPG public key management service.
May 01 12:29:51 localhost systemd[279397]: Listening on pk-debconf-helper.socket - debconf communication socket.
May 01 12:29:51 localhost systemd[279397]: Listening on snapd.session-agent.socket - REST API socket for snapd user session agent.
May 01 12:29:51 localhost systemd[279397]: Listening on dbus.socket - D-Bus User Message Bus Socket.
May 01 12:29:51 localhost systemd[279397]: Listening on gpg-agent-ssh.socket - GnuPG cryptographic agent (ssh-agent emulation).
May 01 12:29:51 localhost systemd[279397]: Reached target sockets.target - Sockets.
May 01 12:29:51 localhost systemd[279397]: Reached target basic.target - Basic System.
May 01 12:29:51 localhost systemd[279397]: Reached target default.target - Main User Target.
May 01 12:29:51 localhost systemd[279397]: Startup finished in 125ms.

auth_log_review.txt:
Apr 21 09:00:01 labhost sshd[1201]: Failed password for invalid user admin from 10.10.10.50 port 53122 ssh2
Apr 21 09:00:05 labhost sshd[1208]: Failed password for invalid user test from 10.10.10.50 port 53130 ssh2
Apr 21 09:00:11 labhost sshd[1215]: Failed password for jccsah001001 from 10.10.10.50 port 53144 ssh2
Apr 21 09:00:18 labhost sshd[1221]: Accepted password for jccsah001001 from 10.10.10.50 port 53155 ssh2
```
Meaning of the output: Basic monitoring is not just about collecting logs, but about understanding changes, failures, and indications of abnormal activity.

**Phase 5.2. Review a PCAP Sample with Tshark**
```
tshark -r ~/datasets/pcap/localhost_web_lab.pcap | head -n 20 | tee ~/module03/evidence/monitoring/pcap_review.txt
```

Function of the command:
- Review traffic samples without needing live packet capture.
- Introduce packet summary analysis as part of network observation.

Expected output:
- A frame summary from the `PCAP` is displayed.
- There are indications of traffic toward ports 3000 or 8081 if the PCAP file is available.
```
pcap_review.txt:
    1   0.000000    127.0.0.1 → 127.0.0.1    TCP 74 51306 → 3000 [SYN] Seq=0 Win=65495 Len=0 MSS=65495 SACK_PERM TSval=2922010966 TSecr=0 WS=128
    2   0.000021    127.0.0.1 → 127.0.0.1    TCP 74 3000 → 51306 [SYN, ACK] Seq=0 Ack=1 Win=65483 Len=0 MSS=65495 SACK_PERM TSval=2922010966 TSecr=2922010966 WS=128
    3   0.000036    127.0.0.1 → 127.0.0.1    TCP 66 51306 → 3000 [ACK] Seq=1 Ack=1 Win=65536 Len=0 TSval=2922010966 TSecr=2922010966
    4   0.000307    127.0.0.1 → 127.0.0.1    HTTP 143 GET / HTTP/1.1 
    5   0.000323    127.0.0.1 → 127.0.0.1    TCP 66 3000 → 51306 [ACK] Seq=1 Ack=78 Win=65408 Len=0 TSval=2922010966 TSecr=2922010966
    6   0.010115    127.0.0.1 → 127.0.0.1    TCP 14546 HTTP/1.1 200 OK  [TCP segment of a reassembled PDU]
    7   0.010133    127.0.0.1 → 127.0.0.1    TCP 66 51306 → 3000 [ACK] Seq=78 Ack=14481 Win=108416 Len=0 TSval=2922010976 TSecr=2922010976
    8   0.010164    127.0.0.1 → 127.0.0.1    TCP 51591 3000 → 51306 [PSH, ACK] Seq=14481 Ack=78 Win=65536 Len=51525 TSval=2922010976 TSecr=2922010976 [TCP segment of a reassembled PDU]
    9   0.010186    127.0.0.1 → 127.0.0.1    TCP 66 51306 → 3000 [ACK] Seq=78 Ack=66006 Win=77056 Len=0 TSval=2922010976 TSecr=2922010976
   10   0.010580    127.0.0.1 → 127.0.0.1    HTTP 9532 HTTP/1.1 200 OK  (text/html)
   11   0.010594    127.0.0.1 → 127.0.0.1    TCP 66 51306 → 3000 [ACK] Seq=78 Ack=75472 Win=118400 Len=0 TSval=2922010977 TSecr=2922010977
   12   0.010662    127.0.0.1 → 127.0.0.1    TCP 66 51306 → 3000 [FIN, ACK] Seq=78 Ack=75472 Win=118400 Len=0 TSval=2922010977 TSecr=2922010977
   13   0.012414    127.0.0.1 → 127.0.0.1    TCP 66 3000 → 51306 [FIN, ACK] Seq=75472 Ack=79 Win=65536 Len=0 TSval=2922010979 TSecr=2922010977
   14   0.012433    127.0.0.1 → 127.0.0.1    TCP 66 51306 → 3000 [ACK] Seq=79 Ack=75473 Win=118400 Len=0 TSval=2922010979 TSecr=2922010979
   15   0.018024    127.0.0.1 → 127.0.0.1    TCP 74 58088 → 8081 [SYN] Seq=0 Win=65495 Len=0 MSS=65495 SACK_PERM TSval=2922010984 TSecr=0 WS=128
   16   0.018037    127.0.0.1 → 127.0.0.1    TCP 74 8081 → 58088 [SYN, ACK] Seq=0 Ack=1 Win=65483 Len=0 MSS=65495 SACK_PERM TSval=2922010984 TSecr=2922010984 WS=128
   17   0.018048    127.0.0.1 → 127.0.0.1    TCP 66 58088 → 8081 [ACK] Seq=1 Ack=1 Win=65536 Len=0 TSval=2922010984 TSecr=2922010984
   18   0.018101    127.0.0.1 → 127.0.0.1    HTTP 143 GET / HTTP/1.1 
   19   0.018110    127.0.0.1 → 127.0.0.1    TCP 66 8081 → 58088 [ACK] Seq=1 Ack=78 Win=65408 Len=0 TSval=2922010984 TSecr=2922010984
   20   0.024162    127.0.0.1 → 127.0.0.1    HTTP 489 HTTP/1.1 302 Found
```
Meaning of the output: Analysts are often safer and more efficient working from existing capture files rather than capturing production traffic.

**Phase 5.3. Run YARA on Relevant Samples**
```
cat > ~/module03/tmp/basic_keywords.yar <<'EOF' 
rule suspicious_keywords { 
strings: 
$a = "urgent" nocase 
$b = "verify" nocase 
$c = "password" nocase 
condition: 
any of them 
}
EOF
yara ~/module03/tmp/basic_keywords.yar ~/datasets/phishing/sample_email_1.txt | tee ~/module03/evidence/monitoring/yara_result.txt
```

Function of the command:
- Introduce rule-based review on simple text artifacts.
- Demonstrate that YARA can be used early as an observation tool, without waiting for malware-focused labs.

Expected output:
- `YARA` results are displayed if the targeted strings are found.
```
suspicious_keywords /home/jccsah001012/datasets/phishing/sample_email_1.txt
```

## Module 03_Phase 6 — Automation for Parsing and Evidence Summarization

This phase serves as a direct bridge to scripting and automation modules. We are ask to build two complementary approaches: Bash for quick filtering and Python for more structured summaries.

**Phase 6.1. Bash Parser for Authentication Log Samples**
```
cat > ~/module03/tmp/auth_parser.sh <<'EOF' 
#!/usr/bin/env bash 
LOG=~/datasets/logs/linux/auth.log.sample 
echo "=== Failed logins ===" 
grep -i "Failed password" "$LOG" 
echo 
echo "=== Source IP summary ===" 
grep -i "Failed password" "$LOG" | awk '{print $(NF-3), $(NF-1)}' | sort | uniq -c
EOF
chmod +x ~/module03/tmp/auth_parser.sh
~/module03/tmp/auth_parser.sh | tee ~/module03/evidence/automation/bash_parser_output.txt
```

Function of the command:
- Train students to create small but useful and explainable parsers.
- Demonstrate that Bash is still highly relevant for quick triage tasks.

Expected output:
- A list of `failed logins and a summary of source IP addresses` are displayed.
```
=== Failed logins ===
Apr 21 09:00:01 labhost sshd[1201]: Failed password for invalid user admin from 10.10.10.50 port 53122 ssh2
Apr 21 09:00:05 labhost sshd[1208]: Failed password for invalid user test from 10.10.10.50 port 53130 ssh2
Apr 21 09:00:11 labhost sshd[1215]: Failed password for jccsah001001 from 10.10.10.50 port 53144 ssh2

=== Source IP summary ===
      1 10.10.10.50 53122
      1 10.10.10.50 53130
      1 10.10.10.50 53144
```
Meaning of the output:
Good automation does not have to be complex; it must be clear, useful, and purpose-driven.

**Phase 6.2. Python Parser for Authentication Log Samples**
```
cat > ~/module03/tmp/auth_parser.py <<'EOF'
import re
from collections import Counter
from pathlib import Path

log = Path.home() / 'datasets' / 'logs' / 'linux' / 'auth.log.sample'
pattern = re.compile(r'Failed password .* from (\S+)')
ips = Counter()

for line in log.read_text().splitlines():
    m = pattern.search(line)
    if m:
        ips[m.group(1)] += 1

print('failed_login_ip_summary=', dict(ips))
EOF
python3 ~/module03/tmp/auth_parser.py | tee ~/module03/evidence/automation/python_parser_output.txt
```

Function of the command:
- Build a more structured summary using Python.
- Familiarize students with working with files, regular expressions, and counters in a simple yet useful way.

Expected output:
- A summary of source IP addresses and their occurrence counts is displayed.
```
failed_login_ip_summary= {'10.10.10.50': 3}
```
Meaning of the output:
Python will become a primary tool in future modules. Here, students begin using it within the context of real evidence.

**Phase 6.3. Record Notes on Scheduled Automation Concepts and Their Limitations**
```
cat > ~/module03/evidence/automation/cron_notes.md <<'EOF' 
# Cron Notes
 - Cron dapat dipakai untuk task berkala seperti parsing log, checksum file, atau report 
generation.
 - Pada shared lab, student tidak melakukan perubahan sistem global.
 - Fokus modul ini adalah memahami konsep scheduling dan peran otomasi, bukan memodifikasi service sistem.
EOF
cat ~/module03/evidence/automation/cron_notes.md
```

Function of the command:
- Explain the role of automation in host operations without encouraging unnecessary system modifications.

Expected output:
- The `cron_notes.md` file is available.
```
# Cron Notes
 - Cron dapat dipakai untuk task berkala seperti parsing log, checksum file, atau report 
generation.
 - Pada shared lab, student tidak melakukan perubahan sistem global.
 - Fokus modul ini adalah memahami konsep scheduling dan peran otomasi, bukan memodifikasi service sistem.
```
Meaning of the output: We understand that automation must be used with discipline and within the defined scope.


## Module 03_Phase 7 — Final Assessment and Bridge to the Next Module

This module concludes with a brief assessment so that we become accustomed to transforming technical output into evaluations that can be understood by others. It also serves as a bridge to Python, automation, secure coding, and blue team analysis modules.

**Phase 7.1. Write a Brief Host Security Readiness Assessment**
```
cat > ~/module03/reports/os_security_assessment.md <<'EOF' 
# OS Security Assessment 
 
## Summary 
Ubuntu berfungsi sebagai primary analysis host dan Kali sebagai validation workstation. 
Baseline host, metadata file, process state, session information, logs, PCAP sample, dan 
automation dasar telah ditinjau. 
 
## Key observations
 - Host context and resource state collected
 - File metadata and permissions reviewed
 - Process and session information reviewed
 - Logs and PCAP sample analyzed
 - Bash and Python parsing completed 
 
## Operational takeaway 
Host analysis harus dimulai dari context, metadata, process, dan evidence, bukan asumsi. Validation dari Kali membantu melihat exposure dari sudut pandang yang berbeda.
EOF
cat ~/module03/reports/os_security_assessment.md
```

Function of the command:
- Build the habit of concluding practicum activities with concise assessments that can be read professionally.

Expected output:
- The `os_security_assessment.md` file is available.
```
# OS Security Assessment 
 
## Summary 
Ubuntu berfungsi sebagai primary analysis host dan Kali sebagai validation workstation. 
Baseline host, metadata file, process state, session information, logs, PCAP sample, dan 
automation dasar telah ditinjau. 
 
## Key observations
 - Host context and resource state collected
 - File metadata and permissions reviewed
 - Process and session information reviewed
 - Logs and PCAP sample analyzed
 - Bash and Python parsing completed 
 
## Operational takeaway 
Host analysis harus dimulai dari context, metadata, process, dan evidence, bukan asumsi. Validation dari Kali membantu melihat exposure dari sudut pandang yang berbeda.
```
Meaning of the output:
The value of an analyst is not limited to the commands they run, but also the quality of the assessments they can explain.

**Phase 7.2. Write Operating Notes as a Bridge to the Next Module**
```
cat > ~/module03/reports/operating_notes.md <<'EOF' 
# Operating Notes 
 
## What should carry forward
 - Evidence must stay organized
 - Ubuntu remains primary analysis host
 - Kali remains validation workstation
 - Bash and Python should be used together, not separately
 - Process, logs, and metadata should be correlated, not reviewed in isolation 
 
## Bridge to next module
 - Modul 04 akan memperluas scripting dan automation
 - Beberapa pola parsing di modul ini akan dipakai ulang pada log analysis, hunting, dan tool building
EOF
cat ~/module03/reports/operating_notes.md
```

Function of the command:
- Encourage students to see continuity between modules rather than viewing each module as isolated.

Expected output:
- The `operating_notes.md` file is available.
```
# Operating Notes 
 
## What should carry forward
 - Evidence must stay organized
 - Ubuntu remains primary analysis host
 - Kali remains validation workstation
 - Bash and Python should be used together, not separately
 - Process, logs, and metadata should be correlated, not reviewed in isolation 
 
## Bridge to next module
 - Modul 04 akan memperluas scripting dan automation
 - Beberapa pola parsing di modul ini akan dipakai ulang pada log analysis, hunting, dan tool building
```
Meaning of the output: We are positioned to enter the next module with a more mature context.

## Module 03_Phase 8 — Closing

Module 03 positions the operating system as a real object of observation. We not only read files and processes, but also learn to connect metadata, permissions, sessions, services, logs, and traffic samples into a unified context. With this foundation, the next modules can move further into scripting, automation, secure coding, blue team analysis, and threat hunting without needing to return to the fundamentals of host observation.
