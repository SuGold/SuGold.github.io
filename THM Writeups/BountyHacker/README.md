Let's start off with an nmap scan.

```nmap -sC -sV -O 10.10.87.196```

`ftp` `ssh` and `http` are open on standard ports.

Let's see what's on the FTP server first, since it allows anonymous login.

Two files in here. "locks.txt" and "task.txt" let's grab them both and see what they say.

Meanwhile, we'll check out the website and start enumerating it.

`dirb http://10.10.87.196 /usr/share/dirb/wordlists/big.txt` and ``dirb http://10.10.87.196 /usr/share/dirb/wordlists/big.txt` -X .php`

Going back to the files we grabbed from the FTP server, it looks like "locks.txt" contains a list of passwords. Let's use that list to try and brute-force the ssh service we found. We'll use Hydra for this.

As for the user, between the website and the note on the ftp server we've found 6 potential usernames so far.

- Spike
-  Jet
-  Ed
-  Faye
-  Lin
-  Vicious

Let's go through each of them using hydra. First let's put add all the users to a file and then pass it onto Hydra along with "locks.txt"

Then we'll run the following hydra command:

`hydra -v -V -u -L users.txt -P locks.txt -u 10.10.87.196 ssh`

> -u to loop around users, -L to specify login file, -v for verbose mode, -P for password file, and -u to specify IP, also add the service at the end

