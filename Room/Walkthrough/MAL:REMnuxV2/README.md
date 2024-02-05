# REMnux v2


## Task 1 - Intro

This room is a rewrite of an original REMnux room. It is designed for hands-on work with REMnux and its tools...vs. reading cheatsheets for tasks. I've been wanting to dive in and see what REMnux is really about. This room isn't designed with point-farming in mind, but to provide enough guidance throughout that entices a curiosity to explore the topics & resources introduced.   

This room includes the following activites:

- Identifying and analyzing malicious payloads of various formats embedded in PDF's, EXE's and Microsoft Office Macros
- Learning how to identify obfuscated code and packed files - and in turn - analyze these.
- Analyzing the memory dump of a PC that became infected with the Jigsaw ransomware in the real-world using Volatility.

At the end of the room we have additional, useful material about some of the topics covered, as well as cheatsheets and related articles.

LET's GO!

* * * 

## Task 2 - Deploy

I have deployed the machine instance from inside the room.

* * *

## Task 3 - Analyzing Malicious PDFs

PDF's are capable of containing various types of code that can be executed without the user's knowledge, including:

- Javascript
- Python
- Executables
- Powershell Shellcode

Task 3 covers Javascript embeds and analyzing embedded executables.

### Looking for Embedded Javascript

Javascript can be easily embedded into a PDF file, and executed upon opening, unbeknownst to the user. Javascript, like other languages covered later, allows a malicious actor the opportunity to gain a foothold, and to establish a mechanisim whereby malware can be downloaded and executed.

### Practical

We'll be using `peepdf` to begin a precursory analysis of a PDF file to determine the presence of Javascript. If found, we will extract this Javascript code (without executing it) for our inspection.

While the room does not instruct this, we must first navigate to the directory: /Tasks/3/ to find the file called "nonsuspicious.pdf" *(NOT "demo_notsuspicious.pdf")*

Once inside the ~/Tasks/3/ directory, we can simply enter the command: `peepdf notsuspicious.pdf`

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ae1148c9-0c9b-4524-9a82-88e64b875456)

As shown above, the output confirms the presence of Javascript, as well as how it is executed... **OpenAction** will execute the code when the PDF is opened.

To extract this Javascript, we can use `peepdf`'s "extract" module. This requires a few steps to set up but is fairly trivial.

1. The following command will first create a script file for `peepdf` to use called "extracted_javascript.txt":

`echo 'extract js > javascript-from-notsuspicious.pdf' > extracted_javascript.txt`

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/326dd643-1363-47c3-a220-b9049c05a596)


2. Now we will tell peepdf the name of our script: "extracted_javascript.txt", and the PDF file we are extracting from:

   `peepdf -s extracted_javascript.txt notsuspicious.pdf`

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/5867f5a7-7996-4921-9434-9240b7d4f08a)

The script created (red), causes the Javascript from our original file (green) to output into a file called "javascript-from-notsuspicious.pdf" (yellow), as shown above.

Now, we can use `cat javascript-from-notsuspicious.pdf` to see the contents.

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/02b45bf5-356a-452c-a5af-9afd98ab1c92)

As shown above, the PDF file we analyzed contains some javascript code, with our flag for Q2:

`app.alert("THM{*_flag_*}");`

> Note: This might seem a bit convoluted, and as I was writing it up, I had to re-read my notes several times, because they seemed out-of-sync with the steps I took to acheive the first flag. Because I took notes and screenshots as I went along, it's definitely correct, as indicated by the answers. That said, the room itself has slightly different output in many steps. I did my best to make logical sense of it in the write-up so far. Because of the time it took for me to write it out, I have to come back and complete the next few tasks at another time.

Thank you for reading so far!

## TBC
