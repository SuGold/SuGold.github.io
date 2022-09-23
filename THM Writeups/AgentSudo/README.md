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

We can assume we have a username now (chris) which we'll use to try and access the FTP service.




