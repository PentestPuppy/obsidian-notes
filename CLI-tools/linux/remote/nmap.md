ub
# Nmap: Network Mapper
```
nmap [Scan Type(s)] [Options] {target sppecification}
```
## Basics
When scanning a host with Nmap, the output looks something like this:
```shell
trshpuppy@htb[/htb]$ nmap 10.129.42.253

Starting Nmap 7.80 ( https://nmap.org ) at 2021-02-25 16:07 EST
Nmap scan report for 10.129.42.253
Host is up (0.11s latency).
Not shown: 995 closed ports
PORT    STATE SERVICE
21/tcp  open  ftp
22/tcp  open  ssh
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds

Nmap done: 1 IP address (1 host up) scanned in 2.19 seconds
```
### `STATE` column
This column tells us about the report and how it responded to Nmap's probes. The port will either be `closed`,  `open`, `filtered`, `unfiltered`, `open|filtered`, or `closed|filtered`.
#### `filtered` and `unfiltered`
`filtered` usually means Nmap *couldn't tell the state of the port* because it's behind a firewall or other network device. `unfiltered` means the port responded to Nmap but Nmap couldn't tell whether it was open or closed.
#### `open|filtered` and `closed|filtered`
Nmap reports these states when it can't determine between which of the two states the port was in. You may see this *especially with UDP scans.*
## Usage
### Host Discovery
Nmap Automatically does host discovery with each scan unless *told otherwise*. If not host discovery options are provided, nmap sends an [ICMP](networking/protocols/ICMP.md) echo request, a TCP `SYN` packet to port 443 ([HTTPS](www/HTTPS.md)), and a TCP `ACK` packet to port 80 ([HTTP](www/HTTP.md)).

These defaults *change depending on if nmap is run w/ root privileges*. Additionally, if any targets are *on a local ethernet connection*, the default host discovery includes [ARP](networking/protocols/ARP.md) and 'Neighbor Discovery' scans.
#### `-Pn` No Ping
This tells nmap to *skip host discovery*. Setting `-Pn` will tell nmap to attempt the additional flags/ scanning functions *against every IP address specified*.
#### `-PA` & `-PS` TCP ACK & TCP SYN
The `-PS` flag tells nmap to send a *TCP SYN* packet to the specified ports. When the host responds (w/ TCP ACK), the *kernel of the scanning machine* sends a TCP RST packet. If no ports are specified, nmap will scan the defaults (443 & 80). *There should be no space between `-PS` and the ports*:
```bash
nmap -PA1-1000 $t
Starting Nmap 7.94 ( https://nmap.org ) at...
```
The `-PS` flag is similar to `-PA` but instead of sending an SYN packet, nmap *sends an ACK* packet. The reason for sending different packet types *is to test for [firewalls](cybersecurity/defense/firewalls.md)*.
### `-sT` TCP scan:
TCP connect scan scans for [TCP](/networking/protocols/TCP.md) ports and performs the *entire three way handshake* with the target system. nmap reports the port status depending on the flags it exchanges with the target.
#### Results:
If nmap receives an `ACK` flag, it reports the port as _open_. If it receives the `RST` flag, it reports the port as *closed*.

If there is no response, that likely means *the port is hidden behind a firewall.* This is because older [firewalls](/cybersecurity/defense/firewalls.md) will drop the packet. *Some firewalls are configured to send a fabricated `RST` flag* which nmap will mark as "closed."
### `-sS` (half/open or "stealth")
SYN connect scan/ "half-open" or "stealth" scan. When the target sends the `SYN/ACK` flag, nmap (the client) then sends the `RST` flag which closes the connection. This prevents the target from repeatedly sending requests.
#### Advantages
- This is *the default scan* run by nmap when executed with `sudo`. It's considered "stealthy" because most *older* IDS systems only log a connection if it was fully-established (i.e. the entire three-way handshake was performed).
- This scan is also *faster than the TCP*.
#### Disadvantages
- May not work with *newer IDS's* which track/log all connection requests whether or not they completed
- May cause unstable target OSes to *crash*
- Requires *sudo privileges*
### `-sV` (versioning)
When **NOT** given this flag, nmap will scan a target and list the services which are *likely* to be hosted at specific ports. For example, most systems will host their [SMTP](/networking/protocols/SMTP.md) [email](/networking/email.md) service on port 25. If nmap finds that port is open, it will report that the service running on the target is SMTP on port 25.

