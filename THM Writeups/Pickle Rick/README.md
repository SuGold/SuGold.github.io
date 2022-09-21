
# Pickle Rick Writeup

We first navigate to the web application. There's nothing we can click on so we'll run dirb to enumerate any hidden directories.

`dirb http://10.10.194.139 /usr/share/wordlists/common.txt`

![Dirb Enum](https://i.ibb.co/DfTnt4Y/dirbEnum.png)

The only interesting link directory seems to be  `assets` but once inside there's really nothing of value there at first glance. We notice it's running Apache version 2.4.18 for Ubuntu but after searching exploit-db we don't find anything that'll give us an initial foothold on the system.

Let's try running an nmap scan and see if there's any interesting ports open on the webapp.

![nmap Scan](https://i.ibb.co/88G0q6q/nmapScan.png)

Looks like we have an SSH port open, maybe we can leverage that later after we get some credentials.

Since this is an Apache server, it's safe to assume it's using PHP somewhere. Let's try running dirb again but this time with the `-X .php` option to check for PHP files.

`dirb http://10.10.194.139 /usr/share/wordlists/common.txt -X .php`

![Dirb Enum2](https://i.ibb.co/cCdpRDR/dirb-Enum2.png)

Nice! We found a login page.

Ok so we've done a decent amount of enumeration, let's go back to the index page and see if we missed anything. We'll check the page source to make sure there aren't any hidden clues.

![indexSource](https://i.ibb.co/8Y6GV5M/index-Source.png)

Awesome, we now have a username (R1ckRul3s) for what we can assume is the login portal we found earlier. Let's try and bruteforce it now using Burp.

Activate the proxy in FireFox and turn intercept on in Burp and capture a login request. Forward that request to intruder to set up our brute force attack.

Since bruteforcing can be pretty slow on free version of Burp. We'll use the best1050 password list and hope we get a hit.

No luck...we'll have to try another method. This is an "Easy" room so we can assume that the creator isn't expecting us to spend days trying to brute force the password. Let's go back and see if we missed something in our initial recon.

There was a `robots.txt` that we initially skipped over. This file normally tells a search engine which pages it's allowed to index, but this one just says "Wubbalubbadubdub"

We've got nothing else to go off of so let's try using that as a password.

![Command Panel](https://i.ibb.co/HD7kgm7/command-Panel.png)

Well, can't say it was a surprise but it worked.

We now have access to a convenient web-based terminal we can execute commands on.

Let's do some more recon. A simple `ls -alh` turns up two interesting files. `Sup3rS3cretPickl3Ingred.txt` and `clue.txt`

`cat Sup3rS3cretPickl3Ingred.txt` and `cat clue.txt` tells us the command is disabled. Checking our permissions we can see that we don't have read permissions on those files with our current privileges and we can't `chmod` our way out of this.

Let's try and use curl to bypass the web terminal and read those files from our machine.

![curlRead](https://i.ibb.co/q520qv1/curlRead.png)

Great! We found the first flag.

Checking out the other sections on the web app, we see a message that says "Only the **REAL** rick can view this page.." We can assume that there's another user we're gonna have to find. For now however, let's try and traverse directories and see what other interesting files are on the system.

Let's check the home directory.

`cd ../../../../home; ls -alh`

Now let's check what's in rick's folder.

`cd ../../../../home/rick; ls -alh`

There's an ASCII file called "second ingredients", again we can't use cat. Let's try `less`

`cd ../../../../home/rick; ls -alh; less "second ingredients"`

And we found our second flag.

Onto the last one.

So previously, we learned that only the **REAL** rick can access certain pages. We'll need to escalate our privileges and see if that's good enough.

`sudo -l` in the web terminal to check our current sudo privileges.

> User www-data may run the following commands on ip-10-10-194-139.eu-west-1.compute.internal: (ALL) NOPASSWD: ALL

Well that's convenient. We can run any command as sudo.

Let's see what's in the root directory

`sudo ls /root`

There's a file called "3rd .txt", that's probably the flag. Let's check the contents.

`sudo less /root/3rd.txt`

Got it!

Overall, this was a pretty fun box for a beginner. I disliked having to guess that the password was in the robots.txt file, but it does teach you to try every single possibility.


