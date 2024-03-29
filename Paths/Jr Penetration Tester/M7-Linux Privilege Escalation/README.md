# Task 1 - Introduction

# Task 2 -

# Task 3 - Enumeration

# Task 4 -

# Task 9 - Privilege Escalation - Cron Jobs

Cron Jobs are task automations using the cron command-line utility for Unix-type operating systems. Common examples include creating backups at regular intervals, clearing the cache, sending out notices, etc. A CronJob is used to run the scripts or binaries, scheduled at recurring intervals depending on the need. 

The default privilege is according to the owner of the CronJob, NOT the current user. If properly configured, a CronJob is not inherently vulnerable. That said, if we can find a task that runs with root privileges, and we can modify the script, it will run with root privileges as per the owner of the job.

The CronJob configs are stored as crontabs (tables), showing the next task scheduled to run. Each user has their own crontab file and tasks can obviously be scheduled to run whether the user is logged in or not. 

Interestingly, the file holding system-wide CronJobs are stored under `/etc/crontab` and can be viewed by any user.

As we see below, the backup.sh script is run as root!

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/0f562fa5-0459-4636-8cee-e5fe2fc198e2)

![Pasted image 20231031115752](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ae2aeca2-a04d-4d45-9057-8ea8016c7325)

Note: I had to chmod +x to actually get the connection

> TBC

* * *

# Task 10 - Privilege Escalation - PATH


# Task 11 - Privilege Escalation - NFS

SSH to the target machine with the low-privilege user credentials (I'm using AttackBox):

	Username: karen
	Password: Password1

- Privilege escalation vectors also exist using network shares and remote management systems, such as SSH and Telnet, which ultimately could lead to gaining root access on the target.

- That said, a vector such as a misconfigured network share can be a valuable means to escalate our privileges.

- NFS (Network File Shares) configuration is kept in the /etc/exports file, which is created upon NFS server installation. This is typically readable by users, as shown below:

![Pasted image 20231101092330](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d72dd01b-f9c9-4f4b-b93d-0201a06b63cd)

As seen above, the "no_root_squash" option is our key element for escalation. If a directory is configured as no_root_squash, it would allow the root user on the client to access files on the NFS server as root

### Privilege Escaltation

#### Remote Exploit

1. We start by enumerating mountable shares from the attacking machine in a new tab:
	`showmount -e [MACHINE IP]`

![Pasted image 20231101094014](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/629f109c-fcb5-493f-8f0b-a525286345d6)

2. Then we create a new directory, (I called it /tmp/squash) and mount one of the shares to our attacking machine's new directory:

![Pasted image 20231101100818](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/b2e3f380-da76-4165-a59d-371278bc1985)


3. We will use a simple executable that will run /bin/bash on the target and save as nfs.c:

![Pasted image 20231101101017](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/1b032cee-994d-44d8-b92a-c68f78dbbcea)

4. Then we compile the code and set the SUID bit:

![Pasted image 20231101101422](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/98ac7b74-ebbd-419f-8443-8bee5a40a774)

5. Now, we can go back to the target machine. We see that both files are present as we worked on the mounted share and thus we did not need to transfer them:

![Pasted image 20231101100708](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/28defae1-41f3-4272-a797-66af8bb7ae66)


6. We are Root! Now we can answer Q3 and Q4!

![Pasted image 20231101102733](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/9937eb73-3782-4336-a8a7-d54311a3a82d)

***On to the next and final task in this Module!***

# Task 12 - Capstone Challenge

***Context:***

We have gained access to a large scientific facility. Try to elevate privileges until we are Root.

Note: This room was designed to help us build a thorough methodology for Linux privilege escalation that will be useful in exams such as OSCP, as well as in Pentesting engagements. Privilege escalation is often more of an art than a science!

I will be using the AttackBox to SSH into the machine using the following credentials:

- Username: leonard
- Password: Penny123

### Enumeration

1. Let's start with the basics
   
   ![Pasted image 20231101104232](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/743960f4-395f-44c8-91d2-add1c83ef698)

* * *

> TBC
