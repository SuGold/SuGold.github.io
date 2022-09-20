
# Writeup for RedTeamFirewalls

### Task 3

> What is the size of the IP packet when using a default Nmap stealth (SYN) scan?
Add the size of the TCP header and the IP header to find the total size of the packet. 

`24` + `20` = `44`

> How many bytes does the TCP segment hold in its data field when using a default Nmap stealth (SYN) scan?

A default SYN scan holds `0` bytes in its data field.

> Approximately, how many packets do you expect Nmap to send when running the command ```nmap -sS -F MACHINE_IP```? Approximate to the nearest 100, such as 100, 200, 300, etc.

The answer is `200` packets. -F Scans the top 100 ports and each port is expected to recieve 2 SYN packets.

> Approximately, how many packets do you expect Nmap to send when running the command nmap -sS -Pn -D RND,10.10.55.33,ME,RND -F MACHINE_IP? Approximate to the nearest 100, such as 100, 200, 300, etc.

2 packets per port * 100 ports * 4 IPs = `800`

> What do you expect the target to see as the source of the scan when you run the command nmap -sS -Pn --proxies 10.10.13.37 MACHINE_IP

`10.10.13.37`

> What company has registered the following Organizationally Unique Identifier (OUI), i.e., the first 24 bits of a MAC address, 00:02:DC?

Lookup `00:02:DC` on https://macaddress.io/ the answer is `Fujitsu General Ltd`

> To mislead the opponent, you decided to make your port scans appear as if coming from a local access point that has the IP address 10.10.0.254. What option needs to be added to your Nmap command to spoof your address accordingly?

`-S 10.10.0.254`

> You decide to use Nmap to scan for open UDP ports. You notice that using nmap -sU -F MACHINE_IP to discover the open common UDP ports won’t give you any meaningful results. What do you need to add to your Nmap command to set the source port number to 53?

`-g 53`

### Task 4
