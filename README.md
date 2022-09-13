# The-Source
Collection of useful knowledge, commands, and shortcuts.

## Working with packets

Use `tcpdump` to read .pcap files in the command line. Useful for when you don't have access to WireShark.

```tcpdump -qns 0 -X -r file.pcap```

### Snort Rules

Detect login attempts with the "Administrator" account and a wrong/no password.

```alert tcp any 21 <> any any (msg: "FAILED LOGIN"; content: "331 Password"; content: "Administrator"; sid: 1000004; rev:1;)```

Dump raw packet data

```sudo snort -v -X -r snort.log.xxxxxxx```

Run Snort as IPS on interfaces eth0 and eth1

```sudo snort -c /etc/snort/snort.conf -q -Q --daq afpacket -i eth0:eth1 -A full```

## Reconnaissance

### NMAP

Default scan suitable for most networks.

```nmap -sV -sC -O <ip>```