Nmap uses their nmap-services database of about 2200 known, common ports and their services to do this. *However* any port can theoretically host *any service*.

To more accurately determine the service hosted on a target port, the `-sV` flag can be given. This enables version detection.
#### Version detection mode:
When `-sV` is given and a scan is performed in version detection mode, nmap will attempt to determine the service by *querying each open port* using probes from their database.

*If nmap cannot match port responses* to their database, it will print a fingerprint to the command line which you can submit to nmap (via a URL printed with the fingerprint).
```bash
nmap -sV -sT -Pn 10.0.2.4                                        
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-20 13:24 EDT
Nmap scan report for 10.0.2.4
Host is up (0.0052s latency).
Not shown: 995 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
135/tcp  open  msrpc         Microsoft Windows RPC
445/tcp  open  microsoft-ds?
1042/tcp open  afrog?
1043/tcp open  ssl/boinc?
2222/tcp open  ssh           OpenSSH 9.2p1 Debian 2 (protocol 2.0)
2 services unrecognized despite returning data. If you know the service/version, please submit the following fingerprints at https://nmap.org/cgi-bin/submit.cgi?new-service :
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1042-TCP:V=7.94%I=7%D=7/20%Time=64B96DEA%P=x86_64-pc-linux-gnu%r(Ge
SF:tRequest,27E,"HTTP/1\.1\x20404\x20Not\x20Found\r\nVary:\x20Origin\r\nCo
SF:ntent-Security-Policy:\x20default-src\x20'self'\r\nX-DNS-Prefetch-Contr
SF:ol:\x20off\r\nExpect-CT:\x20max-age=0\r\nX-Frame-Options:\x20SAMEORIGIN
SF:\r\nStrict-Transport-Security:\x20max-age=15552000;\x20includeSubDomain
SF:s\r\nX-Download-Options:\x20noopen\r\nX-Content-Type-Options:\x20nosnif
SF:f\r\nX-Permitted-Cross-Domain-Policies:\x20none\r\nReferrer-Policy:\x20
SF:no-referrer\r\nX-XSS-Protection:\x200\r\nContent-Type:\x20text/html;\x2
SF:0charset=utf-8\r\nContent-Length:\x20139\r\nDate:\x20Thu,\x2020\x20Jul\
SF:x202023\x2017:25:01\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTYPE\x20
SF:html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"utf-8\">\n<tit
SF:le>Error</title>\n</head>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\n</bod
SF:y>\n</html>\n")%r(HTTPOptions,D2,"HTTP/1\.1\x20204\x20No\x20Content\r\n
SF:Vary:\x20Origin,\x20Access-Control-Request-Headers\r\nAccess-Control-Al
SF:low-Methods:\x20GET,HEAD,PUT,PATCH,POST,DELETE\r\nContent-Length:\x200\
SF:r\nDate:\x20Thu,\x2020\x20Jul\x202023\x2017:25:01\x20GMT\r\nConnection:
SF:\x20close\r\n\r\n")%r(RTSPRequest,2F,"HTTP/1\.1\x20400\x20Bad\x20Reques
SF:t\r\nConnection:\x20close\r\n\r\n")%r(RPCCheck,2F,"HTTP/1\.1\x20400\x20
SF:Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(DNSVersionBindReqTCP
SF:,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n
SF:")%r(DNSStatusRequestTCP,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConn
SF:ection:\x20close\r\n\r\n")%r(Help,2F,"HTTP/1\.1\x20400\x20Bad\x20Reques
SF:t\r\nConnection:\x20close\r\n\r\n")%r(SSLSessionReq,2F,"HTTP/1\.1\x2040
SF:0\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(TerminalServerC
SF:ookie,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\
SF:n\r\n")%r(TLSSessionReq,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConne
SF:ction:\x20close\r\n\r\n")%r(Kerberos,2F,"HTTP/1\.1\x20400\x20Bad\x20Req
SF:uest\r\nConnection:\x20close\r\n\r\n")%r(SMBProgNeg,2F,"HTTP/1\.1\x2040
SF:0\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(X11Probe,2F,"HT
SF:TP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n");
==============NEXT SERVICE FINGERPRINT (SUBMIT INDIVIDUALLY)==============
SF-Port1043-TCP:V=7.94%T=SSL%I=7%D=7/20%Time=64B96DF5%P=x86_64-pc-linux-gn
SF:u%r(GetRequest,27E,"HTTP/1\.1\x20404\x20Not\x20Found\r\nVary:\x20Origin
SF:\r\nContent-Security-Policy:\x20default-src\x20'self'\r\nX-DNS-Prefetch
SF:-Control:\x20off\r\nExpect-CT:\x20max-age=0\r\nX-Frame-Options:\x20SAME
SF:ORIGIN\r\nStrict-Transport-Security:\x20max-age=15552000;\x20includeSub
SF:Domains\r\nX-Download-Options:\x20noopen\r\nX-Content-Type-Options:\x20
SF:nosniff\r\nX-Permitted-Cross-Domain-Policies:\x20none\r\nReferrer-Polic
SF:y:\x20no-referrer\r\nX-XSS-Protection:\x200\r\nContent-Type:\x20text/ht
SF:ml;\x20charset=utf-8\r\nContent-Length:\x20139\r\nDate:\x20Thu,\x2020\x
SF:20Jul\x202023\x2017:25:12\x20GMT\r\nConnection:\x20close\r\n\r\n<!DOCTY
SF:PE\x20html>\n<html\x20lang=\"en\">\n<head>\n<meta\x20charset=\"utf-8\">
SF:\n<title>Error</title>\n</head>\n<body>\n<pre>Cannot\x20GET\x20/</pre>\
SF:n</body>\n</html>\n")%r(HTTPOptions,D2,"HTTP/1\.1\x20204\x20No\x20Conte
SF:nt\r\nVary:\x20Origin,\x20Access-Control-Request-Headers\r\nAccess-Cont
SF:rol-Allow-Methods:\x20GET,HEAD,PUT,PATCH,POST,DELETE\r\nContent-Length:
SF:\x200\r\nDate:\x20Thu,\x2020\x20Jul\x202023\x2017:25:12\x20GMT\r\nConne
SF:ction:\x20close\r\n\r\n")%r(TerminalServerCookie,2F,"HTTP/1\.1\x20400\x
SF:20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(TLSSessionReq,2F,"
SF:HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close\r\n\r\n")%r(
SF:Kerberos,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConnection:\x20close
SF:\r\n\r\n")%r(SMBProgNeg,2F,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nConne
SF:ction:\x20close\r\n\r\n")%r(FourOhFourRequest,2E6,"HTTP/1\.1\x20500\x20
SF:Internal\x20Server\x20Error\r\nVary:\x20Origin\r\nContent-Security-Poli
SF:cy:\x20default-src\x20'self'\x20blob:\x20127\.0\.0\.1:1042\x20127\.0\.0
SF:\.1:1043;script-src\x20'self'\x20'unsafe-inline';style-src\x20'self';co
SF:nnect-src\x20'self'\x20ws:\x20wss:\x20blob:\x20127\.0\.0\.1:1042\x20127
SF:\.0\.0\.1:1043\x20127\.0\.0\.1:9013\x20127\.0\.0\.1:9014;worker-src\x20
SF:'self'\x20blob:\x20127\.0\.0\.1:1042\x20127\.0\.0\.1:1043;img-src\x20's
SF:elf'\x20data:\x20blob:\x20127\.0\.0\.1:1042\x20127\.0\.0\.1:1043\r\nX-D
SF:NS-Prefetch-Control:\x20off\r\nExpect-CT:\x20max-age=0\r\nX-Frame-Optio
SF:ns:\x20SAMEORIGIN\r\nStrict-Transport-Security:\x20max-age=15552000;\x2
SF:0includeSubDomains\r\nX-Download-Options:\x20noopen\r\nX-Content-Type-O
SF:ptions:\x20nosniff\r\nX-Permitted-Cross-Domain-Policies:\x20none\r\nRef
SF:errer-Policy:\x20no-referrer\r\nX-XSS-Protection:\x200\r\nDate:\x20Thu,
SF:\x2020\x20Jul\x202023\x2017:25:12\x20GMT\r\nConnection:\x20close\r\n\r\
SF:n");
Service Info: OSs: Windows, Linux; CPE: cpe:/o:microsoft:windows, cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 28.42 seconds
```
#### Versioning Options:
##### `--all-ports`:
Every port scanned will be submitted to version detection (querying for a response from the service).
##### `--version-intensity <intensity>`:
The probes sent by nmap for versioning are assigned a rarity value b/w 1 and 9. Lower numbers *have had the most success against targets in the past*. Higher numbers are more accurate but take longer.

