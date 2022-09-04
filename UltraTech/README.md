UltraTech: Write-Up (TryHackMe)
September 4, 2022 Jarrod

This is a WriteUp on how to complete the room UltraTech on TryHackMe.

Note\* I used Kali Linux to complete this room. The IP address of my room was 10.10.26.7, so that will be the IP you see in the write-up.

- Click on images to enlarge.

Let’s begin this room by deploying the virtual machine. We have nine questions we need to answer to finish this room.

Now that the virtual machine is running, we can begin enumerating and gathering information about the server by performing an Nmap scan to see which ports are open. This will help us answer the first few questions.
Nmap

Running the command:
nmap -sV -p- -Pn 10.10.26.7 -oA nmap_scan shows ports 21,22,8081, and 31331 are open.

How this works:
nmap – The command used to execute Nmap.
-sV – This means Nmap will run a service/version detection scan.
-p- – This tells Nmap to check all ports.
-Pn – This tells nmap to not ping the server to check to see if it is up and to just port scan.
-oA nmap_scan – This means we want to save the nmap output in a gnamp file, nmap file, and xml file.

This scan will answer the first few questions on TryHackMe.

Task 2 Question 1: Which software is using the port 8081? Node.js

Task 2 Question 2: Which other non-standard port is used? 31331

Task 2 Question 3: Which software using this port? Apache

Task 2 Question 4: Which GNU/Linux distribution seems to be used? Ubuntu

Let’s take a look at the FTP port and see if anonymous login works.

Trying to run ftp 10.10.26.7 and supplying a user of anonymous and an empty password doesn’t seem to work. If we need to we can come back to this later. For now let’s check out the website on port 31331.

Nothing special at first glance. Looks to be a generic website.

Let’s now check out port 8081. Here we can see UltraTech API v0.1.3.

Let’s use FFUF (or whatever dir/file fuzzer you like) to enumerate files, routes, and directories to try and find some clues.

Running the command:
ffuf -u http://10.10.26.7:31331/FUZZ -w /usr/share/wordlists/SecList/Discovery/Web-Content/directory-list-2.3-medium.txt shows output for images, js, javascript, and server-status for the website running on port 31331.

Running the command:
ffuf -u http://10.10.26.7:8081/FUZZ -w /usr/share/wordlists/SecList/Discovery/Web-Content/directory-list-2.3-medium.txt shows output for auth and ping for the website running on port 8081.

How this works:
ffuf – The command used to execute ffuf.
-u – Tells ffuf what url to fuzz.
-w – Tells ffuf what wordlist to use.
FUZZ is important as it tells ffuf where to fuzz in the url.

Let’s take a look at some of the folders on 31331.

Turns out /js has a few interesting files located in it.

The api.js file appears to have the ability to perform remote code execution via the http://10.10.26.7:8081/ping?ip= call.

Navigating to http://10.10.26.7:8081/ping?ip reveals the message “Invalid ip parameter specified”. Let’s try and ping the localhost by providing the param ?ip=127.0.0.1 to see what the output would be.

Looks like we were successful in pinging the localhost of the server. We have a very simple form of remote code execution. Let’s try and see what else we can do.

Trying to do a ls displays the following message. Looks like we might need to try a few tricks to get this to work.

Turns out we can use backticks to perform commands. The backtick has a very special meaning. Everything you type between backticks is evaluated (executed) by the shell before the main command is performed. In this case we can do ?ip=`ls` and see the output displays a sqlite file.

This helps answer our next question on TryHackMe.
Task 3 Question 1: There is a database lying around, what is its filename? utech.db.sqlite
Let’s try and cat the file by doing ip=`cat utech.db.sqlite`

This displays the output from the sqlite file. We have two users from this.

root with a hash of f357a0c52799563c7c7b76c1e7543a32

admin with a hash of 0d0ea5111e3c1def594c1684e3b9be84

Taking these to Crackstation, they are easily cracked and we now have passwords.

We can also answer the next question on TryHackMe.

Task 3 Question 2: What is the first user’s password hash? f357a0c52799563c7c7b76c1e7543a32
Task 3 Question 3: What is the password associated with this hash? n100906

Let’s try to SSH in as r00t using our new password.

We have successfully logged in as r00t, but we are not the root yet. Let’s do some basic enumerating such as sudo -l, groups, id, whoami, and checking our home directory.

The most interesting thing we see is we are in the Docker Group.

knowing our user r00t is in the docker groups, let’s take a trip to GTFObins and see if we can abuse this. Turns out a user in the Docker group can break out into a root system shell.

Abusing this, we can become root by running docker run -v /:/mnt –rm -it bash chroot /mnt sh. The example in GTFObins is using alpinie and not bash, remember to use bash or the command will not work.

With that we are now root and can obtain the final bits of information to complete the room and get the SSH Private key.

That completes the room! Well done! If you found this helpful, please send me a tweet and tell me what you thought! Feedback is always appreciated!

Jarrod
