# AttackerKB

_Learn how to leverage AttackerKB and find exploits in your workflow!_

* * * 

## Task 1 - I'm Attacking, What Now?

Throughout this room, we'll be examining how we can leverage AttackerKB both as an attacker and defender to gain further insight into the ever-changing landscape of vulnerabilities.

### Q: LET's GO!

A: NA

* * * 

## Task 2 - Lay of the Land

In this task we start from an attacker's perspective with a black-box assessment. Start the machine and attackbox if that's what you're using. (I will be using attackbox) Start scanning to see what's been installed!

### Q2: Start the Machine

A:  NA

### Q3: Enumerate! Scan the machine with Nmap. What non-standard service can be found running on the high-port?

- I run with basic flags:
  - -sS: SYN Stealth Scan
  - -sC: Default Script Scan
  - -sV: Service Detection
    
![image](https://github.com/ne1atonin/TryHackMe/assets/135453212/699883cc-dab8-4f06-9b12-9a4e3ee12f16)

We find an interesting service on port 10000 and can answer Q3.

### Q4: Further enumerate this service, what version of it is running?

- Our answer is in the scan results above

## TBC
