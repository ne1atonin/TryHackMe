# Hijack

***Misconfigs Conquered, identities claimed.***

* * *

## Task 1 - Deploy the Machine and Get the Flags!

### Enumerate!

As always, starting with an nmap scan of the target IP using the following:

`nmap -sS -sC -A -v [Machine IP]`

![Pasted image 20231029094201](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/64071500-221a-45c5-85dc-aef41de366be)

 As shown above, we have five ports open: 21 (ftp), 22 (ssh), 80 (http) and 111 (rpcbind).
- Occassionally when finding an FTP server, it might also allow Anonymous access. But not in this case.
- I did a search for 'port 111 rpc bind' and found some info on [Hacktricks](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rpcbind). Basically, this port provides info between Unix based systems. It's often probed, and gives OS info, as well as services on other uncommon ports as seen above. 
- We also see an NFS share (using version 2, 3 and 4), which is a good find. NFS is a client/server system that allows users to access files over a network and across various architectures and operating systems. Files are treated as if they reside in a local directory. Importantly - NFS has no authentication itself, but may use RPC to authenticate the file request by authenticating the computer making the request - ***Not the User***. As such, a user can run su and impersonate the owner, thus anyone with root access can introduce arbitrary data into the network and extract from the network.  [Oracle.com](https://docs.oracle.com/cd/E19683-01/817-0204/6mg168c10/index.html)
- We also see the mountd running so we know that checks are in place
- We can first use the command: `showmount -e [machine IP]` to check for available shares and export the list.

![Pasted image 20231029094201](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8d747d9f-15f8-4d46-85b2-eee08d5fcfb9)


### Mount the Share!

1. Create a new directory - I'm using /tmp/jack
2. Since we know the partition location: (/mnt/share/) we can mount it to the new directory /tmp/jack.

![Pasted image 20231029130210](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d3734a43-4299-4a9d-ae92-94c93c5eccd2)

3. As shown above, we don't have permissions to cd to the new 'jack' directory, so, we need to create a new user (in my case 'rat'). But we still don't have access, so we need to edit permissions based on the UID for the 'jack' directory:

![Pasted image 20231029132348](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/79e8950c-4f6d-4299-a4b2-f9b7284b06bb)

4. Now we can change user to 'rat' and have access to 'jack'! Here we find our file pulled from the share, giving our creds for the FTP server.

![Pasted image 20231029132106](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/f667e0f8-85a3-43fd-baad-3c9087c6ba9e)

5. With this info we can now log in to the FTP server at our Target IP and find some more goodies. To make it cleaner, I created a new directory called 'hijack' under the root directory on AttackBox, then logged in to the FTP server with the above creds from this directory so that any files would be pulled into that new directory.

![Pasted image 20231029133612](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/3bfc0bd2-7c1a-49c8-b1e7-bce67764902e)

6. Now, we can use `mget .*` to remotely pull the files from the FTP server:

![Pasted image 20231029134114](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/f210b3d0-a9ab-4c6b-ab35-c85e51eb299d)


7. Now we have our files. What we know so far:
	- We have a note from the admin
	- We have a password list
	- We have a website
	- We know that the admin uses a password from the password list file
	- We have a user named rick
	- Login attempts are rate limited 

![Pasted image 20231029134726](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/bf1e94cd-221d-410c-b67f-d64e7fdb3c57)

8. We can now navigate to the website at [Machine IP] port 80 and have a look around.

![Pasted image 20231029094803](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/db5af6a5-7dec-4bd4-9fdf-ca0f2ff49be5)

A few things to note off the bat:
- Clicking on the 'Administration' tab gives a response, and lets us know it's built on php. May be helpful, might not...
  
![Pasted image 20231029095549](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/db71a0e9-cf59-4e7e-8ddb-7786336c1669)

- We know the site has an admin account. And, clicking on the 'Login' tab gives us a form. Again, brute force protections are in place, but we can create an account. I use Test:Tester.
- Before logging in I make sure to start up Burp and intercept my login.

![Pasted image 20231030094834](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/94c7007b-bb7c-4ea4-97ba-38619c58c348)

![Pasted image 20231030100745](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d207042b-0926-44f6-910d-2444130f9dfa)

The site is storing my login in a cookie. Decoding from URL encoding we find that it's Base64 encoded my username and password. Decoding further we see the username, but the password is stored as an MD5 hash. 
- Using [hashes.com](https://hashes.com/en/decrypt/hash) I confirmed this:
  
![Pasted image 20231030103704](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/94e8ff0c-5c3e-4ebc-a189-3cd450022ee6)

## TBC
