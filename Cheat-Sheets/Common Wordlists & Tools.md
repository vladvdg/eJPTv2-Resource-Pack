# Username & Password Wordlists

- `/usr/share/wordlists/rockyou.txt`  
    → One of the most popular password lists for brute-forcing.  
    _(Extract first: `gzip -d /usr/share/wordlists/rockyou.txt.gz`)_
    
- `/usr/share/metasploit-framework/data/wordlists/common_passwords.txt`  
    → Contains frequently used passwords — ideal for fast brute-force attempts.
    
- `/usr/share/metasploit-framework/data/wordlists/unix_passwords.txt`  
    → Unix/Linux-specific passwords.
    
- `/usr/share/metasploit-framework/data/wordlists/common_users.txt`  
    → List of common usernames (useful with Hydra, SMB, FTP, etc.).
    
- `/usr/share/metasploit-framework/data/wordlists/unix_users.txt`  
    → Unix/Linux user accounts — pair with `unix_passwords.txt` for targeted attacks.
    
- `/usr/share/wordlists/metasploit/root_userpass.txt`  
    → Default/root user-password combinations — ideal for SSH or Telnet brute-force.
    
- `/usr/share/commix/src/txt/usernames.txt`  
    → Used for testing login forms or command injection username discovery.

<br>

# Directories & Files Enumeration

- `/usr/share/metasploit-framework/data/wordlists/directory.txt`  
    → Basic list of web directories — works well with `dir_scanner`.
    
- `/usr/share/dirb/wordlists/common.txt`  
    → Frequently used for `dirb` and `gobuster`; includes common directories and files.
    
- `/usr/share/wordlists/dirb/common.txt`  
    → Alternative location for the same `dirb` wordlist.
    
- `/usr/share/metasploit-framework/data/wordlists/sensitive_files.txt`  
    → Tries to find sensitive files (e.g., `.env`, `config.php`, `db.sql`).

<br>

# Web Shells (Post-Exploitation)

- `/usr/share/webshells/php/php-reverse-shell.php`  
    → PHP reverse shell — upload this to gain a shell from LFI/RFI or insecure upload.
    
- `/usr/share/webshells/asp/`  
    → ASP reverse shells — useful for IIS targets, especially during WebDAV or `rejetto_hfs` exploitation.
    
- `/usr/share/webshells/jsp/`  
    → JSP reverse shells — often used with Tomcat exploits like `jsp_upload_bypass`.

<br>

# Windows Post-Exploitation Binaries

- `/usr/share/windows-resources/binaries/`  
    → Useful Windows binaries for file transfer or reverse shells:  
    `nc.exe`, `wget.exe`, `ncat.exe`, `PowerUp.ps1`, `PowerShell Empire`, etc.

<br>

# Common Payloads

### Windows Payloads

- `windows/meterpreter/reverse_tcp`  
    → Full-featured Meterpreter reverse shell (32-bit systems).
    
- `windows/x64/meterpreter/reverse_tcp`  
    → Same as above, for 64-bit Windows systems.
    
- `windows/shell/reverse_tcp`  
    → Simple reverse shell without Meterpreter features.
    
- `windows/shell_bind_tcp`  
    → Binds a shell to the target — attacker connects in.

<br>

### Linux Payloads

- `linux/x86/meterpreter/reverse_tcp`  
    → Reverse Meterpreter shell for Linux.
    
- `linux/x64/shell/reverse_tcp`  
    → Reverse shell for 64-bit Linux systems.
    
- `cmd/unix/reverse_bash`  
    → Bash-based reverse shell (useful in RCE or command injection).
    
- `cmd/unix/reverse_python`  
    → Python reverse shell — works well if Python is available on target.
    
- `cmd/unix/reverse_perl`  
    → Perl-based reverse shell — often usable on older systems.

<br>

### Web-Based Payloads

- `php/meterpreter/reverse_tcp`  
    → Meterpreter shell through a PHP upload.
    
- `php/reverse_php`  
    → Basic PHP reverse shell (non-Meterpreter).
    
- `asp/reverse_tcp`  
    → Basic ASP reverse shell — works with IIS targets.
    
- `jsp/reverse_tcp`  
    → JSP shell for Java-based targets like Tomcat.

<br>

### Multi-Platform

- `multi/handler`  
    → Listener module used to catch incoming reverse shells (match to payload used).
    
- `multi/shell/reverse_tcp`  
    → Basic reverse TCP shell that works across different platforms.
