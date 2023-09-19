# RootMe
 A Beginner CTF - Can you Root ME?

 > Note: I am using attackbox for this.

 ## Task 1 - Deploy Machine

 ## Task 2 - Reconnaissance
- This beginner-level CTF uses basic Recon techniques. That said, an nmap scan is a great starting point for all levels of complexity. As such, we start with a basic nmap scan of the target machine using the following flags:
  
   - -sS (TCP SYN [stealth] Scan)
  
   - -sV (Service/Version Detection; alternatively can use -A)

   - -sC (Default Script Scan)

  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/97b5805f-9079-4d13-8e71-74c113a76faf)


- We see the results indicate two ports open, and can now answer Q2, 3 and 4
- Now, we will use GoBuster (Q5) to answer Q6
  
  *Note: Use 'gobuster help' in terminal for details on usage*
  
  For our purposes, a GoBuster scan will require the following commands:
    
   - dir [uses directory/file bruteforcing mode]
    
   - -u [to indicate target machine]
    
   - -w [sets path to wordlist, on our attack box, used as a basis for a directory scan]. We will use the wordlist 'common.txt'

   - In addition, we can optionally use -x to specify extensions (php, phtml, php3)

   ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/e645221e-214b-415e-8332-fdec37388882)



    In the scan results, two directories stand out: /panel and /uploads. We can use this info to answer Q6.

## Task 3 - Getting a Shell

Q:  Find a form to upload and get a reverse shell, and find the flag.

- From the GoBuster scan we see that the web server uses php code. Alternatively we can determine this by visiting the target machine IP in firefox, and using the Wappalyzer extension to determine the website's framework or CMS. (Note: This extension *can* be installed in Firefox within the Attackbox, and used during the session, but will be removed when the attacking machine is terminated.)

- Navigating to the target IP in Firefox brings us to this website:

  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/3b91fcdd-75c0-4f60-a691-99df5838fc4d)


  Using Wappalyzer, we see the server and programming language:

  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/3fe1f070-457e-41b0-bab2-b837f2e8ed88)


- With this knowledge, we can play around a bit. Appending the hidden sub-directory found in Q6, we discover a hidden form that allows for file uploads:

  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/5638cc5e-39e4-4ef7-b5dd-6ba510fa03c7)


- Since we know the website is built on php, and found a hidden upload form, we can assume that a php reverse-shell will get us in.
- Some basic methods to locate the shell:

  A web search for "php reverse shell" and/or "file upload vulnerabilities' provides some direction, notably, a github user [PENTESTMONKEY has created this for us](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

  I prefer to open the raw file, copy it to the clipboard and paste it into a new file on the attackbox using nano and save as "php-reverse-shell.php"
  
- Now that we have our shell, before uploading the payload to the web server, we need to make some changes. I use nano php-reverse-shell.php and change the IP to the listening (Attacking) IP and the listening port to 8000 (or keep it as 1234 if you want). Type ctrl+x to save
  
  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/053448d6-7e03-4e45-a7f1-72fe21a87460)



#### Exploiting File Upload Functionality

- Now, let's try to upload our shell...
  
  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/dcd9b6c9-7ff7-40c6-b38c-5e387adf5057)


  It appears php files are not allowed here!
   
  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/371e590a-9a05-4ab1-9835-cf46f299f861)


  It can be helpful to check the source code, but in this case, not much info in the source code of the upload form, so some filtering must be in place that we can't see.

  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/d48cb893-9890-40a9-9802-a696ffafce33)


  A common first strategy for identifying file upload vulnerabilities is to manipulate the file extension to see what may be permitted or filtered out.

  A great resource for alternative file extensions can be found on [hacktricks](https://book.hacktricks.xyz/pentesting-web/file-upload)

  After playing around with some basic manipulations, I renamed the file with the extension _.phtml_:
  
  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/f84fa710-2fc6-46c6-b4f0-8a89d288a14c)

  And attempted to upload the php-reverse-shell.phtml file:
  
  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/b5d04dac-5faf-4f84-90a5-2a44b655b97d)


  AND IT WORKS!
  
  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/1a5842fe-56e5-4cfa-af90-f83775c99f89)


- The next step is to set up a Netcat listener in our terminal, which I do in a new tab, using the same port number (8000), that we changed in the shell code above, and with the following flags:
   - -l is to listen for incoming connections
   - -v is for verbose output
   - -n will skip the DNS lookup
   - -p specifies the port to listen on

  ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/48c6c098-9271-4b11-8cbe-4edd9d8687be)

 Now that we have our listener set up, we can navigate to back to the target site where we know our upload worked. On upload, the file is moved to the /uploads directory (which we also saw in our gobuster scan). Let's go to the /uploads directory on the target website and find our file...

 ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/e87ccc22-394e-42d7-9ebd-463ba1080e45)

 Clicking on the file, you'll notice the site hangs, but activates the shell, let's go back to our listener...

 ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/5355975f-0721-4822-9095-1bf11a9077f8)


 AND WE'RE IN!

 After a little snooping around we eventually find the answer to Q7:

 ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/2877e9a0-42bc-44da-9595-d2810a55a60a)

 ## Task 4 - Privilege Escalation

 _Now that we have a shell, let's escalate our privileges to root_:

 Q8:  Search for files with SUID permission, which file is weird?

 Here we get a hint: `find / -user root -perm /4000`. I had some trouble with the results using those flags, so I did a web search for "find files with SUID permissions", and ultimately found this:

 [Linux Prv Esc with SUID Files](https://medium.com/go-cyber/linux-privilege-escalation-with-suid-files-6119d73bc620)

 ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/525ce836-e7b9-4afb-a53f-8c53bd1c827b)


 Note that the question refers to a weird _file_ so we can add the -type f flag to only search for files, and also use 2>/dev/null to exclude errors; thus the command would look like this: `find / -type f -user root -perm -4000 2>/dev/null`

 and we find the "weird" file to get our answer:

 ![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/44a66cbb-937f-42de-9c33-28bf6cd3c794)

Q8:  Find a form to escalate your privileges. We get a hint to search [gtfobins](https://gtfobins.github.io/), which is a good first stop to look for possible priv esc commands. In the search bar we can search "Python", since that was the name of our file.

![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/0f234bbe-41b2-4ce1-ab3c-68c96ff4f0d4)

![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/7cf882d2-3463-4a07-a962-c73c4e2e96ff)

It's worth reading the description and always good practice. In this case we find two commands. We can skip the first since the binary already has SUID permissions. 

So, here we will copy the second command and paste into our shell ans see if it works. It hung for a bit, but it worked! And we can answer with our flag!

![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/85e644f3-3e86-45ae-8ee2-912a6bf2cbc7)

Congrats! We got root! 

Thank you very much for reading. I hope this was helpful, it's pretty much my first full posted walkthrough. I've got a lot more as I work through this learning process.

  

