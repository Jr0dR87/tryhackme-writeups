This is a WriteUp on how to complete the room <a href="https://tryhackme.com/room/colddboxeasy">ColddBox: Easy</a> on <a href="https://tryhackme.com">TryHackMe</a>.

<img class="alignnone size-full wp-image-301" src="https://jarrodrizor.com/wp-content/uploads/2021/06/banner_canyoucatchthemall.png" alt="" width="1842" height="499" />

Note* I used Kali Linux to complete this room. The IP address of my room was <span id="machine-ip">10.10.170.248</span>, so that will be the IP you see in the writeup. Replace <span id="machine-ip">10.10.170.248</span> with the IP of your target box.

Let's begin this room by enumerating and gathering information about the server by doing an Nmap scan to see what ports are open.

Running the command:
<strong>sudo nmap -sV -T4 -O -oN nmap_scan 10.10.170.248 </strong>shows ports 45122 and 80 are open and running SSH and Apache. Note that SSH is running on a different port than normal.

<img class="alignnone size-full wp-image-324" src="https://jarrodrizor.com/wp-content/uploads/2021/07/nmap_coldbox.png" alt="" width="1279" height="732" />

How this works:
nmap - The command used to execute Nmap.
-sV - This means Nmap will run a service/version detection scan.
-O - Does OS Detection and Fingerprinting.
-T4 - This is the timing option. T4 is a good mix of speed and accuracy for CTF's.
-oN nmap_scan - This means we want to save the output to a file named nmap_scan for future use.

Since Apache is running on port 80. Let us take a look and see if a website is being hosted on the server. Navigating to http://10.10.170.248 displays a landing page for a WordPress site.

<img class="alignnone size-full wp-image-325" src="https://jarrodrizor.com/wp-content/uploads/2021/07/home_page_coldbox.png" alt="" width="1636" height="1126" />

Looking around the website doesn't show anything interesting and nothing appears in the source code either. Let's do more enumeration with Gobuster and see if we can find hidden files/directories.

Running the command:

<strong>gobuster dir -u http://<span id="machine-ip">10.10.170.248</span> -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -o gobuster_scan.txt </strong>displays hidden directories.

<img class="alignnone size-full wp-image-327" src="https://jarrodrizor.com/wp-content/uploads/2021/07/gobuster_coldbox.png" alt="" width="1738" height="608" />

How this works:
gobuster – The command to execute Gobuster.
dir – (scan for directories).
-u – Target URL.
-w – the wordlist we are using to scan for hidden directories. In this case, I used the wordlist in Dirbuster called directory-list-2.3-small.txt.
-o - saves the output to a file. In this case, we are saving to gobuster_scan.txt.

Here are the results of the scan. The interesting directory that stands out is /hidden. Let's take a look.

<img class="alignnone size-full wp-image-328" src="https://jarrodrizor.com/wp-content/uploads/2021/07/gobuster_hidden_coldbox.png" alt="" width="1193" height="209" />

Navigating to http://10.10.170.248/hidden displays a message note intended for general public, but found with simple scanning techniques.

<img class="alignnone size-full wp-image-329" src="https://jarrodrizor.com/wp-content/uploads/2021/07/hidden_page_coldbox.png" alt="" width="1608" height="213" />

Since we know this is a WordPress site. Let us bring in another tool to learn more about it. WPScan.

Running the command:

<strong>wpscan --url http://10.10.170.248 --enumerate </strong>will scan the site and give us more information about the WordPress CMS. This displays a lot of information and I encourage you to run it and see all the results from the scan.

How this works:
wpscan – The command to execute WPScan.
--url – Target URL.
--enumerate - Tells WPScan to scan the site to learn about plugins, themes, configs, users and other info.

<img class="alignnone size-full wp-image-332" src="https://jarrodrizor.com/wp-content/uploads/2021/07/wpscan_pic1_coldbox.png" alt="" width="1090" height="1137" />

The results we should be interested in are the three users. They are philip, c0ldd, and hugo.

<img class="alignnone size-full wp-image-334" src="https://jarrodrizor.com/wp-content/uploads/2021/07/wpscan_pic3_coldbox.png" alt="" width="1186" height="415" />

