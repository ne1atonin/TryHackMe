# Task 1 - Intro

The room is designed for hands-on work with REMnux and its tools...vs. reading cheatsheets for tasks; let's see what REMnux is really about...I'm looking forward to working with this distro/toolset.

The objective isn't to farm points/questions, but to learn and develop a curiosity to explore the topics & resources introduced.   

We will work through the following activites:

- Identifying and analyzing malicious payloads of various formats embedded in PDF's, EXE's and Microsoft Office Macros (the most common method that malware developers use to spread malware today)
- Learning how to identify obfuscated code and packed files - and in turn - analyze.
- Analyzing (using Volatility) the memory dump of a PC that became infected with the Jigsaw ransomware in the real-world.

At the end of the room there is additional, useful material about some of the topics covered, as well as cheatsheets and related articles.

LET's GO!

## Task 2 - Deploy

> We are using THM to deploy the REMnux instance, and will follow along through the room tasks.

## Task 3 - Analyzing Malicious PDF's

PDF's are capable of containing many more types of code that can be executed without the user's knowledge. This includes:

- Javascript
- Python
- Executables
- Powershell Shellcode

Not only will this task be covering Javascript embeds (like we did previously), but also analysing embedded executables.

### Looking for Embedded Javascript

We previously discussed how easily javascript can be embedded into a PDF file, whereupon opening is executed unbeknownst to the user. Javascript, much like other languages that we come on to discover in Task 4, provide a great way of creating a foothold, where additional malware can be downloaded and executed.

### Practical

We'll be using `peepdf` to begin a precursory analysis of a PDF file to determine the presence of Javascript. If there is, we will extract this Javascript code (without executing it) for our inspection.

While the room does not instruct this, we must first navigate to /Tasks/3/ directory to find the file called "nonsuspicious.pdf" *(NOT "demo_notsuspicious.pdf")*
Once inside the ~/Tasks/3/ directory, we can simply do `peepdf notsuspicious.pdf`

![Pasted image 20240205162059](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c58bcbe2-6004-4fa2-8046-d1377d805b40)

Note the output confirming that there's Javascript present, but also how it is executed? **OpenAction** will execute the code when the PDF is launched.

To extract this Javascript, we can use `peepdf`'s "extract" module. This requires a few steps to set up but is fairly trivial.

1. The following command will create a script file for `peepdf` to use:

`echo 'extract js > javascript-from-notsuspicious.pdf' > extracted_javascript.txt`

![Pasted image 20240205163114](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/bdb771d4-e97d-41aa-bc0f-5db045be0a08)


The script will extract all javascript via `extract js` and pipe `>` the contents into "javascript-from-notsuspicious.pdf"  

2. We now need to tell `peepdf` the name of the script (extracted_javascript.txt) and the PDF file (notsuspicious.pdf) that we want to extract from: 

`peepdf -s extracted_javascript.txt notsuspicious.pdf`

**To recap:** "extracted_javascript.txt" (highlighted in red) is our script, where "notsuspicious.pdf" (highlighted in green) is the original PDF file that we think is malicious.

![Pasted image 20240205164418](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/a5d63502-f710-4cc4-bd8e-b8f2aad966a1)

The script will cause the Javascript to output into a file called "javascript-from-notsuspicious.pdf" (highlighted above in yellow). This file now contains our extracted Javascript, and we can simply `cat` this to see the contents.

![Pasted image 20240205165355](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ed905f6a-6c71-42fc-a50f-9994ec01ba4f)

As shown above, the PDF file we analyzed contains javascript code:

`app.alert("THM{*_flag_*}");`

And we have our answer to Q1 and the flag for Q2.As shown above, the PDF file we analyzed contains javascript code:

`app.alert("THM{*_flag_*}");`

And we have our answer to Q1 and the flag for Q2.

* * *

## TBC 
