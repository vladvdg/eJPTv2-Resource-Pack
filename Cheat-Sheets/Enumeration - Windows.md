# Metasploit modules

With a Meterpreter session, we can use post-exploitation modules to automate information enumeration.

```shell
use post/windows/gather/win_privs # Enumerates current user privileges
use post/windows/gather/enum_logged_on_users # Lists current and previously logged-in users
use post/windows/gather/checkvm # Detects if the system is a virtual machine
use post/windows/gather/enum_applications # Lists installed applications on the target
use post/windows/gather/enum_computers # Enumerates other computers on the same LAN
use post/windows/gather/enum_shares # Lists shared folders accessible from the target
use post/windows/gather/enum_av # Detects installed antivirus software
use post/windows/gather/enum_patches # Lists installed Windows patches
```

<br>

# JAWS (Just Another Windows Enum Script)

**[JAWS](https://github.com/411Hall/JAWS)** (Just Another Windows Enum Script) is an open-source PowerShell script designed to help penetration testers automate local enumeration and identify privilege escalation vectors on Windows systems.

```shell
# Upload the script on the target machine
upload /root/jaws-enum.ps1

# After uploading the script successfully, we will need to spawn a command shell session
shell
# We can now execute the jaws-enum.ps1 script
powershell.exe -ExecutionPolicy Bypass -File .\jaws-enum.ps1 -OutputFilename JAWS-Enum.txt
# jaws-enum.ps1 script will run and save the results into a file called JAWS-Enum.txt
# Note: JAWS will take a couple of minutes to complete the enumeration process
```

<br>

# System Info

```shell
sysinfo # Enumerating System Information

hostname # Enumerate the hostname of the system by spawning a native shell session on the target system

# We can also get a list of installed updates on the system with more detailed information like when the update was installed by running the following command.
wmic qfe get Caption,Description,HotFixID,InstalledOn

# File that can provide information pertinent to the operating system version and also provide with additional information like the build number and the service pack. This file is not always available.
cd C:\\Windows\\System32
cat eula.txt 

systeminfo # Comprehensive information regarding the target system
whoami # Get current user
whoami /priv # Get current user privileges
net localgroup <user> # Get group membership of user
net user <user> # Get user info
getuid # Enumerate the current users we have access to on the target system
getprivs # Enumerate the Windows privileges that the Administrator user has

# Check the list of currently logged-on users
background
use post/windows/gather/enum_logged_on_users
set SESSION 1
run
```

<br>

# Network Info

```shell
# Display the system's network configuration details, including IP addresses, subnet masks, and default gateways for all network interfaces
ipconfig
ipconfig /all

route print # Routing table

# This command in Windows shows the ARP cache, listing IP addresses and their associated MAC addresses recently resolved on the local network
arp -a # This information is going to be important during the Pivoting phase

netstat -ano # View a list of open ports being used by services on the target system

# Display the current firewall settings for all network profiles (Domain, Private, and Public), including their state (on/off), inbound/outbound rules, and logging configurations
netsh firewall show state
# OR
netsh advfirewall show allprofiles
```

<br>

# Processes & Services

```shell
ps # Enumerate a list of running processes on the Windows target
net start # We can enumerate a list of running services 
net stop <servicename> # Stop a service 
wmic service list brief # We can learn more about the running services
tasklist /svc # We can also enumerate a list of running tasks and the corresponding services for each task

# List of scheduled tasks on the Windows target
schtasks /query /fo list /v # A lot of info so we need to copy paste in a document separately
# This information is important because in certain cases, scheduled tasks could be misconfigured or configured in a way that makes them vulnerable to exploitation, and most specifically, they can be exploited in some cases to elevate our privilege

netstat -tuln 127.0.0.1
netstat -ano | findstr 127.0.0.1
# Enumerate local services

nc 127.0.0.1 8888
# Interact with the service
```
