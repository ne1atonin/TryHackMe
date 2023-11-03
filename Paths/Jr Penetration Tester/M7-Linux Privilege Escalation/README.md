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