The default is set to 7. 
##### `--version-all`
Try *every single probe*.
### `-O` OS Detection:
This flag enables nmap to do *OS detection* on the target. It requires *sudo* privileges.

Nmap does OS detection by using [TCP/IP](/networking/protocols/TCP.md) stack fingerprinting. Basically, it sends TCP and [UDP](/networking/protocols/UDP.md) packets to the remote target and examines every single bit in the responses.

After performing a number of tests on the responses (including TCP ISN sampling, checking the window size, IP ID sampling, etc) it will compare the results to nmap's *OS database* of >2600 known OS fingerprints.
#### Result reliability:
If the results of the scan include *both an open and closed port* than the OS detection results can be considered "good". However, if not, then nmap reports the results as "unreliable".

When nmap can't detect a perfect match, then it will perform *aggressive OS detection* and offer up near-possibilities w/ % of likelihood:
```bash
... 
2222/tcp open  EtherNetIP-1
MAC Address: 52:54:00:12:35:04 (QEMU virtual NIC)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|bridge|switch|printer
Running (JUST GUESSING): QEMU (96%), Oracle Virtualbox (95%), Bay Networks embedded (88%), Samsung embedded (87%), Dell embedded (87%), Wind River VxWorks (87%), Xerox embedded (86%)
OS CPE: cpe:/a:qemu:qemu cpe:/o:oracle:virtualbox cpe:/h:baynetworks:baystack_450 cpe:/h:samsung:clp-315w cpe:/h:dell:1815dn cpe:/o:windriver:vxworks cpe:/h:xerox:workcentre_4150
Aggressive OS guesses: QEMU user mode network gateway (96%), Oracle Virtualbox (95%), Bay Networks BayStack 450 switch (software version 3.1.0.22) (88%), Samsung CLP-315W printer (87%), Dell 1815dn printer (87%), VxWorks (87%), Xerox WorkCentre 4150 printer (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop
```

