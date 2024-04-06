# Part One - Wonderland - Task 1

> This is another long one, bear with me as I complete this room.

***Enter Wonderland and Capture the Flags!***

> This is part one (titled "Wonderland") of the two part challenge series "Wonderland".


## Enumerate!

`nmap -sS -sC -A [target_ip]`

![Pasted image 20240123092251](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/a0e85cea-8b75-41f4-8cef-7ed3a797cb28)

We have a web server, let us navigate to ip:

![Pasted image 20240123092632](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/fc68c5a8-0073-43f9-8baa-961ac74d1949)

Should we follow the white rabbit! Follow We must!

Checking page source code, we see the image is in a directory:

![Pasted image 20240123093210](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/6defc8ef-c1bd-41e0-9da8-e6f7735025ab)

And so appending the /img directory to the ip in the address bar leads us here:

![Pasted image 20240123093336](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/d2582526-2b1d-441b-af94-3063aa808041)

All I can tell for now is that we have a subdirectory for images, with two .jpg and one .png.

Let's enumerate for any subdirectories using gobuster!

![Pasted image 20240123094213](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/532a4eef-f94d-4fc5-94c8-f9509777bacb)

Nice! Let's go down the rabbit hole! Starting from the top, let's check each directory:
/r brings us to a page titled "Follow the White Rabbit" with a quote and hint, apparently:

![Pasted image 20240123094425](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/7ca50cc8-2338-4dc8-9660-6e69e2c6c433)

Next directory, /poem:

"The Jabberwocky" - a poem from the Alice books by Lewis Carroll.

![Pasted image 20240123094720](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/b1f875f1-448c-4fbf-b6aa-96b29e158e07)

In one of my university classes I did a study on the Alice books. I remember learning that the poem is written using [portmanteaus](https://literarydevices.net/portmanteau/), or words that are formed by two or more words combined, creating a new concept. A common example is "brunch".

> Off topic slightly...but I've always had a fascination with wordsmithing, word puzzles (cryptographs, etc.), acrostic, mathematics, etc. I have followed [Martin Gardner](https://martin-gardner.org) since I was young.  If this stuff interests you, his writings are definitely worth checking out.

Back to the hole! The other directories are not leading anywhere, so I am going further "down the rabbit hole" to enumerate for further subdirectories. Thinking about it, rabbit starts with "r", so let's start there.

![Pasted image 20240123101721](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/7594137e-f452-4664-827b-f5a155d34649)

Ok! Now we have an /a subdirectory within /r. I think we can go further on this trail...
Just checking the directory in the browser gives us a continuation of the message we found in /a, which is actually the conversation between Alice and the Cheshire Cat:

Back to /r:

![Pasted image 20240123102710](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/87f8698b-69fe-4762-abe3-ffa9dc289c8d)

And we'll continue through the subdirectories:

Now, /r/a/:

![Pasted image 20240123102741](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/35cf8bb6-796d-4c9b-afe2-6c86edfdc39f)

/r/a/b/:

![Pasted image 20240123102809](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/2033dd25-b732-4c7d-b17c-3750c0368645)

/r/a/b/b:

![Pasted image 20240123102857](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/1fd1f1b7-6ccf-4d4e-86ad-0c03359e3061)

/r/a/b/b/i:

![Pasted image 20240123102935](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/1ba6006a-27c5-42ed-915f-3025f48e8397)

/r/a/b/b/i/t/:

![Pasted image 20240123103214](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/83b073ee-7114-4b48-a938-da80ba01bf40)

So, after 5 "Keep Goings" we have something new, "Open the door and enter wonderland..." I'm assuming this is a clue that there is a door, which may need a key!

Just for kicks, checking the page source at this point, we see this, though I'm not sure if it has any meaning:

![Pasted image 20240123103551](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c5bdd774-62ed-47ee-a229-f084ed85b8aa)

Actually, since we found an SSH port open in our Nmap scan, maybe that's our key to open the door!

![Pasted image 20240123104014](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/f35630aa-6950-478b-9735-22becc6897b7)

Yep!

![Pasted image 20240123104115](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/a2298072-d387-40f8-8eeb-19a9a4adee40)

But unfortunately, no user.txt flag. We do have root.txt, but no permissions to open it:

![image](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/ce24d7d7-7cce-4c47-9ec8-cf51bacce6ef)

But, as shown above, we can run the script we found as rabbit? I'm starting to think rabbit = root?

Anyways, opening the file walrus_and_the_carpenter.py, we find this:

![Pasted image 20240203152309](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/42e3d88b-a83b-4fba-994b-f7c97b918e24)

![Pasted image 20240203152443](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/3b264b4a-86cf-41bb-be5b-a9fd3409d8a6)

![Pasted image 20240203153530](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/3efdbbd2-7354-4554-a0b2-f8d2661799d1)

It's a poem! With a module called "random" and a variable called "poem".

Running the script, we see that it apparently chooses ten random lines and prints them to the terminal:

![Pasted image 20240203153425](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/c3e9cca4-dd9a-4353-8dc9-898d3e20b804)

Second run:

![Pasted image 20240203153650](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/19da4d6e-98dd-4204-b2f3-594d2db7fb71)

And a third for kicks:

![Pasted image 20240203153806](https://github.com/ne1atonin/TryHackMe-WriteUps/assets/135453212/da5d389f-26a4-47f2-bee3-e8964cd91a28)

According to the [python docs](https://docs.python.org/3/library/random.html#functions-for-sequences), random is a module that returns a list of elements according to the defined weight (in our case it's 10), while the split function returns strings split into chunks, as we see in the  code, it prints a tab before the actual line based on \t.

```
line = random.choice(poem.split("\n"))
print("The line was:\t", line)

```

So, what to make of this?

> We will find out, but unfortunately I had to cut out and pick this back up later. TBC


