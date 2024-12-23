# RootMe Writeup TryhackMe 
### [Lab Link](https://tryhackme.com/r/room/rrootme)

Today we’re going to be rooting this machine which is pretty easy and great practice for beginners. 


## Task 1 - Deploy the machine

Start the machine and wait til the given IP address shows up and make sure your VPN is up and running 

## Task 2 - Reconnaissance

First thing we have to do is scan the target machine using Nmap

![1]

After scanning, we can see that there are two ports open which are Port 22 SSH and Port 80 HTTP which is a web server. 

We can also see the version of Apache its running which is 2.4.29

#### 1. Scan the machine, how many ports are open?
2
#### 2. Which version of Apache is running?
2.4.29
#### 3. What service is running on port 22?
SSH

Next we need to find the directories on the web server using the Gobuster tool. 

By using the following command we can find some directories that may be hiding:
```
gobuster dir -u http://[machine_ip]/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```
This allows us to search for potential directories using the dirbuster wordlist that comes pre-installed with kali. 

![2]

After the search, it seems like we’ve discovered some pages called /uplaods and /panel.

#### 4. What is the hidden directory?
/panel/


## Task 3 - Getting a shell

To get a shell spawn we are going to utilize the uploading files feature on this site to exploit using a php script. 

First we need to generate one. I did this by going to revshells.com and using the PHP PentestMonkey template. 

![3]

Once the file is saved as a .php we are going to test to see if the script works. 

We need to spawn a netcat listener to listen for the connection. We do this by using the following command: 
```
nc -lvnp [PORT]
```
Now we can upload the file and try to execute it at /uploads, but it seems like it doesn’t allow .php files. 

![4]

Nothing too serious, as the hint indicates we are going to have to perform a file upload bypass. 

> You can do this by simply adding to the file extension such as “example.php.html” or “exmaple.php1”

Doing so allows us to upload!

![5]

Now just head over to /uploads and click on the file. Make sure the listener is up and live too. 

We now have a shell!

![6]

Now that we have access, lets turn it into a proper shell using the following command: 
```
python3 -c ‘import pty; pty.spawn(“/bin/bash”)’ 
```

Doing so allows us to get the user.txt file a lot easier.

![7]


## Task 4 - Privilege Escalation

Let’s start escalating some privileges!

We need to search for files with SUID perms and we can do this by using the command given in the hint. 
`find / -user root -perm /4000`

![8]


After scrolling we can see a python file which can be run as root.


#### 1. Search for files with SUID permission, which file is weird?
/usr/bin/python


To escalate privileges we are going to use gtfobins https://gtfobins.github.io/ and searching for a python file exploit. 

![9]

Great now let's put it to use with the command: 
```
python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
```

We got root!

Now let's find the root flag in the root directory and complete this challenge.

![10]

There you have it, both flags acquired, this machine has been rooted!

[1]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215222.png
[2]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215322.png
[3]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215402.png
[4]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-22%20192555.png
[5]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-22%20192541.png
[6]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215511.png
[7]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215552.png
[8]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215615.png
[9]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-22%20194824.png
[10]: https://github.com/DON-CYR/CTFs/blob/main/THM/RootMe/pics/Screenshot%202024-12-20%20215552.png
