Let's start off with an nmap scan.

```nmap -sC -sV -O 10.10.87.196```

`ftp` `ssh` and `http` are open on standard ports.

Let's see what's on the FTP server first, since it allows anonymous login.

Two files in here. "locks.txt" and "task.txt" let's grab them both and see what they say.

Meanwhile, we'll check out the website and start enumerating it.

`dirb http://10.10.87.196 /usr/share/dirb/wordlists/big.txt` and ``dirb http://10.10.87.196 /usr/share/dirb/wordlists/big.txt` -X .php`

Going back to the files we grabbed from the FTP server, it looks like "locks.txt" contains a list of passwords. Let's use that list to try and brute-force the ssh service we found. We'll use Hydra for this.

As for the user, between the website and the note on the ftp server we've found 6 potential usernames so far.

-  Spike
-  Jet
-  Ed
-  Faye
-  Lin
-  Vicious

Let's go through each of them using hydra. First let's add all the users to a file and then pass it onto Hydra along with "locks.txt"

Then we'll run the following hydra command:

`hydra -v -V -u -L users.txt -P locks.txt -u 10.10.87.196 ssh`

> -u to loop around users, -L to specify login file, -v for verbose mode, -P for password file, -t to specify threads, and -u to specify IP, also specify the ssh service at the end

So our first loop didn't find anything. I ran it again but this time making the usernames lowercase and we got a hit with the username "lin".

SSH in and grab the user flag from the desktop and let's move on to escalating our privileges.

`sudo -l` (our sudo password is the same as our ssh password)

We can run `tar` as root so let's check out GTFOBins and see what we can do with that. 

`sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh` is our exploit.

Run it and we should have root. Let's grab the flag from /root and we're done!

Pretty simple box, one of the few so far where I haven't had to look anything up besides some command syntax and of course the GTFOBins code.


