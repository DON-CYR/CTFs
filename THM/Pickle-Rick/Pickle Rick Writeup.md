## Today we’re doing a nice short and simple walkthrough on the famous [Pickle Rick](https://tryhackme.com/r/room/picklerick) room on TryHackMe.

### Port Scanning & Enumeration
First things first we are going to scan the machine to find any open ports that we can exploit using the following command:

```
nmap -sV -sC 10.10.160.35
```
> -sV: Finds the service version on the port

> -sC: Enables Nmap’s Scriping Engine (NSE) which is great for port discovery


As you can see we have two open ports SSH and HTTP.

Since SSH is only useful if we have the proper credentials lets just focus on HTTP instead.

Enter the IP in your web browser to display the web page and we can find this:


To do some more digging we should try to view the source code and see if there’s any hidden things in there:


Looks like there’s a comment here with a username. Lets be sure to make a note of that for later.

### Finding Hidden Directories & URLs
Now that we’ve got a username let’s do some more digging to find where we can put it to use by uncovering some hidden directories on this page using gobuster.

Enter the following command:
```
gobuster -u http://10.10.160.35 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -x php,txt,
```
> -u: specify the URL you want to use

> -w: specify the wordlist file to use

> -x: for any file extensions to look for while searching


Looks like we managed to find some! Let’s try the text file first and see what we have. Just enter /robots.txt after the URL.


Looks like we just found ourselves a potential password we could use for a login of some kind. Thankfully we also found a /login.php that we could explore as well.


Seems like we have a login portal of some kind and we might have a potential user and password that we could use as well from the previous steps!

### Capturing the Flags
Lets login using the username we found in the HTML comment earlier and the password from the robots.txt file.


Success! We’ve reached the Command Panel. We could use this to potentially look for the 3 flags.

With the ls command we can view what’s on this filesystem:


If we try to view the txt file using the cat command we get an error BUT we can also use the less command as a workaround ;) to view the text files.


And as you can see we have now obtained the first flag for this challenge easy-peasy.

Next if we use the less command again on clue.txt we can see a nice hint that we can use to find the other two flags.


Alright with this clue let’s navigate to try to navigate to the /home directory using the cd command and ls to find a directory called second ingredients


Now, using the same command to view files previously we can view what is contained in this directory by typing:
```
less '/home/rick/second ingredients'
```
Nice! Now we’ve got the second ingredient aka the 2nd flag.


Now to find the 3rd flag, let’s see if we can get into the root directory using the ls /root command.

When trying to it looks like we get an error, but lets try to see if we can elevate our privileges using by typing:
```
sudo ls /root
```
Looks like we’ve been able to locate where the 3rd ingredient is! Now lets use the less command on the 3rd.txt file to get the final flag!


And that’s all of them! Challenge complete!
