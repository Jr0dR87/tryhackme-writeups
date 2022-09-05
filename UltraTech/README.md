This is a WriteUp on how to complete the room <a href="https://tryhackme.com/room/ultratech1">UltraTech</a> on TryHackMe.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_banner.png"><img class="aligncenter size-full wp-image-825" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_banner.png" alt="" width="1278" height="228" /></a>

Note* I used Kali Linux to complete this room. The IP address of my room was <span id="machine-ip">10.10.26.7</span>, so that will be the IP you see in the write-up.

<strong>* Click on images to enlarge. </strong>

Let’s begin this room by deploying the virtual machine. We have nine questions we need to answer to finish this room.

Now that the virtual machine is running, we can begin enumerating and gathering information about the server by performing an Nmap scan to see which ports are open. This will help us answer the first few questions.

Running the command:
<strong>nmap -sV -p- -Pn 10.10.26.7 -oA nmap_scan </strong>shows ports 21,22,8081, and 31331 are open.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_nmap.png"><img class="aligncenter size-full wp-image-826" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_nmap.png" alt="" width="1145" height="363" /></a>
How this works:

nmap – The command used to execute Nmap.
-sV – This means Nmap will run a service/version detection scan.
-p- - This tells Nmap to check all ports.
-Pn - This tells nmap to not ping the server to check to see if it is up and to just port scan.
-oA nmap_scan – This means we want to save the nmap output in a gnamp file, nmap file, and xml file.
This scan will answer the first few questions on TryHackMe.
<div class="room-task-question-details">

<strong>Task 2 Question 1: Which software is using the port 8081? </strong>Node.js
<div class="room-task-question-details"><strong>Task 2 Question 2: Which other non-standard port is used? </strong>31331<strong>
</strong>
<div class="room-task-question-details">

<strong>Task 2 Question 3:</strong> <strong>Which software using this port? </strong>Apache
<div class="room-task-question-details"><strong>Task 2 Question 4: Which GNU/Linux distribution seems to be used? </strong>Ubuntu</div>
<div class="room-task-question-details">Let's take a look at the FTP port and see if anonymous login works.</div>

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ftp_failed_anonymous_login.png"><img class="aligncenter size-full wp-image-824" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ftp_failed_anonymous_login.png" alt="" width="628" height="244" /></a>

Trying to run ftp 10.10.26.7 and supplying a user of anonymous and an empty password doesn't seem to work. If we need to we can come back to this later. For now, let's check out the website on port 31331.
There is nothing special here, at first glance. It looks to be a generic website.</div>

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_website.png"><img class="aligncenter size-full wp-image-827" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_website.png" alt="" width="1617" height="1121" /></a>
Let's now check out port 8081. Here we can see UltraTech API v0.1.3.</div>

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_api.png"><img class="aligncenter size-full wp-image-828" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_api.png" alt="" width="932" height="181" /></a>

<div class="room-task-question-details">Let's use FFUF (or whatever dir/file fuzzer you like) to enumerate files, routes, and directories to try and find some clues.
Running the command:
<strong>ffuf -u http://10.10.26.7:31331/FUZZ -w /usr/share/wordlists/SecList/Discovery/Web-Content/directory-list-2.3-medium.txt</strong> shows output for images, js, javascript, and server-status for the website running on port 31331.</div>
<div class="room-task-question-details">

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ffuf_31331.png"><img class="aligncenter size-full wp-image-829" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ffuf_31331.png" alt="" width="1622" height="1067" /></a>

Running the command:
<strong>ffuf -u http://10.10.26.7:8081/FUZZ -w /usr/share/wordlists/SecList/Discovery/Web-Content/directory-list-2.3-medium.txt</strong> shows output for auth and ping for the website running on port 8081.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ffuf_8081.png"><img class="aligncenter size-full wp-image-830" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ffuf_8081.png" alt="" width="1533" height="1013" /></a>

How this works:
ffuf – The command used to execute ffuf.
-u – Tells ffuf what url to fuzz.
-w – Tells ffuf what wordlist to use.
FUZZ is important as it tells ffuf where to fuzz in the url.
Let's take a look at some of the folders on 31331.
Turns out /js has a few interesting files located in it.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_js_folder.png"><img class="aligncenter size-full wp-image-832" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_js_folder.png" alt="" width="1823" height="1092" /></a>

