# Simple CTF

With our target machine and attackbox deployed, we'll run our default nmap scan and see what we get.

```nmap -sV -sC -O 10.10.166.73```

Once that finishes we can answer the first two questions.

> How many services are running under port 1000?

`2` 

We have an FTP service on port 21, and a webserver running on port 80.

> What is running on the higher port?

`SSH`

SSH is running on a non-standard port here.

Let's check the website out first...

![DefaultPage](https://i.ibb.co/BthRwnJ/Default-Page.png)

aaand it's just a default welcome page.

Let's see if the FTP version our nmap scan found is vulnerable. The box is using `vsftpd 3.0.3`

Well it's vulnerable to a remote DoS attack, but that's irrelevant for this challenge. Let's try logging in anonymously.

So once inside the FTP server, we land in the root directory. `ls` shows us that there's a pub directory, once inside pub there's a text file called ForMitch.txt

Let's transfer that to our machine and read what it says.

![ForMitch](https://i.ibb.co/610pSc1/ForMitch.png)

The note says:

> Dammit man... you'te the worst dev i've seen. You set the same pass for the system user, and the password is so weak... i cracked it in seconds. Gosh... what a mess!

So from that we can gather that there's most likely a user named Mitch.

I'm feeling kind of stuck here, the next question is:

> What's the CVE you're using against the application

but so far I haven't been able to find an exploit that suits our needs for the services we enumerated.

Let's try and scan the website with dirb for any hidden pages.

`dirb http://10.10.166.73 /usr/share/wordlists/dirb/common.txt`

Nice! We found a CMS interface at http://10.10.166.73/simple

It appears to be CMS Made Simple version 2.2.8 and it looks to be vulnerable to an SQL injection attack. More specifically, `CVE-2019-9053`

Let's try and crack mitch's account using that exploit.

`python2 46635.py -u http://10.10.166.73/simple/ --crack -w /usr/share/wordlists/SecLists/Passwords/Common-Credentials/best110.txt`

After installing some missing modules (it's a python2 script and using pip install on the THM Attackbox defaults to trying to install the module for python3, to fix this I used `python2 -m pip install some_module` to specify it was for python2 and not python3) we found a username, and cracked the password.

![Cracked](https://i.ibb.co/tZTFTxQ/Cracked.png)

We can now succesfully login to the SSH server on port 2222 with the credentials we just found.

> What's the user flag?

`cat user.txt`

> Is there any other user in the home directory? What's its name?

`ls /home`

> What can you leverage to spawn a privileged shell?

`sudo -l`

Looks like we can use `vim` to spawn a privileged shell. Let's check https://gtfobins.github.io/ for more details.

`sudo vim -c ':!/bin/sh'`

`whoami` to confirm we're root.

`cd /root` then `ls -alh`

`cat root.txt`

And we're done!

This was a fun box to work through as a beginner. I had trouble with the python exploit because of the missing modules, but some quick googling fixed the issue. Once we got an initial foothold through SSH, the rest of the questions were a piece of cake. Truly a great beginner box.

