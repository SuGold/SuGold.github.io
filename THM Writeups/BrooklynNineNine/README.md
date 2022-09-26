Start off with an nmap scan.

`nmap -sC -sV -O 10.10.106.189`

Looks like we have 3 ports open. 21, 22, 80 with FTP, SSH and HTTP services respectively.

We can login to get ftp service anonymously and it looks like there's a file there called "note_to_jake.txt" so let's go ahead and grab that.

The file asks Jake to change his password because it's too weak. Might be able to crack it later on if we get the hash. For now let's move onto the website.

The page is a full background image, no links we can click on but there's a comment when viewing the page source that says "Have you ever heard of steganography" so let's use `steghide` to see what we find.

`wget http://10.10.106.189/brooklyn99.jpg`

`steghide extract -sf brooklyn99.jpg`

It needs a passphrase so let's use binwalk to see if there's any hidden files within the image.

`binwalk brooklyn99.jpg -e`

Looks like it's just the image.

The error we get from steghide when using an empty password says:

> steghide: can not uncompress data. compressed data is corrupted.

Let's try and use `stegcracker` to crack the passphrase

`stegcracker brooklyn99.jpg /usr/share/wordlists/SecLists/Passwords/probable-v2-top12000.txt `

After a few seconds, the password turns out to be "admin"

Let's extract the data hidden within the jpg now. We get a file called "note.txt"

The note tells us that Holts Password is fluffydog12@ninenine

Let's see where we can use those creds. SSH maybe?

`ssh holt@10.10.106.189`

Nice, we have an initial foothold in the system now. Let's grab the user flag from holts directory.

Now let's try and escalate our privileges.

`sudo -l`

We have sudo perms with nano. Let's see what GTFObins has to say about this.

`sudo nano
^R^X
reset; sh 1>&0 2>&0`

Perfect, we got root.

Let's grab the flag now and we're done.

Wow! This box was super simple. Total time from running the first nmap scan to getting root was around ~10 mins. Definitely a nice confidence booster.

I had to research what program to use to crack the passphrase on the jpeg (`stegcracker`) but after that it was smooth sailing.


