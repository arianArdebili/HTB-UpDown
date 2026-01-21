Executive Summary
This repository contains my technical research and the full-chain exploitation of the UpDown environment. This mission required a deep understanding of web architecture, source code forensics, and legacy logic flaws. The compromise was achieved by chaining an SSRF vulnerability to leak source code, bypassing file upload filters using PHP stream wrappers, and escalating privileges through a legacy Python 2.7 logic bug.
Target Intelligence
IP Address: 10.129.x.x (Target-Specific)
Hostname: siteisup.htb | dev.siteisup.htb
OS: Ubuntu Linux
Primary Services: Apache 2.4.41, OpenSSH 8.2p1, PHP 8.0.20
The Kill Chain
1. Reconnaissance & Virtual Host Discovery
Initial scans identified a standard web interface. Analysis of the application footer revealed a custom hostname, leading to the discovery of a restricted Development Subdomain via VHost fuzzing.
Vulnerability: Information Leakage (Hostname) & Insecure Virtual Host configuration.
Action: Bypassed 403 Forbidden via custom HTTP headers identified in configuration forensics.
2. Git Forensics & Source Code Leakage
Identified an exposed .git directory on the development subdomain. I utilized git-dumper to reconstruct the application’s source code.
Loot: Discovered the internal logic for the "Site Checker" and identified a blacklist-based file upload filter.
3. Web Exploitation: PHAR/ZIP SSRF-to-LFI Chain
By analyzing the PHP source code, I identified an omission in the extension blacklist (allowing .phar) and a vulnerable include() function.
Savage Move: Crafted a malicious PHP shell inside a ZIP archive (disguised as .phar) and triggered it using the phar:// stream wrapper via a Race Condition exploit to bypass immediate file deletion (unlink).
Result: Obtained initial shell access as www-data.
4. Privilege Escalation (User: developer)
Local enumeration identified a custom SUID binary /home/developer/dev/siteisup.
Vulnerability: Command Injection in Python 2.7 input() function.
Exploit: Injected a Python breakout payload into the interactive prompt to hijack the process owner's identity.
Result: Escalated to user developer.
5. Privilege Escalation (User: root)
Analysis of Sudo permissions (sudo -l) revealed NOPASSWD rights for /usr/local/bin/easy_install.
Exploit: Leveraged GTFOBins logic to hijack the setup.py installation process, spawning a privileged root shell.
Result: Full System Compromise.
🛠️ Custom Tooling & Patches
In this mission, I didn't just use tools; I fixed them.
dfunc-bypasser-patched.py: I manually patched the original dfunc-bypasser script to handle raw text input and inject the required custom headers for the Updown environment.
proc_open_revshell.php: A surgical PHP payload designed to bypass heavily restricted disable_functions environments.
