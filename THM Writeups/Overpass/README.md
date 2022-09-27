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
        window.location = "/admin"````
