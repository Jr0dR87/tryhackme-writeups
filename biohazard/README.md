Pages found
/
/mansionmain
/diningRoom/
/teaRoom/
/artRoom/
/barRoom/
/diningRoom2F/
/tigerStatusRoom/
/galleryRoom/
/studyRoom/
/armorRoom/
/attic/
/hidden_closet/

Flags
Emblam emblem{fec832623ea498e20bf4fe1821d58727} http://10.10.174.74/diningRoom/emblem.php
Lock Pick lock_pick{037b35e2ff90916a9abf99129c8e1837} http://10.10.174.74/teaRoom/master_of_unlock.html from Barry in TeaRoom
Music Sheet music_sheet{362d72deaf65f5bdc63daece6a1f676e} http://10.10.174.74/barRoom from decdoding base32 NV2XG2LDL5ZWQZLFOR5TGNRSMQ3TEZDFMFTDMNLGGVRGIYZWGNSGCZLDMU3GCMLGGY3TMZL5
Gold Emblem gold_emblem{58a8c41a9d08b8a4e38d02a4d7ff4843} from /barRoom after using Music Flag
Blue Gem blue_jewel{e1d457e96cac640f863ec7bc475d48aa} Push the statue to the first floor by using Rot13 ciper decoder from DiningHall2f page source 
Shield Key Flag is shield_key{48a7a9227cd7eb89f0a062590798cbac}. Got this after decrypting Vigenere cipher and navigating to diningRoom/the_great_shield_key.html
Helmet Key Flag. After using steganography on the images and combining the findings, we get the password for the gpg file and get the flag 
helmet_key{458493193501d2b94bbab2e727f8db4b} helmet_key{458493193501d2b94bbab2e727f8db4b}

Unique Findings
on /diningRoom page. Found in page source SG93IGFib3V0IHRoZSAvdGVhUm9vbS8= which is base64 for How about the /teaRoom/
Lock Pick can be used on page /barRoom/
Music Sheet is used on Bar Room and then you get Gold Emblem flag
The Blue Gem is on Dining Room 2F and in the page source is a Rot13 Ciper that translates to You get the blue gem by pushing the status to the lower floor. The gem is on the diningRoom first floor. Visit sapphire.html
diningRoom/emblem_slot.php cipher is a Vigenere Cipher. Decrypted shows there is a shield key inside the dining room. The html page is called the_great_shield_key
In the closet room that has Enrico, clicking on M0 Disk 1 shows another Vigenere cipher wpbwbxr wpkzg pltwnhro, txrks_xfqsxrd_bvv_fy_rvmexa_ajk
translates to weasker login password, stars_members_are_my_guinea_pig
In the studyRoom/ use the helmet key to download doom.tar.gz to get the SSH user

Crest1 RlRQIHVzZXI6IG
Crest2 h1bnRlciwgRlRQIHBh
Crest3 c3M6IHlvdV9jYW50X2h
Crest4 pZGVfZm9yZXZlcg==

Stego images
image 1. I used steghide. steghide extract -sf 001-key.jpg
image 2. I used exiftool. exiftool 002-key.jpg
image 3. I used binwalk -e 003-key.jpg

From Base64 cGxhbnQ0Ml9jYW5fYmVfZGVzdHJveV93aXRoX3Zqb2x0 plant42_can_be_destroy_with_vjolt

Together, all the crest make RlRQIHVzZXI6IGh1bnRlciwgRlRQIHBhc3M6IHlvdV9jYW50X2hpZGVfZm9yZXZlcg==
which is base64 for 
FTP user: hunter, FTP pass: you_cant_hide_forever

ssh in as umbrella_guest using T_virus_rules. ls -la shows .jailcell. this is where chris is. 
umbrella_guest can become Weasker with su - weasker and Weasker has sudo ALL

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
