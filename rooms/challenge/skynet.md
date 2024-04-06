# Intro

## Task 1 - Skynet

### Deploy and Compromise the Vulnerable Machine!

_Hasta la vista, baby._

Can we compromise this Terminator themed machine?

**Let's Find Out!**

***Note, I am using AttackBox to complete the room.***

### 1.1 Enumerate!

As always, we will begin with a basic nmap scan to look for open ports, services, shares and other leads. I use the following flags:

`nmap -sS -A -v [target_IP]`
-  -sS: TCP SYN scan - Default scan option, quick, relatively unobtrusive ans stealthy, as it does not complete the TCP connection. (AKA half-open scan)
-  -A: Agressive Scan - Enables advanced and agressive options, such as: OS detection, version scanning, script scanning and traceroute. Comprehensive but intrusive.
-  -v: Increased Verbosity - Nmap wil print more info regarding the scan in progress.

![Pasted image 20231130121802](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8d4e3b86-4671-4490-89ec-a41269ffe027)

As we can see above, we have a number of open ports:
- Port 22 - SSH
- Port 80 - A webpage called *"Skynet"*
- Port 110 - A pop3 mail server using Dovecot pop3d
- Port 139/445 - SMB / Samba shares
- Port 143 - IMAP mail server using Dovecot imapd

We will start investigating the web port first, by navigating to the site at:
`http://[target_IP]` 

Here we find some sort of 'Skynet' search:

![Pasted image 20231130125059](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ddda07ea-dc1b-4909-a4f7-532c1507b331)

Playing around and looking at the source gives us no information. Searching and / or using "I'm Feeling Lucky" returns an empty box. So, let's check for directories using Gobuster.

`gobuster dir -u http://[target_IP] -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`

![Pasted image 20231130131403](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/0b56cb93-fbca-4c29-8b3a-9b22011dfd0c)

And, we have some directories to check out!

I go from the top but receive a Forbidden Error from all but /squirrelmail, which brings us to this php login page:

![Pasted image 20231130131829](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c01ced8b-65f8-45b4-8bc6-f05e75452e25)

But we need more info. Let's move on.

I go back and have a look at the nmap script scan results:

![Pasted image 20231130133147](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/a281f3f4-e19e-481d-b35a-5056ee8eaaf5)

Not sure if there's anything helpful here, but we do see that there is a guest account on one of the hosts.

* * * 

# TBC

