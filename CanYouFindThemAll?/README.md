This is a WriteUp on how to complete the room [Gotta Catch'em All!](https://tryhackme.com/room/pokemon) on [TryHackMe](https://tryhackme.com).

![](https://jarrodrizor.com/wp-content/uploads/2021/06/banner_canyoucatchthemall.png)

Note* I used Kali Linux to complete this room. The IP address of my room was 10.10.154.53, so that will be the IP you see in the writeup. Replace 10.10.154.53 with the IP of your target box.

**Task 1: Can You Catch'em All?\
**

Let's begin trying to Catch'em All by doing a Nmap scan to see what ports are open.

Running the command:\
**sudo nmap -sV -O -T4 -oN scans/nmap_scan 10.10.154.53** shows ports 22 and 80 are open and running SSH and Apache.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/nmap_scan_canyoucatchthemall.png)

How this works:\
nmap -- The command used to execute Nmap.\
-sV -- This means Nmap will run a service/version detection scan.\
-O -- Does OS Detection and Fingerprinting.\
-T4 -- This is the timing option. T4 is a good mix of speed and accuracy for CTF's.\
-oN nmap_scan -- This means we want to save the output to a file named nmap_scan.

Since we see Apache is running on port 80, let's run Gobuster to see if we can find any hidden files and directories.

Running the command:

**gobuster dir -u http://10.10.154.53 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o scans/apache_root_gobuster_scan_canyoucatchthemall.txt -x php,html,txt,cgi,bak**

How this works:\
gobuster -- The command to execute GoBuster.\
dir -- (scan for directories).\
-u -- Target URL.\
-w -- the wordlist we are using to scan for hidden directories. In this case, I used the wordlist in Dirbuster called directory-list-2.3-small.txt.\
-o -- saves the output to a file. In this case, we are saving to apache_root_gobuster_scan_canyoucatchthemall.txt.\
-x -- Extensions for files to locate. In this case, find php, html, txt, cgi, and bak.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/gobuster_index_canyoucatchthemall.png)

The results display just the index page for Apache. Let's check it out because we don't really have anywhere else to look.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/apache_page_canyoucatchthemall.png)

Navigating to http://10.10.154.53 shows the default Apache landing page.

The source code displays interesting information though. It's always worth your time to view source code as it can not only display comments on how the application works. Some developers could leave behind credentials and other useful information to use.

This bit shows a snippet of JavaScript to create an array of various Pokemon. We can see it on display in the browser console.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/apache_page_js_code_canyoucatchthemall.png)

![](https://jarrodrizor.com/wp-content/uploads/2021/06/apache_page_array_canyoucatchthemall.png)

What's interesting is what is at the bottom of the source code. We might have found some credentials.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/apache_source_code_hint_canyoucatchthemall.png)

Let's use these and try to SSH into the server.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/ssh_login_canyoucatchthemall.png)

Looks like we are in! Let's run ls -la to see what's here for us.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/home_directory_canyoucatchthemall.png)

It seems the user pokemon cannot sudo.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/cantsudo_canyoucatchthemall.png)

We should also take a look at /etc/passwd to see if we can find other users for this machine. The one that sticks out is **ash**. We will probably use that user later, so let's keep that in our notes.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/passwd_canyoucatchthemall.png)

**Flag: Find the Grass-Type Pokemon**

Let's start by checking out what is on the Desktop. CTF's normally have something useful there. Here we find a strange zip file named P0kEm0n.zip.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/desktop_canyoucatchthemall.png)

Let's unzip it with the **unzip** command.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/unzip_pokemon_zip_canyoucatchthemall.png)After unzipping, we find the file grass-type.txt with Hex in it.

**50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d**

Let's take this to [CyberChef](https://gchq.github.io/CyberChef/) and see what we can learn.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/hex_flag_canyoucatchthemall.png)

Since this is Hex, let's paste the contents of the grass-type.txt into the Input Section. Now search for From Hex on the left-hand side and use it for the Recipe. Turns out this helps us get the first flag! One Pokemon down!

**Flag: Find the Water-Type Pokemon**

Let's check out what else is in the Apache folders. Navigating to /var/www/html, we find a file named water-type.txt.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/apache_folder_contents_canyoucatchthemall.png)

After Opening the text file, we find this.

**Ecgudfxq_EcGmP{Ecgudfxq}**

Going back to [CyberChef](https://gchq.github.io/CyberChef/) we can assume this is ROT13 based on its appearance.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/rot13_flag_canyoucatchthemall.png)

Since this is ROT13, let's paste the contents of the water-type.txt into the Input Section and search for ROT13 on the left-hand side and use it for the Recipe. If we tweak the amount to 14 we should get the flag in the output for the water-type Pokemon! Another Pokemon down!

**Flag: Find the Fire-Type Pokemon**

The naming convention for the Pokemon element-type files seems to be element-type.txt. Let's see if we can try and find the fire-type Pokemon with that naming convention. We can try that with the **find** command.

**Command: find / -type f -name "fire-type*" 2>/dev/null**

This is going to search the server starting in the root directory and look for files (-type f) with the name fire-type* (-name "fire-type*") and throw all the errors we get into /dev/null so they don't flood the command line (2>/dev/null).

![](https://jarrodrizor.com/wp-content/uploads/2021/06/find_firetype_canyoucatchthemall.png)

We got a hit and the file is located in /etc/why_am_i_here?/ Inside the folder is a file named fire-type.txt.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/etc_whyamiherefile_canyoucatchthemall.png)\
After Opening the text file, we find this.

**UDBrM20wbntDaGFybWFuZGVyfQ==**

Based on the last two characters ==, this is a good indicator that it is probably base64.

Going back to [CyberChef](https://gchq.github.io/CyberChef/) one more time!

![](https://jarrodrizor.com/wp-content/uploads/2021/06/base64_flag_canyoucatchthemall.png)

Since this is base64, let's paste the contents of the fire-type.txt into the Input Section and search for From Base64 on the left-hand side and use it for the Recipe. This shows the third Pokemon!

**Final Flag: Who is Root's Favorite Pokemon**

Back in the home directory of pokemon, let's check the contents of all the folders to see if we can find anything else that's interesting. Using the command ls -R

![](https://jarrodrizor.com/wp-content/uploads/2021/06/deep_directory_dive_canyoucatchthemall.png)

The Videos directory seems to hold multiple recursive folders. Navigating down brings us to a file named Could_this_be_what_Im_looking_for?.cplusplus. Reading the contents of the file shows this snippet of code.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/c_canyoucatchthemall.png)

The important thing from this is we seem to have found the password for the user ash. Let's try and **su -- ash** and see if this is the password.

![](https://jarrodrizor.com/wp-content/uploads/2021/06/su_ash_canyoucatchthemall.png)

This seems to work! We are now the user ash.

The first place we should check for the flag is ash's home directory. Inside /home we can see ash's folder can only be accessed by root. We do see the file roots-pokemon.txt with permissions that show ash can read, write and execute the file. We can open the file and get the final flag!

![](https://jarrodrizor.com/wp-content/uploads/2021/06/lastflag_canyoucatchthemall.png)

I would like to add though, ash has the ability to use sudo for all commands and you can become root by running sudo su -- .

![](https://jarrodrizor.com/wp-content/uploads/2021/06/becoming_root_canyoucatchthemall.png)

That completes the room! Well done! If you found this helpful, please send me a [tweet](https://twitter.com/JarrodR87) and tell me what you thought! Feedback is always appreciated!

Jarrod
