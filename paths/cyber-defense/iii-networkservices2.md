# Network Services 2

## Task 1 - Get Connected
> We are using AttckBox for this room.

## Task 2 - Understanding NFS

### What is NFS?
NFS stands for "Network File System" and allows a system to share directories and files with others over a network.  NFS allows users and programs access to files on remote systems, as if they were local files. This is done by "mounting" all, or a portion of a file system on a server. The mounted portion can be accessed by clients with respective priveleges according to each file.

### How Does NFS Work?

## Task 3 - Enumerating NFS

### What is Enumeration?
- A process which establishes an active connection to the target host to discover potential attack vectors in the system. The same may be used for further exploitation of the system.
- Critical phase as the info gathered will inform attacks.
  
### Requirements
- For advanced enumeration, we will use the tool, NFS-Common (install using "sudo apt install nfs-common")
  
### Steps:

##### Step 1 - nmap Port Scan:

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/2b1873e0-f3e4-45e9-9345-8d50ccbe1622)

Q1: How many Ports are Open?

A:

Q2: Which Port Contains the Service we're Looking to Enumerate (nfs/acl)?

A:

Q3: Using /usr/bin/showmount -e {IP} to list the NFS shares, what is the name of the visible share?

A:  

##### Step 2 - Mounting NFS Shares

We will need to create a directory (mount point) on our machine with which we can access the content shared by the host server. This can be created anywhere on the system. 
	
From the /root directory, use: 
  `mkdir /tmp/mount
	
Then we will use the mount command to to mount the NFS share to our local machine:
		
  `sudo mount -t nfs IP:share /tmp/mount/ -nolock	
	
Q4: Change to the directory where we mounted the share, what is the name of the folder inside?

A:  

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/43fbad4e-1ad0-414a-bb94-7556b1649ed8)

Q5: Have a look around.

A: N/A

Q6:  Looking through the folders, which of these could contain keys that could provide remote access to the server?

A: 

Q7: Which of these keys is most useful to us?

A:  

Copy this file to a location on our local machine:

`cp id_rsa /tmp/mount

Change permissions to "600" using `chmod 600 id_rsa

Assuming this could be a user's home directory, let's work out the name of the user this key corresponds to

Q8: Can we log into the machine using:

`ssh -i  id_rsa [username]@target ip

A: Y

## Task 4 - Exploiting NFS

* * *
# TBC
