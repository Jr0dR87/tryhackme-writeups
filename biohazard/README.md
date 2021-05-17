1 Enumeration.
Running nmap 
nmap -sV -T4 -A -oA biohazard_nmap 10.10.174.74
results
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-17 18:22 EDT
Nmap scan report for 10.10.174.74
Host is up (0.11s latency).
Not shown: 997 closed ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c9:03:aa:aa:ea:a9:f1:f4:09:79:c0:47:41:16:f1:9b (RSA)
|   256 2e:1d:83:11:65:03:b4:78:e9:6d:94:d1:3b:db:f4:d6 (ECDSA)
|_  256 91:3d:e4:4f:ab:aa:e2:9e:44:af:d3:57:86:70:bc:39 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Beginning of the end
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.76 seconds

Running gobuster
gobuster dir -u http://10.10.174.74 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o root_gobuster
gobuster dir -u http://10.10.174.74/mansionmain -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o mansionmain_gobuster


