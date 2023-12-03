# Task 1 - Lazy Admin

## Enumerate!

As always, we start with an nmap scan:

![Pasted image 20231112105447](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/446ac53b-ff7e-4487-8efc-7eb8532ca502)

We know there's a webserver, let's check that first. Unfortunately we get the default Apache2 page.

![Pasted image 20231112105724](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d00f3310-e489-42b3-a22e-40fa08b153f2)


Let's try a gobuster scan:

!![Pasted image 20231112110747](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/780673e4-e69f-4fbd-bda8-6ec4d6d19813)

And we see a hidden directory: /content. Let's see if this offers anything.

Ok, so a few things to check out here:

![Pasted image 20231112110940](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/e00f9e87-9d7f-4c51-b7c2-3977246a8484)

And at the bottom left, we have a link:

![Pasted image 20231112115107](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5589aaf7-93dd-4c26-b028-904b58a08a5b)

Nothing indicating Basic CMS Sweetrice version number so far, and the links don't give us anything.

I looked at the source code, not much there either.


Let's try to see if there are any subdirectories we can access. Maybe the comment to the webmaster is a clue?

![Pasted image 20231112115714](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/4a543a3d-4524-49d1-8933-afede04f1ce5)

Ok, so appending the /content directory to our URL in gobuster revealed some additonal subdirectories as seen above.


I start from the top, to have a look around. Not much in /images. Moving into the /inc subdirectory might give us more to go on. 

So I take some time and look around, most of the files are blank, but we have a mysql_backup, which can often contain passwords. 

![Pasted image 20231112122344](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/0e56c443-63bc-4ea3-ba43-d434ef347dd3)


Looking at the backup file, we can see some possibly useful info:

![Pasted image 20231112125937](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c4dcd400-9367-434f-9add-af2c2ea07e27)

Interesting, as shown above, we see the only real indicators are the lines with the words "admin", "manager" and "passwd", and a alphanumeric string "42f7...fcb" which this could be a hashed password. I already had [cyberchef] open, so I use the "Analyze Hash" recipe. It doesn't provide exact info, but gives the possible hashing functions:

![Pasted image 20231112123636](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/f7210bff-afe0-423d-8329-e950a59d6215)

![Pasted image 20231112123556](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/25afec62-67ed-4525-9546-03c0d6dccf51)

So I confirm this and decrypt using [hashes.com](https://hashes.com/en/tools/hash_identifier).

![Pasted image 20231112124334](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/bd752e17-c095-4ae0-92e5-f7d9c11847fe)

So, we have a password and likely username "manager"! 

I go back to the directory list, and just out of curiosity look next at the /as directory. Looks like the admin panel! Let's enter "manager:password123" for username and password.

![Pasted image 20231112130402](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/3399caef-5f95-435f-a8f8-c8c4ca4ba301)


_Note: I had to complete the exploitation phase after an errand, so the ip's of the target and/or AttackBox may be different from here forward._

![Pasted image 20231113141025](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/6906a8a5-11a8-4103-b606-3aa6973f4d3a)

And we are in! We can see the version of SweetRice cms (1.5.1), now that we have access.  

What we know so far:
- We have user:password with admin access to the site
- We have access to several sub-directories
- The site is built on PHP
- We have a way to upload files
- We have several exploits for v 1.5.1, as shown below:

![Pasted image 20231113141552](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/2c905d8d-9689-4fc2-b0dd-61a633535329)


Let's start by looking at the above exploits found using searchsploit, which on AttackBox are located in /opt/searchsploit/exploits/php/webapps. Navigating to that directory we find the text files. 

After reading through the first 40718.txt file for version 1.5.1, I realized that we've already obtained that info from our enumeration of the directories, so that's not it.

![Pasted image 20231113144040](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/b6fde1ae-9be2-4023-9e76-35f773cf5659)

![Pasted image 20231113145228](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/75e27ba9-d6ac-483c-8c9e-2d4e59d25315)


I search for sweetrice 1.5.1 on [exploit-db](https://www.exploit-db.com/exploits/40700) and found pretty much the same. So having a look at 40700, we learn that we can insert and execute PHP code in the Ads section of the SweetRice CMS.


![Pasted image 20231113150140](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c4c73ff0-575c-41be-a3c1-35e986d4a3d8)


So, I will try the well known reverse shell created by [pentest monkey], editing the Attacker IP field to my AttackBox IP:

![Pasted image 20231113151436](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/a126be24-fff6-4667-a2b0-d3022a3e0e2f)


And I go back to the CMS to upload the file. According to the exploit, we apparently upload in the ads directory section, which is actually found in /content/as/.

![Pasted image 20231113153708](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8a9be223-5061-45e0-a465-9a5730d47793)


We actually just paste the script directly into the code box, and name it "reverse-shell"...hopefully it works.

Set up our netcat listener, I kept the port 1234 as in the default script:

![Pasted image 20231113153325](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8bde6c3a-9bc7-4d9f-8bfa-6bf00e33ee07)


I browsed the site and had a hard time finding the file, looking at the exploit again, I saw the last line telling us where the file would be located after upload: 

![Pasted image 20231113154042](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/30275153-c99d-479e-9889-feae6d1d8a3d)

`http://localhost/sweetrice/inc/ads/hacked/php`

And here it is! So with our netcat listener waiting patiently, let's try and execute the shell, hopefully it works.

![Pasted image 20231113154444](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/e4294f50-b8f7-49f5-9e2a-272da5d7f541)

And we're in! 

![Pasted image 20231113155139](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c6ed0edf-30be-4fc4-9278-9d7a89c94ac7)


Apparently we have an unprivileged shell, but we can at least answer Q1 with the flag:

![Pasted image 20231113155509](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/1f6a7479-c352-4ff2-b066-87ee826b5737)

* * * 

## Privilege Escalation

Well, we found another piece of info. Just look at the files above user.txt:

![Pasted image 20231113155843](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/b80d2b38-8058-4dc2-adff-eb8bdb462736)


Let's begin by finding what we can do as www-data using:

`sudo -l`

![Pasted image 20231113160417](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/46cb4762-a52a-40e0-a7fb-1bad9869c13a)


And check permissions of that directory using:

`ls -la /etc/copy.sh`

![Pasted image 20231113160710](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8e83d054-13fe-42c2-b2e5-7581259a7824)


Let's see what we have in copy.sh

![Pasted image 20231113161235](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/cc17834d-8f7f-49ce-ad68-6379fb496586)


So from here, we must pipe, but change the above IP to our Attacker IP:

![Pasted image 20231113163557](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/fcd77fff-edc2-4a32-a60f-e1920750d3d6)


And in another tab on the AttackBox, start another netcat listener on port 5554:

`nc -lvnp 5554`

And back on our remote, or 'itguy' machine, run:

![Pasted image 20231113163834](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/9c5e8f1e-858c-4b87-a330-7ca0f092edde)

![Pasted image 20231113163405](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/6ff4f6d2-1fa3-4f51-a479-aa648f991455)

And we got root! And our flag:

![Pasted image 20231113164723](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/38c49e4c-89eb-4314-82bf-9ae3f0908a97)

Thanks for reading!

