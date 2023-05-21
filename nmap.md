# Theory
Exists 65536 port but the 0 is reserved and only can use 65535

The well-know port are 0-1024, but it's the same the port 0 is reserved and can't use
## Types of scan
- `TCP Connect Scans (-sT)` --> Send TCP packets, if the target system respond with a SYN-ACK packet, **the port is open**.
- `SYN "HALF-open" Scans (-sS)` --> nmap don't respond with the final ACK packet and don't establish the full conection. It's **more difficult to detect** because the connection is not full establish
- `UDP Scans (-sU) ` --> Send UDP packet. If the target response is a ICMP packet is close, but it doesn't respond the **port is open**
- `TCP Null Scan (-sN)` --> The flag header is 0. If the targer send RST packet that say that the port is close, but if we didn't recive them that means that is open. This type of scan can by useful for bypassing some types of firewall protections that block other type of scan.
- `TCP FIN Scan (-sF)` --> Send TCP packet with FIN (finish) flag, if the target don't respond with a RST packet that means that is open. It's useful to bypass some firewall.
- `TCP Xmas Scan (-sX)` --> send TCP package with flags FIN, PSH (push) and URG (urgent)

### TCP Scan
This work by three-way handshake (three stages):
1. Our machine send a TCP request to the target with the **SYN flag**, their intention is to establish a connection with the target.
2. The target responds with a **SYN/ACK** flag that is posible to establish a connection.
3. Our machine responds with a **ACK** flag indicating that it has recived the packet and is ready to communicate (**port open**)
![[Pasted image 20230512000223.png]]

But if the connection it isn't allowed the server send tcp packet with a **RST** (reset) flag (**port closed**).

![[Pasted image 20230512001243.png]]

Sometimes when a port is open but is hidden behind a firewall the nmap say *filtered*, this means that the firewall is drop the tcp packet and the server **don't send any packet**. 

Somes firewall are configurate to respond with a RST TCP packet. This complicates the detection a lot.

### SYN Scans / stealth scans / half-open
The difference between TCP Scan is that SYN scans send a **RST** TCP packet after receiving SYN/ACK packet. The rules to identify open and close ports are the same that TCP scan, if a port is closed the server respond with a **RST TCP packet**, if the port is filtered by a firewall the SYN packet is dropped.

![[Pasted image 20230514233359.png]]

#### ¿Which are the advantages?
- It can be used to bypass **older** Intrusion Detection systems that are looking for full three way handshake
- Normaly the SYN scansnot logged by applications because they save the log when the connection is fully established, but this is not the case with this type of scan
- This types of scan are more faster because they don't finish the full three way handshake

**Syn scans are used for default when you run nmap with sudo permission**, in case that you execute the program without this permission, the scan by default is TCP.

#### Disadvantages
- They require sudo permission. This is because they require the ability to create raw packets (allow to have a control of the content of the packet)
- Unastable services sometimes can get down

#### nmap switch
-

### UDP Scans
The packet that send to the target can arrive or not. This type of packet are very faster becuase they don't have response ( for nmap the port that don't have response is open|filtered ), but if the port have a UDP response (is very unusual) the port is open.
The port is close when receive a ICMP packet (ping) saying that is unreachable

This is very **slow** to realice, for this is used with ``--top-ports <number>``, `--top-ports 20` the 20 ports more used in UDP

#### nmap switch
- **-sU**

### NULL, FIN and Xmas
More information in tryhackme furthernmap
They generally used for firewall evasion

## ICMP Scanning
In the first connection, we need to do a map of the network structure. We can see what host are active or not.

With nmap we can do **ping sweep**.
We can send a ping to all host of a network range.

To do that we use **-sn** that tells to nmap that don't scan any port (except 80 and 443) and send ICMP packets

### Example
````bash
nmap -sn 192.168.0.1-254

nmap -sn 192.168.0.0/24
````

## NSE scripts
**N**map **S**cripting **E**ngine ( NSE ), they are written in Lua.

### Categories
- ``safe``: Don't affect the target
- `intrusive`: This is not safe, they affect target
- `vuln`: Scan for vulnerabilities
- ``exploit``: Attempt to exploit vulnerabilities
- `auth`: Attempt to bypass authentication for running services like ftp server anonymously
- `brute`: Attempt to bruteforce credentials in running services
- `discovery`: Attempt to query services for information about network.

### How to work with them

To run any script of a category we can use
````bash
--script=safe
````

To run a specific script we use:
````bash
--script=http-fileupload-exploiter
````

And multiple script:
````bash
--script=smb-enum-users,smb-enum-shares
````

Some script require some arguments
This example take two arguments, the location of the file and
````bash
nmap -p 80 --script http-put --script-args http-put.url='/dav/shell.php',http-put.file='./shell.php'
````

The arguements are **url** (http-put.url) and **file** (http-put.file) 


remember that all scripts have help menus, you can use:
````bash
nmap --script-help script-name
````

### How to search NSE scripts
We have two option for this, search in the [nmap website](https://nmap.org/nsedoc/) or we can search in our local storage, all the scripts are at /usr/share/nmap/scripts.

We have two option to see the installed script
One is using the /usr/share/nmap/scripts/scripts.db, we can **grep** the key words to find scripts
````bash
grep "ftp" /usr/share/nmap/script.db
````

The second form is using the ls command:
````bash
ls -l /usr/share/nmap/scripts/*ftp*
````

Remember that the asterisks  complete the rest of the characters

### How to intall new script
If we lose some scripts we can update and reinstall nmap
````bash
sudo apt update && sudo apt install nmap
````

Or we can install scripts manually one by one
````bash
sudo wget -O /usr/share/nmap/scripts/<script-name>.nse https://svn.nmap.org/nmap/scripts/<script-name>.nse
````

You can do your **own script** in LUA for nmap

Don't forget update the script.db, because nmap use this for internal engineer.
````bash
nmap --script-update-db
````

## Firewall evasion
Nmap have a param **-Pn** that tells to nmap that don't do ping to the host before scan it. This means that nmap** always treat the target host(s) as beign alive** and if the target host have a ICMP block they don't detected. The wrong part of this is that the scan **take a long time**. in the case that the host was dead the nmap was checking port and double checking the port.

==Important!==
Don't do that were you are in the local network, you can use the **ARP requests** to determiny the host activity

Some switches that are used for evation firwall:
- `-f` --> Fragment the packet,  when the packets are smaller it will be most dificult to be detected.
- `--scan-delay <time>ms` --> add a delay between packet sent.
- `--badsum` --> this generate a invalid checksum for the packets. The firewalls may potentially responds drop this packets. It is useful to detect a firewall
- `--data-length` --> To create a random length of arbitrary packet
# Codes examples

Here Im going to put code example of different type of enumeration and script

# Webgraphy
- https://tryhackme.com/room/furthernmap
- [chatGPT](https://chat.openai.com/)