If you don't want nmap to do OS detection unless it will be reliable (at least one open and one closed port), you can add the `--osscan-limit`. With this flag, OS detection will not be performed unless there is at least one open and one closed port.
#### Other useful info:
The `-O` scan also returns info that is useful about the target beyond the OS including:
- MAC address
- Device type
- CPE (Common Platform Enumeration)
- Network distance (how many hops)
### `-sU` UDP/ "stateless":
Since UDP doesn't require a handshake, it's considered "stateless". When there is no response to a packet sent by nmap to the port, it is more difficult for nmap to determine the state of the port.

Scanning for ports using the UDP protocol *takes longer* which means that *some security auditors will ignore UDP ports*. These include common, important which host services like [DNS](/networking/DNS/DNS.md), [SNMP](/networking/protocols/SNMP.md), [DHCP](/networking/protocols/DHCP.md), etc..

Nmap attempts to detect UDP ports by sending a UDP packet *without a payload* to every target port. It will report the results based on the response:

| Probe Response                                                  | Assigned State   |
| --------------------------------------------------------------- | ---------------- |
| Any UDP response from target port (unusual)                     | `open`           |
| No response received (even after retransmissions)               | `open\|filtered` |
| ICMP port unreachable error (type 3, code 3)                    | `closed`         |
| Other ICMP unreachable errors (type 3, code 1, 2, 9, 10, or 13) | `filtered`       |
>	[Nmap.org](https://nmap.org/book/scan-methods-udp-scan.html)
#### Speeding up UDP scans
```bash
sudo nmap -sUV -T4 -F --version-intensity 0 scanme.nmap.org
Starting Nmap 7.94 ( https://nmap.org ) at 2023-07-20 15:13 EDT
Nmap scan report for scanme.nmap.org (45.33.32.156)
Host is up (0.012s latency).
Other addresses for scanme.nmap.org (not scanned): 2600:3c01::f03c:91ff:fe18:bb2f
Not shown: 99 open|filtered udp ports (no-response)
PORT    STATE SERVICE VERSION
123/udp open  ntp     NTP v4 (secondary server)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.89 seconds
```
>	command from [Nmap.org](https://nmap.org/book/scan-methods-udp-scan.html)

- Set version detection to `0` if doing version scanning
- If possible *scan after bypassing the firewall* (from behind the firewall)
- Use `--host-timeout` to skip 'slow' hosts (some are ICMP rate limited) (ex: `--host-timeout 5m` = 5 minutes)
- Use `-v` (verbosity): this flag will tell you real time how long nmap thinks the scan will take
- Limit the number of ports scanned: `-p <number or range>`
### TCP FIN, NULL, & Xmas Scans
These scans send *malformed packets* which allows them to be stealthier in some contexts. This is because *most* firewalls automatically block incoming requests w/ the `SYN` flag but *don't block packets w/o the `SYN` flag*.

According to [Nmap.org](https://nmap.org/book/scan-methods-null-fin-xmas-scan.html):
>	"Page 65 of RFC 793 says that “if the [destination] port state is CLOSED .... an incoming segment not containing a RST causes a RST to be sent in response.” Then the next page discusses packets sent to open ports without the SYN, RST, or ACK bits set, stating that: “you are unlikely to get here, but if you do, drop the segment, and return.”

Closed ports will respond to all of these scans with an `RST` flag:

| Probe Response                                              | Assigned State   |
| ----------------------------------------------------------- | ---------------- |
| No response received (even after retransmissions)           | `open\|filtered` |
| TCP RST packet                                              | `closed`         |
| ICMP unreachable error (type 3, code 1, 2, 3, 9, 10, or 13) | `filtered`       |
> 	Nmap.org
#### `-sN` NULL scan
This scan sends a TCP packet *without any flags set* so the packet is empty (TCP header is 0).
#### `-sF` TCP Fin
The only bit set in the packet is the `TCP FIN` bit.
#### `-sX` Xmas scan
Sets the FIN, PSH, and URG flags which "lights the packet up like a xmas tree"
### Nmap Scripting Engine 
Nmap comes with the [Nmap Scripting Engine](https://nmap.org/book/nse.html) (NSE) which is a collection of user-written [Lua](/coding/languages/lua.md) scripts used to automate a wide-variety of networking tasks.
```bash
sudo nmap -sSUC -p 111 10.0.3.5
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-08 18:36 EDT
Nmap scan report for 10.0.3.5
Host is up (0.00042s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1          32768/tcp   status
|_  100024  1          32768/udp   status
111/udp open  rpcbind
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1          32768/tcp   status
|_  100024  1          32768/udp   status
MAC Address: x:x:x:x:x:x (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.32 seconds
```
In this example, the `-sS` (TCP Syn scan), `-sU` (UDP), and `-sC` (scripting) flags are given with a target port of `111` (usually [RPC](/networking/protocols/RPC.md)). With these flags, nmap is able to return *extra* information as it relates to `port 111` on the target.

Nmap ran default script/s written specifically for port 111. In this case, the script was able to enumerate specific programs, including their numbers, version, and service.

The same scan, on the same target *without the `-sC` flag* returns the following, less-detailed report:
```bash
sudo nmap -sSU -p 111 10.0.3.5                                                   
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-08 19:01 EDT
Nmap scan report for 10.0.3.5
Host is up (0.00040s latency).

PORT    STATE SERVICE
111/tcp open  rpcbind
111/udp open  rpcbind
MAC Address: x:x:x:x:x:x (Oracle VirtualBox virtual NIC)

Nmap done: 1 IP address (1 host up) scanned in 0.40 seconds
```
#### `-sC` Scripting
To use the NSE, the `-sC` flag is provided. You can either use it without any arguments (and default scripts will be ran based on the rest of the `nmap` command), or you can give `-sC` the path to a custom script you want run.
#### Looking up nmap scripts
```bash
ls /usr/share/nmap/scripts/ | grep <service> | grep <vuln, etc>
```
You can also use the `locate` command like so:
```bash
$ locate scripts/citrix

/usr/share/nmap/scripts/citrix-brute-xml.nse
/usr/share/nmap/scripts/citrix-enum-apps-xml.nse
/usr/share/nmap/scripts/citrix-enum-apps.nse
/usr/share/nmap/scripts/citrix-enum-servers-xml.nse
/usr/share/nmap/scripts/citrix-enum-servers.nse
```
#### `--script smb-vuln`
Used to look for all smb vulnerabilities against a host
#### Updating the NSE
To add new scripts to the NSE database, we can use the `--script-updatedb` flag. For example, if we find a script for the vulnerability `2021-41773` as a POC on someone's. GitHub, we can download the `.nse` file, copy it to nmap's script directory, and then use the `--script-updatedb` command:
```bash
kali@kali:~$ sudo cp /home/kali/Downloads/http-vuln-cve-2021-41773.nse /usr/share/nmap/scripts/http-vuln-cve2021-41773.nse

kali@kali:~$ sudo nmap --script-updatedb
[sudo] password for kali: 
Starting Nmap 7.92 ( https://nmap.org )
NSE: Updating rule database.
NSE: Script Database updated successfully.
Nmap done: 0 IP addresses (0 hosts up) scanned in 0.54 seconds
```
### Vuln Scanning
![Vulners script](../../../OSCP/Vulnerability%20Scanning/nmap-vuln-scanning.md#Vulners%20script)
[Vulnerability Scanning w/ Nmap](../../../OSCP/Vulnerability%20Scanning/nmap-vuln-scanning.md#Vulnerability%20Scanning%20w/%20Nmap)
## Examples which worked
```bash
nmap -sS -sC -sV -Pn -n -p- -vvv --open --min-hostgroup 256 --min-rate 1000 --max-rtt-timeout 300ms --max-retries 2 --script targets-xml --script -args newtargets,iX=nmap/livehosts-fulltcp.xml -oA nmap/livehosts-allports-scripts
```
### How to actually run scripts you dingus
Adding `+<script-title>` will force the script to run whether it's specific conditions are met or not.
```bash
┌─[25-03-17 17:32:33]:(root@192.168.144.131)-[/home/trshpuppy/oscp/recon]
└─# nmap 192.168.210.149 --script +http-title -p53,88,389,445,464,593,636,5985,9389,47001,49664,49665,49666,49667,49671,49674,49675,49681,49687,49709,49855
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-03-17 17:33 EDT
Nmap scan report for 192.168.210.149
Host is up (0.11s latency).

PORT      STATE SERVICE
53/tcp    open  domain
88/tcp    open  kerberos-sec
389/tcp   open  ldap
445/tcp   open  microsoft-ds
464/tcp   open  kpasswd5
593/tcp   open  http-rpc-epmap
636/tcp   open  ldapssl
5985/tcp  open  wsman
|_http-title: Not Found
9389/tcp  open  adws
47001/tcp open  winrm
|_http-title: Not Found
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49671/tcp open  unknown
49674/tcp open  unknown
49675/tcp open  unknown
49681/tcp open  unknown
49687/tcp open  unknown
49709/tcp open  unknown
49855/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 8.81 seconds

```
## Other TidBits
### Banner Grabbing
You can use Nmap to grab the banner of a port which you want to investigate further:
```bash
# Single Host
nmap -sV --script=banner 10.10.10.10 2121

# CIDR range
nmap -sV --script=banner 10.10.10.0/24 
```
### Probe Timeout
> **--min-rtt-timeout** time, **--max-rtt-timeout** time, **--initial-rtt-timeout** time (Adjust probe timeouts).  
>
> Nmap maintains a running timeout value for determining how long it will wait for a probe response before giving up or retransmitting the probe. This is calculated based on the response times of previous probes.
>
> If the network latency shows itself to be significant and variable, this timeout can grow to several seconds. It also starts at a conservative (high) level and may stay that way for a while when Nmap scans unresponsive hosts.
>
> Specifying a lower **--max-rtt-timeout** and **--initial-rtt-timeout** than the defaults can cut scan times significantly. This is particularly true for pingless (**-PN**) scans, and those against heavily filtered networks. Don´t get too aggressive though. The scan can end up taking longer if you specify such a low value that many probes are timing out and retransmitting while the response is in transit.  If all the hosts are on a local network, 100 milliseconds is a reasonable aggressive
>
>  **--max-rtt-timeout** value. If routing is involved, ping a host on the network first with the ICMP ping utility, or with a custom packet crafter such as **hping2**.  that is more likely to get through a firewall. Look at the maximum round trip time out of ten packets or so. You might want to double that for the **--initial-rtt-timeout** and triple or quadruple it for the **--max-rtt-timeout**. I generally do not set the maximum RTT below 100 ms, no matter what the ping times are. Nor do I exceed 1000 ms.
>
> **--min-rtt-timeout** is a rarely used option that could be useful when a network is so unreliable that   even Nmap´s default is too aggressive. Since Nmap only reduces the timeout down to the minimum when the network seems to be reliable, this need is unusual and should be reported as a bug to the nmap-dev mailing list..
-`man nmap`

> [!Resources]
> - `man nmap`
> - [Nmap: Version Detection](https://nmap.org/book/man-version-detection.html)
> - [Nmap: OS Detection](https://nmap.org/book/man-os-detection.html)
> - [Nmap: UDP scanning](https://nmap.org/book/scan-methods-udp-scan.html)
> - [Nmap: NULL, FIN, XMAS](https://nmap.org/book/scan-methods-null-fin-xmas-scan.html)
> - [Nmap: Nmap Scripting Engine](https://nmap.org/book/nse.html)
> - [Nmap: Host Discovery](https://nmap.org/book/man-host-discovery.html)

> [!Related]
> - [Port Scanning with Nmap](../../../OSCP/Enumeration%20&%20Info%20Gathering/active/nmap-scanning.md#Port%20Scanning%20with%20Nmap)
> - [Nmap scanning:](../../../PNPT/PEH/scanning-enumeration/scanning-with-nmap.md#nmap%20scanning)
> - [Vulnerability Scanning w/ Nmap](../../../OSCP/Vulnerability%20Scanning/nmap-vuln-scanning.md#Vulnerability%20Scanning%20w/%20Nmap)

