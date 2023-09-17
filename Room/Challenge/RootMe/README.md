# RootMe
 A Beginner CTF - Can you Root ME?

 ## Task 1 - Deploy Machine

 ## Task 2 - Reconnaissance
- This beginner-level CTF uses basic Recon techniques. That said, an nmap scan is a great starting point for all levels of complexity. As such, we start with a basic nmap scan of the target machine using the following flags:
  
  -sS (TCP SYN [stealth] Scan)
  
  -sV (Service/Version Detection; alternatively can use -A)
  
    ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/52545174-39c9-47f3-bbc4-948838d0ca6f)


- We see the results indicate two ports open, and can now answer Q2, 3 and 4
- Now, we will use GoBuster (Q5) to answer Q6
  
  *Note: Use 'gobuster help' in terminal for details on usage*
  
  For our purposes, a GoBuster scan will require the following commands:
    
   - dir [uses directory/file bruteforcing mode]
    
   - -u [to indicate target machine]
    
   - -w [sets path to wordlist, on our attack box, used as a basis for a directory scan]. We will use the wordlist 'common.txt'

   - In addition, we can optionally use -x to specify extensions (php, phtml, php3)

  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/46058e84-685f-4ce4-b190-114197f4154a)


    In the scan results, two directories stand out: /panel and /uploads. We can use this info to answer Q6.

## Task 3 - Getting a Shell

Q:  Find a form to upload and get a reverse shell, and find the flag.

- From the GoBuster scan we see that the web server uses php code. Alternatively we can determine this by visiting the target machine IP in firefox, and using the Wappalyzer extension to determine the website's framework or CMS. (Note: This extension *can* be installed in Firefox within the Attackbox, and used during the session, but will be removed when the attacking machine is terminated.)

- Navigating to the target IP in Firefox brings us to this website:

  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/38af8837-b7cd-48dd-972e-561096df44cb)

  Using Wappalyzer, we see the server and programming language:
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/ebc55ad7-4d58-413e-b584-bb61012b2084)

- With this knowledge, we can play around a bit. Appending the hidden sub-directory found in Q6, we discover a hidden form that allows for file uploads:

  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/71ebbc7e-0976-4747-b136-7d0473dcef53)

- Since we know the website is built on php, and found a hidden upload form, we can assume that a php reverse-shell will get us in.
- Some basic methods to locate the shell:

  A web search for "php reverse shell" and/or "file upload vulnerabilities' provides some direction, notably, a github user [PENTESTMONKEY has created this for us](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php).

  I prefer to open the raw file, copy it to the clipboard and paste it into a new file on the attackbox using nano and save as "php-reverse-shell.php"
  
- Now that we have our shell, before uploading the payload to the web server, we need to make some changes. I use nano php-reverse-shell.php and change the IP to the listening (Attacking) IP and the listening port to 8000 (or keep it as 1234 if you want). Type ctrl+x to save
  
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/d46aeb63-4e7f-460e-b798-724679b5f58e)


#### Exploiting File Upload Functionality

- Now, let's try to upload our shell...
  
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/ee4810fc-1434-41b7-9fb1-c1097f85915b)

  It appears php files are not allowed here!
   
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/060d0989-7fe8-4dd6-918f-40a78eb8c740)

  Not much info in the source code of the upload form, so some filtering must be in place that we can't see.

  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/965d1e4a-9ed4-4700-bf19-e87c789c0daa)

  A common first strategy for identifying file upload vulnerabilities is to manipulate the file extension to see what may be permitted or filtered out.

  A great resource for alternative file extensions can be found on [hacktricks](https://book.hacktricks.xyz/pentesting-web/file-upload)

  After playing around with some basic manipulations, I renamed the extension to .phtml:
  
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/ecc237d4-8857-4e28-bacf-0416513194f2)


  And attempted to upload the php-reverse-shell.phtml file:
  
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/3bb6953d-d392-4862-832a-a80ae2e37c2c)

  AND IT WORKED!
  
  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/d7db7c1d-7af8-4cb9-9f50-56fc2ce5b440)

  The next step is to set up a Netcat listener, which I do in a new tab in the terminal, using the same port number (8000), which was changed in the shell code above, and the following flags:
   - -l is to listen for incoming connections
   - -v is for verbose output
   - -n will skip the DNS lookup
   - -p specifies the port to listen on

  ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/83529f89-aff8-4f25-9361-49a47d0fcc45)


 Now that we have our listener set up, we can navigate to back to the target site where we know our upload worked. The file is moved to the /uploads folder once uploaded (demonstrated in GoBuster shown above). Let's go to the /uploads directory on the target and find our file...

 ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/5d3504d1-1e0f-42a8-bd5f-a30bb8e171d3)

 Clicking on the file activates the shell, let's go back to our listener...

 ![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/08c80ad8-058f-4beb-b213-8165562a69a5)

 AND WE'RE IN!



  

  

  



TBC

  