The api.js file appears to have the ability to perform remote code execution via the http://10.10.26.7:8081/ping?ip= call.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_js.png"><img class="aligncenter size-full wp-image-831" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_js.png" alt="" width="1117" height="1025" /></a>

Navigating to http://10.10.26.7:8081/ping?ip reveals the message "Invalid ip parameter specified". Let's try and ping the localhost by providing the param ?ip=127.0.0.1 to see what the output would be.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ping.png"><img class="aligncenter size-full wp-image-833" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ping.png" alt="" width="2560" height="371" /></a>

Looks like we were successful in pinging the localhost of the server. We have a very simple form of remote code execution. Let's try and see what else we can do.
Trying to do an ls displays the following message. Looks like we might need to try a few tricks to get this to work.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ls_fail.png"><img class="aligncenter size-full wp-image-834" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ls_fail.png" alt="" width="1271" height="213" /></a>

Turns out we can use back ticks to perform commands. The back tick has a very special meaning. Everything you type between back ticks is evaluated (executed) by the shell before the main command is performed. In this case we can do ?ip=`ls` and see the output displays a sqlite file.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_db_cmd.png"><img class="aligncenter size-full wp-image-835" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_db_cmd.png" alt="" width="1419" height="230" /></a>

This helps answer our next question on TryHackMe.
<div class="room-task-question-details"><strong>Task 3 Question 1: There is a database lying around, what is its filename?</strong> utech.db.sqlite</div>
<div>Let's try and cat the file by doing ip=`cat utech.db.sqlite`</div>
<div></div>
<div class="room-task-question-details"><a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_cat_db_sqlite.png"><img class="aligncenter size-full wp-image-836" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_cat_db_sqlite.png" alt="" width="2348" height="249" /></a></div>
This displays the output from the sqlite file. We have two users from this:
root with a hash of f357a0c52799563c7c7b76c1e7543a32
admin with a hash of 0d0ea5111e3c1def594c1684e3b9be84
Taking these to <a href="https://crackstation.net/">Crackstation</a>, they are easily cracked and we now have passwords.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_cracked_hashes.png"><img class="aligncenter size-full wp-image-837" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_cracked_hashes.png" alt="" width="1081" height="756" /></a>

We can also answer the next question on TryHackMe.
<strong>Task 3 Question 2: What is the first user's password hash? </strong>f357a0c52799563c7c7b76c1e7543a32
<strong>Task 3 Question 3: What is the password associated with this hash? </strong>n100906
Let's try to SSH in as r00t using our new password.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ssh_r00t.png"><img class="aligncenter size-full wp-image-838" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_ssh_r00t.png" alt="" width="979" height="875" /></a>

We have successfully logged in as r00t, but we are not the root yet. Let's do some basic enumerating such as sudo -l, groups, id, whoami, and checking our home directory.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_r00t_enum.png"><img class="aligncenter size-full wp-image-840" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_r00t_enum.png" alt="" width="766" height="511" /></a>

The most interesting thing we see is that we are in the Docker Group.
knowing our user r00t is in the docker groups, let's take a trip to <a href="https://gtfobins.github.io/">GTFObins</a> and see if we can abuse this. Turns out a user in the Docker group can break out into a root system shell.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_gtfobins.png"><img class="aligncenter size-full wp-image-842" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_gtfobins.png" alt="" width="1771" height="1280" /></a>

Abusing this, we can become root by running <strong>docker run -v /:/mnt --rm -it bash chroot /mnt sh</strong>. The example in GTFObins is using alpinie and not bash, remember to use bash or the command will not work.
With that we are now root and can obtain the final bits of information to complete the room and get the SSH Private key.

<a href="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_root.png"><img class="aligncenter size-full wp-image-839" src="https://jarrodrizor.com/wp-content/uploads/2022/09/ultratech_root.png" alt="" width="1469" height="732" /></a>

That completes the room! Well done! If you found this helpful, please send me a <a href="https://twitter.com/Jrod_R87">tweet</a> and tell me what you thought! Feedback is always appreciated!
Jarrod
