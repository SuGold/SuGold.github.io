# The-Source
Collection of useful knowledge, commands, and shortcuts.

## Working with packets

Use `tcpdump` to read .pcap files in the command line. Useful for when you don't have access to WireShark.

```tcpdump -qns 0 -X -r file.pcap```

#### Snort Rules

Detect login attempts with the "Administrator" account and a wrong/no password.

```alert tcp any 21 <> any any (msg: "FAILED LOGIN"; content: "331 Password"; content: "Administrator"; sid: 1000004; rev:1;)```

Dump raw packet data

```sudo snort -v -X -r snort.log.xxxxxxx```

Run Snort as IPS on interfaces eth0 and eth1

```sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A full```

## Reconnaissance

#### NMAP

Default scan suitable for most networks.

```nmap -sV -sC -O <ip>```

## CTFs

#### Finding flags

Find flag specifying filetype and name and output errors to dev/null

```find / -type f -name <flag> 2>/dev/null```

### Privilege escalation

Refer to [https://gtfobins.github.io/](https://gtfobins.github.io/) for more methods.

##### Find files with SUID permissions

```find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null```

If for example we find that python has SUID permissions, we can move to the python directory and escalate privileges with:

```./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'```

#### Setup a python http server so download files from attacker machine to target machine.

```python -m http.server```

## Output formatting

#### Cut

```cat``` the contents of /etc/passwd and pass them through ```cut``` to extract the first field delimited by a colon

 ```cat /etc/passwd | cut -d ":" -f 1```


