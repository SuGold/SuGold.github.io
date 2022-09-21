
# Pickle Rick Writeup

We first navigate to the web application. There's nothing we can click on so we'll run dirbuster to enumerate any hidden directories.

`dirb http://10.10.194.139 /usr/share/wordlists/common.txt`

![Dirb Enum](https://i.ibb.co/DfTnt4Y/dirbEnum.png)

The only interesting link directory seems to be  `assets` but once inside there's really nothing of value there at first glance. We notice it's running Apache version 2.4.18 for Ubuntu but after searching exploit-db we don't find anything that'll give us an initial foothold on the system.

Let's try running an nmap scan and see if there's any interesting ports open on the webapp.

![nmap Scan](https://i.ibb.co/88G0q6q/nmapScan.png)

