This is a WriteUp on how to complete the room [Brute It](https://tryhackme.com/room/bruteit) on [TryHackMe](https://tryhackme.com).

Note* I used Kali Linux to complete this room. The IP address of my room was 10.10.143.205, so that will be the IP you see in the writeup. Replace 10.10.143.205 with the IP of your target box.

**Task 1: About this box.**

In this box you will learn about:\
-- Brute-force\
-- Hash cracking\
-- Privilege escalation

Connect to the TryHackMe network, and deploy the machine.

The first task is to deploy the VM.

This concludes Task 1 About this box.

**Task 2: Reconnaissance**

Our second task is doing some reconnaissance. We will use Nmap and perform some port scanning and Gobuster to find hidden directories.

Running the command:\
nmap -sV -oN BruteItnmapScan 10.10.143.205 displayed the following information.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItNmapScan.png)

How this works:\
nmap -- The command used to execute Nmap.\
-sV -- This means Nmap will run a service/version detection scan. These scans can take a while to run, but they return information about what services are running on each open port and what version they are.\
-oN BruteItnmapScan -- This means we want to save the output to a file named BruteItnmapScan.

Question 1: How many ports are open? 2. SSH & HTTP.\
Question 2: What version of SSH is running? OpenSSH 7.6p1\
Question 3: What version of Apache is running? 2.4.29\
Question 4: Which Linux distribution is running? Ubuntu

Running the command:\
gobuster dir -u http://10.10.143.205 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o BruteItGobusterScan displays the hidden directory /admin.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItGobusterScan.png)

How this works:\
gobuster -- The command to execute GoBuster.\
dir -- (scan for directories).\
-u -- Target URL.\
-w -- the wordlist we are using to scan for hidden directories. In this case, I used the wordlist in Dirbuster called directory-list-2.3-small.txt.\
-o -- saves the output to a file. In this case, we are saving to BruteItGobusterScan.

Question 5. What is the hidden directory? /admin

We can navigate to the root of the website, but all we find is the Apache page.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItRootWebPageScreenShot.png)

This concludes Task 2 Reconnaissance

**Task 3: Getting a shell.**

Navigating to http://10.10.143.205/admin shows a login page.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItAdminPage.png)

If we view the page source code we will find a comment letting us know about someone named John and a username admin.\
![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItAdminPageSource.png)

Going back to the login page and trying to brute force in with simple password attempts. admin:password, admin:password123, or admin:admin does not work. We are getting Username or password invalid.

Let's use Hydra and speed this up.

Firefox and Chrome have a Network Console to pull out the Request needed for Hyrda. We could get this with Burp Suite, but this is faster.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItNetworkRequest.png)

Also on the page, the words "Username or password invalid" will be needed so Hyrda knows that the password attempt was incorrect and to try the next password.

Running the command:\
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.143.205 http-post-form "/admin/:user=^USER^&pass=^PASS^:Username or password invalid" display the following information.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItHydraScreenshot.png)

How this works:\
hydra -- The command used to execute Hydra.\
-l -- This is the user we want to authenticate as.\
-P -- The wordlist we want to use when brute forcing.\
10.10.143.205 -- This is the target website. (You will switch out with the IP of your target machine.)\
http-post-form -- Lets Hydra know we are attempting to use Brute Force by\
POSTing to a form.\
"/admin/:user=^USER^&pass=^PASS^:Username or password invalid" -- Let's break this down.\
/admin/ is the target page.\
user=^USER&pass=^PASS^ are the params the form is using to pass data into the form via POST. The ^USER^ and ^PASS^ are being used by the -l and -P part of the hydra command to fill in the username and password.\
The final section is telling Hydra if you see this on the page, keep trying because this isn't the output we want.

Question 1: What is the user:password of the admin panel? admin:xavier

We now know the login and password. We can get into the admin page.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItAdminLoggedIn.png)

This displays a page with the Web Flag that we can copy and paste into Question 4 of Getting a Shell. Web Flag. It's out of order at the time of writing this post, but you can copy and paste the flag now into Question 4. Web flag.

The page also has a link to download John's RSA private key that allows John to SSH into the machine. The catch is the key is password protected and we need to find out what it is with John the Ripper.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItJohnKeyScreenshot.png)

First we have to convert the RSA key into something John will understand.\
Running the command:\
/usr/share/john/ssh2john.py id_rsa > sshhash\
will give John a hash that it can understand in the form of a hash.

Now we can run the command:\
john --wordlist=/usr/share/wordlists/rockyou.txt sshhash

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItJohn.png)

How this works:

john -- The command to execute John.

--wordlist=/usr/share/wordlists/rockyou.txt sshhash -- Using the rockyou.txt, john is brute-forcing and trying to find a matching hash. Once it finds one it will display in the terminal.

We now have the password.

Question 2: Crack the RSA key you found. What is John's RSA Private Key passphrase? rockinroll

SSH is now possible with the user John.

Running the command: ssh -i id_rsa john@10.10.143.205\
will prompt for a password for the id_rsa key.\
Entering rockinroll will get us in and we can cat the user.txt to get the flag.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItUserFlag.png)Question 3. user.txt. Simply cat the file!

Question 4. web flag. This was on the admin panel with the link to the RSA key.

This concludes Task 3 Getting a shell

**Task 4 Privilege Escalation**

The first thing to check is what sudo abilities John has.\
Running the command:\
sudo -l\
shows John can use **cat** as sudo.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItJohnsudocommandsScreenshot.png)\
Let's check gtfobins and see what we can find a simple way to become root from this. Navigating to [gtfobins.github.io](https://gtfobins.github.io/gtfobins/cat/#sudo) shows us we can read from a file as root.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItCatSudo.png)

What we could do is read the /etc/shadow and get the root hash and crack that with John the Ripper.

Let's run the commands:\
LFILE=/etc/shadow\
sudo cat "$LFILE"

and we should get the following output.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItShadowFile.png)

We now have the root hash. Let's take it back to our Attack machine and run John the Ripper on it.

Similar to what we used before. We will run the command:

john --wordlist=/usr/share/wordlists/rockyou.txt roothash\
(roothash is the saved sha256 hash from the /etc/shadow file)

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItRootPassword.png)

Question 1: What is the root's password? football

We can now **su --** as john and when prompted give the password football and we are now root and can get the root flag.

![](https://jarrodrizor.com/wp-content/uploads/2021/04/BruteItRootFlag.png)

That completes the room! Well done! If you found this helpful, please send me a [tweet](https://twitter.com/JarrodR87) and tell me what you thought! Feedback is always appreciated!

Jarrod
