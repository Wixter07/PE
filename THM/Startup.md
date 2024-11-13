# Startup

``` Target IP - 10.10.182.75```

## nmap

```bash
wixter07@HARSHITH:~$ nmap -sV 10.10.182.75
Starting Nmap 7.80 ( https://nmap.org ) at 2024-11-14 03:35 IST
Nmap scan report for 10.10.182.75
Host is up (0.20s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 163.61 seconds
```

ftp it is. Maybe the usual anonymous login would work.

![image](https://github.com/user-attachments/assets/9d65adfa-7695-4079-b769-685af01f0803)

Just press enter when promted for password. Now let's analyze

## Analysis


