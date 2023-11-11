# Probe

### Challenge - Sometimes, the only info we have on a target is an IP address. Can we complete the challenge and conduct an in-depth probe on the target?

LET'S GO!

* * * 

## Task 1 - LETS PROBE

Q1: What is the version of the Apache server?

As is typically the case, we start with a basic nmap scan:

`nmap -sS -sC -A -v [target_ip]`

![Pasted image 20231111122844](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/863e1bb5-36fc-423a-ad60-4c06034fe448)

And we can answer Q1.

* * * 

Q2: What is the port number of the FTP service?

I ran a second scan, using:

`nmap -sS -sC -A -v -p 1000-9999 [target_ip]`

![Pasted image 20231111123248](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/4364a789-d7d9-4042-97f4-20452bace3f7)

Now we have a bit more info, and can answer Q2.

* * * 

Q3: What is the FQDN for the website hosted using a self-signed certificate and contains critical server information as the homepage?

We can refer to the second nmap scan to find this.

* * * 

Q4: What is the email address associated with the SSL certificate used to sign the website mentioned in Q3?

I navigate to the standard  http port 80, and get a 403 forbidden error, likewise with port 443. Next I try 1443, 

I finally get to the target using:

`https://[machine_ip]:1443`

Here we get a warning about an invalid security certificate:

![Pasted image 20231111131712](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/940f998d-162e-4e7b-beea-7413b6d7a38e)

Clicking on "Learn more...", we get the following:

![Pasted image 20231111131823](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/0f6dc8e1-3d54-415f-82e0-73291905267d)

And finally, choose "View Certificate" and in the next tab we see the Issuer and can answer Q4.

* * *

Q5: What is the value of the PHP Extension Build on the server?

Going back to the target website at port 1443 (the warning), click on "Accept the Risk and Continue" 

And it brings us to the public php info page created for the server. 

![Pasted image 20231111134001](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/6b4cb12a-472f-4ec0-aae7-a0179d632408)

But, according to this tutorial found on [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-multiple-php-versions-on-one-server-using-apache-and-php-fpm-on-centos-8) this page should be removed. This might come in handy? We'll see!

![Pasted image 20231111134314](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5ea025f6-dace-4b8e-8814-725a7c4f153c)

Nevertheless, we can answer Q5.

* * *

Q6: What is the banner for the FTP service?

 Referring back to the nmap scan, we see the FTP service running on port 1338.

 I had to find the info on how to get nmap to show the FTP banner [here](https://nmap.org/nsedoc/scripts/banner.html)

 The command is:
 
`nmap -sV --script=banner -p 1338 [target_ip]`

![Pasted image 20231111135741](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/aa3dde20-fce9-4cfa-beb5-1fb556f0c138)

Interesting! And we have the answer to Q6.

* * * 

Q7: What software is used for managing the database on the server?

I did a web search for this..."software used to manage wordpress server" and found the answer right away. 

* * * 

Q8: What is the Content Management System hosted on the server?
Q9: What is the version number of the CMS hosted on the server?

For Q8 and Q9, refer back to the nmap results.

* * * 

Q10: What is the username for the admin panel of the CMS?

So, earlier in the task I navigated to the website at port 9007. 

![Pasted image 20231111125140](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/63b94d0c-2b25-4d38-bef3-a7d5676d2818)

Not much here.  After some digging, I let it go and came back to it after finishing the rest. 

This one I had to research. I knew it involved changing a parameter in the URL, but I was on the wrong track using page numbers and trying to access directories of the other services.

Ultimately, I searched  "username for wp admin panel" and one result stood out right away. Needless to say, I found the answer using this method.

![Pasted image 20231111162725](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d6cf1f13-f5e7-4741-8949-4dd6eac5f4ab)

* * *

Q11: Again, I just did a web search for "OSVDB-3092 wordpress" and found the answer.

* * *

Q12: Refer back to the initial nmap scan and it's pretty obvious.

* * * 

Q13: What is the flag value associated with the web page hosted on port 8000.

For this I used gobuster to find any hidden directories:

    `gobuster dir -u http://[machine_ip]:8000 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

![Pasted image 20231111152943](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/cd66a266-ec27-46c5-a821-b4c7c2c5950f)

I tried the first one and got the flag!

![Pasted image 20231111153619](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/095d040d-bbc7-4781-ab79-bc17918c2b18)

That's all! Nice room that was really about scanning, digging, and as a last resort just look up a term or thought. That's how we learn and reinforce things we may have already learned. 

Thanks for reading!