We can use WPScan and see if we can find passwords for these users and log in.

Running the command:

<strong>wpscan --url http://10.10.170.248 --usernames philip,hugo,c0ldd --passwords /usr/share/wordlists/rockyou.txt </strong>begins the brute force process to see if we can find a password for these users.

After a few minutes, we have a hit on the user C0ldd!

<img class="alignnone size-full wp-image-342" src="https://jarrodrizor.com/wp-content/uploads/2021/07/wpscan_coldd_password_coldbox.png" alt="" width="724" height="66" />

We can now try and log in and the user!

Let's navigate to http://10.10.170.248/wp-login should display the log in page. Let's use the credentials we have and log in.

<img class="alignnone size-full wp-image-339" src="https://jarrodrizor.com/wp-content/uploads/2021/07/wp_login_coldbox-1.png" alt="" width="801" height="616" />

We now have access to the dashboard. Here we can look around and get more information about our target. What's very interesting and poor configuration on the users end is the WordPress Editor is still enabled. This is located in the menu on the left hand side under Appearance.

<img class="alignnone size-full wp-image-340" src="https://jarrodrizor.com/wp-content/uploads/2021/07/wp_dashboard_coldbox.png" alt="" width="2560" height="1097" />

With this feature, we can easily drop in PHP code to perform a reverse shell.

Kali Linux has PHP Reverse Shell scripts located in /usr/share/webshells/php/. The file is named php-reverse-shell.php. Open it with a text editor and we need to update two values. The $ip and $port.

The $ip will be the IP Address. Using the TryHackMe VPN, let's look for tun0 and use that IP! We can find the IP Address with the command <strong>ip addr </strong>(which displays network interfaces).

The $port will be our listening port for the reverse shell. Let's use 4444 as it's out of the range of the known ports.

<img class="alignnone size-full wp-image-343" src="https://jarrodrizor.com/wp-content/uploads/2021/07/reverse_shell_script_coldbox.png" alt="" width="1329" height="1105" />

Before we copy and paste this script to the WordPress Editor, let's get a listener setup with Netcat.

Now let’s use Netcat and get into the system!

<strong>Running the Command: nc -lvnp 4444 </strong>starts a Netcat listener.

What this does
nc – Executes Netcat.
-l – Listen for a request.
-v – Verbose output.
-n – Do not use DNS.
-p – What port to listen on.

<img class="alignnone size-full wp-image-344" src="https://jarrodrizor.com/wp-content/uploads/2021/07/listener_coldbox.png" alt="" width="413" height="86" />

Now that we have the script set up and the listener going. We need to copy and paste it to the editor to get the reverse shell to fire. We also need to pick a page to use that we can navigate to in the web browser. Let's do the 404.php page as it's easy to find and won't break anything in the template to break the site. Let's open it up in the editor and copy and paste!

<img class="alignnone size-full wp-image-341" src="https://jarrodrizor.com/wp-content/uploads/2021/07/editor_404_coldbox.png" alt="" width="2559" height="876" />

Now we just need to navigate to a page that doesn't exist to trigger the 404.php page. Let's go to http://10.10.170.248/?p=3 and trigger it! Let's check back to our Netcat listener. We should have a session going and running as www-date.

<img class="alignnone size-full wp-image-346" src="https://jarrodrizor.com/wp-content/uploads/2021/07/remote_login_coldbox-1.png" alt="" width="1275" height="298" />

We can navigate around the server, but we won't get far as the www-data user. This would be a good time to get Linpeas on the server and let that scan for areas of interest.

The way we can do this is to download the linpeas.sh file from the <a href="https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/linPEAS">GitHub Repository</a>.

<img class="alignnone size-full wp-image-347" src="https://jarrodrizor.com/wp-content/uploads/2021/07/linpeas_github.png" alt="" width="2544" height="1060" />

With linpeas.sh on our machine, we can start a

That completes the room! Well done! If you found this helpful, please send me a <a href="https://twitter.com/JarrodR87">tweet</a> and tell me what you thought! Feedback is always appreciated!

Jarrod
