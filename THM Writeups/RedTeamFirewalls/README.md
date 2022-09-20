
#Writeup for RedTeamFirewalls

### Task 3

> What is the size of the IP packet when using a default Nmap stealth (SYN) scan?
Add the size of the TCP header and the IP header to find the total size of the packet. 

`24` + `20` = `44`

> How many bytes does the TCP segment hold in its data field when using a default Nmap stealth (SYN) scan?
A default SYN scan holds `0` bytes in its data field.

> Approximately, how many packets do you expect Nmap to send when running the command ```nmap -sS -F MACHINE_IP```? Approximate to the nearest 100, such as 100, 200, 300, etc.
The answer is `200` packets. -F Scans the top 100 ports and each port is expected to recieve 2 SYN packets.
