Bolt WriteUp
============

[August 31, 2021](https://jarrodrizor.com/bolt-writeup/ "7:03 pm") [Jarrod](https://jarrodrizor.com/author/jarrod/ "View all posts by Jarrod")

This is a WriteUp on how to complete the room [Bolt](https://tryhackme.com/room/bolt) on [TryHackMe](https://tryhackme.com).

![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Banner.png)

Note* I used Kali Linux to complete this room. The IP address of my room was 10.10.144.210, so that will be the IP you see in the writeup. Replace 10.10.144.210 with the IP of your target box.

Let's begin this room by deploying the virtual machine.

Nmap
====

Now that it is running, we can begin enumerating and gathering information about the server by doing an Nmap scan to see which ports are open and what services are on the ports.

Running the command:\
**sudo nmap -sV -p- -v -T4 -oN nmap_scan 10.10.144.210** shows ports 22, 80, and 8000 are open and running SSH 7.6p1 and Apache 2.4.29 on an Ubuntu Server and a PHP server on port 8000.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_nmap.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_nmap.png)

How this works:\
nmap -- The command used to execute Nmap.\
-sV -- This means Nmap will run a service/version detection scan.\
-p- -- Will check all ports.\
-v -- shows Verbose output.\
-T4 -- This is the timing option. T4 is a good mix of speed and accuracy for CTF's.\
-oN nmap_scan -- This means we want to save the output to a file named nmap_scan for future use.

Here we can see the answer to our first question.

**What port number has a web server with a CMS running?**

*Answer -- Port 8000.*

Let's visit the site and see what we can learn about it.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_homepage.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_homepage.png)

Navigating to http://10.10.144.210:8000 shows us this basic website. The bottom left of the web page does show us something very important. The name of the CMS that runs the website is called Bolt.

We can click around and see if we find anything useful. Clicking on the link Message for IT Department shows us a post by the user bolt and the password boltadmin123. This information can help us answer two more questions on TryHackMe.

**What is the username we can find in the CMS?**

*Answer -- bolt*

**What is the password we can find for the username?**

*Answer -- boltadmin123*

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_It_Message.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_It_Message.png)

Doing some Google searching we can find that the login page for the Bolt CMS is http://10.10.144.210:8000**/bolt/login.** Here we can see a login panel. Let's use our newly acquired username and password.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Login.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Login.png)

We are now on the Admin Dashboard.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Admin_Dashboard.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Admin_Dashboard.png)

Here we can see the answer to the next question about the CMS. Note* The room doesn't go into more detail about needing the dashboard. We just want to get the version of Bolt; knowing the credentials will help us exploit the server using Metasploit.

**What version of the CMS is installed on the server? (Ex: Name 1.1.1)**

*Answer -- Bolt 3.7.1*

Metasploit
==========

With this information we can do more research and find helpful information on exploiting the machine. Using Google we can search for Bolt 3.7.1 Exploits. This should lead us to a link to the following page on [exploit-db](https://www.exploit-db.com/exploits/48296).

**There's an exploit for a previous version of this CMS, which allows authenticated RCE. Find it on Exploit DB. What's its EDB-ID?**

*Answer -- 48296*

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_edb-id.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_edb-id.png)

Let's open up Metasploit and see if we can find a way to help automate our exploitation process. Using the command **msfconsole -q** (-q means do not print the banner) we can start Metasploit. Let's first search for bolt and we what we can find. Let's use the command **search bolt** and hit enter.

This shows us two options. Option 1 shows the version of bolt that we are pretty close to. Let's give that a shot. Let's type **use 1** in Metasploit to use the exploit **exploit/unix/webapp/bolt_authenticated_rce**. Typing **options** will show what parameters we can fill. What we are looking at is the **RHOSTS, RPORT, PASSWORD, USERNAME, LHOST, and LPORT**.

Let's go over each one.

**RHOSTS** -- The target virtual machine. In this case 10.10.144.210.

**RPORT** -- The port the service is running on. Bolt runs on 8000 in our case, so we don't need to change this.

**PASSWORD** -- The password we used to log in with. This was boltadmin123.

**USERNAME** -- The username we used to log in with. This was bolt.

**LHOST** -- Our virtual machines interface IP address. (run **ip addr** if you need to find yours)

**LPORT** -- What we are using to listen to. Our default is 4444. This will work just fine.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_metasploit_exploit.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_metasploit_exploit.png)

After getting these values set let's run options again to make sure everything looks correct.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_metasploit_options.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_metasploit_options.png)

Now we need to run the command **exploit** to begin attacking the machine. We can see the results from the exploit as it's performing the remote code execution.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Becoming_Root-1.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Becoming_Root-1.png)

We now have a reverse shell to the machine and lucky for us, we are root!

Let's run **python3  -c 'import pty; pty.spawn("/bin/bash")'** to spawn a TTY Shell.

Now that we know the exploit worked we can answer the next question.

**Metasploit recently added an exploit module for this vulnerability. What's the full path for this exploit? (Ex: exploit/....)**

*Answer -- exploit/unix/webapp/bolt_authenticated_rce*

**Look for flag.txt inside the machine.**

We can **cd** into the /home directory and find the flag. Running the **cat** command will display the contents of the file and all we need to do is copy and paste it to TryHackMe.

[![](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Flag.png)](https://jarrodrizor.com/wp-content/uploads/2021/08/Bolt_Flag.png)

That completes the room! Well done! If you found this helpful, please send me a [tweet](https://twitter.com/JarrodR87) and tell me what you thought! Feedback is always appreciated!

Jarrod
