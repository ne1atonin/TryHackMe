# Anthem

* * *

_Let's exploit a Windows Machine in this Beginner-Level Challenge_

* * * 

## Task 1 - Website Analysis

_This task serves to grab our attention so we can find the 'keys to the castle'. Designed for beginners, no brute-force logins required, just our preferred browswer and RDP._

### Q1 - Enumerate - _Run nmap to see what ports are open_

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/f2a68715-7080-4351-9418-82b1b675b13a)
![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/b7344aca-88d0-4574-b0bb-34f16784d1d3)

I choose to use nmap [machine IP] -sS -sC -A -v
- -sS  Default, quick TCP SYN scan. I think this is more out of habit than anything else, as it's one of the first flags I learned :D
- -sC  Performs a script scan with the default set of scripts.
- -sV  Version Detection
- -A   Aggressive Scan - enables additional advanced and aggressive options, including: OS detection (-O), version scanning (-sV), script scanning as stated above (-sC) and traceroute (--traceroute). Basically a comprehensive flag with a set of options. Not sure if it's redundant to use both -sC and -A, or if it would be more observable to use both, or perhaps the use of -A cancels -sC.
- -v   Increase verbosity of output

  And with this we can answer Q2 and Q3.
  
### Q2 - _What Port is for the Web Server?_

### Q3 - _What Port is for Remote Desktop Service?_

### Q4 - _What is a possible password in one of the pages web crawlers check for?_

We know that Q4 contains a Hint: "fill in the gap ******.txt". We also know that a particular file lives at the root of our site, a very common file used in SEO to instruct search engine bots how to ***crawl*** pages on their website, or essentially where the bots are allowed or not allowed to crawl and index.

Now, we also know that just the presence of this file does not indicate a security vulnerability. That said, it's often used to identify restricted or private areas of a site's contents. Thus, the information in the file may halp an attacker map out the site's contents, especially if the file mentions or identifies locations that are not specifically apparent or linked to from anywhere within the site. The vulnerability presents itself in situations where the application relies on this file to protect/prevent access to certain areas, but access control to the .txt file itself is not protected.

See [CWE-200](https://cwe.mitre.org/data/definitions/200.html)

OK, so, appending this common file to the end of the web link gives us our answer to Q4, as well as a nice list of additional directories that may be useful later!

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/6bb931de-d664-48a4-a98d-ab7802cec09a)

### Q5 - _What CMS is the Website Using?_

I'm still green, so did a quick google search to find the answer, based on the info above:

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/35ada49a-3b5d-476f-a92f-2aeeec0eca63)

### Q6 - _What is the Domain of the Website?_

Fairly straightforward, this one.

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/3a6800eb-92c2-4ca4-82fd-d0870ae7a0a5)


### Q7 - _What is the Name of the Administrator?_

Again, just reading a bit of the site provides us with the answer after a google search (hint) based on clues in the site:

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/26b22ea4-6868-4f17-8433-3bf9c3934b9f)

### Q8 - _Can we find the email address of the administrator?_

Again, the hint provided here prompts us to find another email address on the site and assume the administrator follows this same email format. Knowing the administrator's name, and applying this format gives us our answer.

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/1e8f540c-b79b-4a0f-bc90-517849b671ad)

* * *

## Task 2 - Spot the Flags

_Our beloved admin left some flags behind that we need to gather before proceeding to the next task..._

> I saw one of these initially, by accident, when viewing the source code for the main site.

### Q1 - _What is Flag 1?_

Had to click through some links in the source but ultimately found this one in the "We are Hiring" post

### Q2 - _What is Flag 2?_

This is the one I saw initially, as noted above. Stands out.

### Q3 - _What is Flag 3?_

Found with respect to "Authors"

### Q3 - _What is Flag 4?_

Found using Inspector

* * *

## Task 3 - Final Stage

_Let's get into the box using the intel we gathered!_

### Q1 - _Figure out Username and password to log in to the box. (Box is not on a domain)_

Based on our early findings, we know we can connect to http://[machine IP]/umbraco/ using the info we discovered: *username: (not email) password: from Task 1 above, though not much info using this method.  Since we know RDP was introduced at the beginning, seems logical that we should try to use this method using similar username and same password.

I used Remmina for RDP

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/134289af-0aac-4bcc-8f51-384a959c3dc7)


![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/126c46e7-4cf8-4486-8753-4ac2e165e59c)

### Q2 - _Gain Initial Access to the Machine, What is the Contents of user.txt?_

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/b1c2e5c3-8967-4d2d-b312-68f00a1db3a2)

### Q3 - _Can We Spot the Admin Password?_

We get a hint here that it is hidden. 

So, we can go to our C drive, or this PC, and try to set "show hidden files and folders" and we get a few new folders, including one called "Backup", with a txt file inside named "restore.txt"

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/de4486f0-fa1d-4535-bcad-79f56fc46683)

But, unfortunately we don't have access rights to this file.

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/b59ec47a-71c1-4125-9676-6ea93860c875)

So, we need to change permissions in the folder by right clicking on the file, selct properties > Security > Edit permissions and select all for "Allow"

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/104efdd9-630b-4e70-a41f-1f450ea88e90)

Now we can open the file and we have our password for Administrator! And can open the file restore.txt, and answer Q3.

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/be330931-e1b4-4130-ac66-091353d360b8)

### Q4 - _Escalate your privileges to root, what is the contents of root.txt?_

We can now open a cmd prompt as administrator, type in the password from Q3, open the file and get our flag!

![image](https://github.com/ne1atonin/ne1atonin.github.io/assets/135453212/88fa4215-87e0-4ba7-b808-5b1f24de20b5)

Thanks for reading!








