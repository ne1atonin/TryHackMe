# Res
Hack into a vulnerable database server with an in-memory data structure in this semi-guided challenge (tag: easy)

## Task 1 - Resy Set Go

### Enumeration

* * *

Q1: Scan the Machine, how many ports are open?

We start with a standard nmap scan:

`nmap -sC -sV -A -p1-10000 [machine IP]`

- -sC: Default Script Scan
- -sV: Service Detection (kind of redundant with -A)
- -A: Enables OS and version detection, script scanning and traceroute
- -p1-10000: Specify that I want ports 1-10000 scanned (this takes a while, but my first scan of the default 1000 ports only revealed port 80 open, and this is not the answer to Q1)

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ea5214c5-9059-438f-93af-f0e4d1b43f8a)

Now we can answer Q2, Q3 and Q4 as well.

* * * 

Q4: Compromise the machine and locate user.txt

We get a hint regarding writing to directory (Apache), which we found on port 80

The web server on port 80 gives us the standard Apache landing page, and nothing stands out in the source code. 

I tried a gobuster scan to check for any hidden directories, but nothing here either

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8109dee4-69fb-49d3-b85c-3e3b461f9e93)

So, moving on, I just did a web search for "enumerating redis". One of the results was from the company site, but the top result came from [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/6379-pentesting-redis), so I checked it first.

Very extensive info on this...but the key takeaway is that it's a text-based protocol, which means we can connect via the redis-cli, which we will install on our attacking machine using:

`sudo apt-get install redis-tools`

So, once that is installed, we use:

`redis-cli -h [target machine IP]`

Which calls the redis command line

By default, Redis can be accessed ***without credentials***, but can also be conifgured to support _only password_, or _username + password_. We can check this simply by entering the `info` command.

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ccf3bf35-f3b9-49dc-beea-faf8eed282ef)

And so here we also see a potential username and config file.

# TBC
