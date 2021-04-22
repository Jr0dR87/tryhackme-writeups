Task 1. About this box.
 - Brute-Force
 - Hash cracking
 - Privilege Escalation

step 1. Deploy Machine.
Start Machine. This completed Task 1.

Task 2. Reconnaissance

Nmap Port/Version Scan
Running nmap -sV -oN BruteItnmapScan 10.10.143.205 displayed the following information.

This target has 2 ports open. Answer to first question.
The version of SSH is OpenSSH 7.6p1. Answer to second question.
The version of Apache is 2.4.29. Answer to third question.
The Linux distribution is Ubuntu. Answer to fourth question.

Gobuster Directory Scan
Running gobuster dir -u http://10.10.143.205 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o BruteItGobusterScan displays that the answer to the last question, what is the hidden directory is 
/admin.

Navigating to http://10.10.143.205/admin shows a login page. I viewed the page source and saw a comment at the bottom of the HTML.     
<!-- Hey john, if you do not remember, the username is admin -->

I tried to brute force in with simple password attempts. admin:password, admin:password123, admin:admin. None worked.

Let's us Hydra and speed this up. 

Firefox and Chrome have a Network Console to pull out the Request needed for Hyrda. Also on the page, the words "Username or password invalid" will be needed so Hyrda knows that the password attempt was incorrect and to try the next.

Running hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.143.205 http-post-form "/admin/:user=^USER^&pass=^PASS^:Username or password invalid"

This gave us the password xavier. We can now login.

Task 3 Getting a shell.

Question 1 What is the user:password of the admin panel?
admin:xavier

Question 2 What is 
Crack the RSA key you found.
What is John's RSA Private Key passphrase?

Let's us John the Ripper to find out so we can SSH into the machine.
First we have to convert the RSA key into something John will understand.
/usr/share/john/ssh2john.py id_rsa > sshhash will give John a hash that it can understand.

Now we can run john --wordlist=/usr/share/wordlists/rockyou.txt sshhash and after a few seconds, we have the password rockinroll

SSH is now possible. 

Running 
ssh -i id_rsa john@10.10.143.205 will prompt a password. Entering rockinroll will get us in and we can cat the user.txt to get the flag.

The first thing to check is what sudo abilities does John have. John can use cat as sudo. Let's check gtfobins and see what we can exploit. 
https://gtfobins.github.io/gtfobins/cat/#sudo
gtfobins gives us a way to read the /etc/shadow file and similar to how we got John's password with John (ironic) we can do the same to the root password.

As John, running 
LFILE=/etc/shadow and then sudo cat "$LFILE"
gives the output of the shadow file and with that, the hash for the root password.
Using John again
john --wordlist=/usr/share/wordlists/rockyou.txt roothash
gives the password football.

We can now su - and when prompted give the password football and we are now root and can get the root flag.

This concludes the write up.
