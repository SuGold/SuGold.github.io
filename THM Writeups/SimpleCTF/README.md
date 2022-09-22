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
