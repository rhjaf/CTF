 **Table of content:**
 - [Scanning](#scan)
 - [Listening](#listen)
 - [Sharing](#share)
 - [Webapp](#web)
 - [Cracking](#crack)
 - [payload generation](#payload)
 - [DNS](#dns)
 - [Searching](#search)
 - [Object Analysis](#file)
 - [Other](#misc)

<a id="scan"></a>
## Scanning
```bash
nmap -sC -sV -p- 10.0.2.5 -oA init-scan1
```
slow but reliable
```bash
sudo nmap -Pn -p- -A -T4 -oN scan.txt <target_ip>
```
faster but target may drop so adjust min-rate
```bash
sudo nmap -Pn -p- -A --min-rate 5000 -oN scan.txt <target_ip>
```
UDP scan
```bash
sudo nmap -Pn -sU --top-ports 500 -A -T4 -oN udp-scan.txt <target_ip>
```


<a id="listen"></a>
## Listening
connect to port
```bash
nc -nv <target-ip> <target-port>
``` 

<a id="share"></a>
## Sharing
```bash
ftp anonymous@10.10.100.44
```
list shares. `-N` means passwordless
```bash
smbclient -N -L //10.10.100.44
```



<a id="web"></a>
## Web application enumeration
Directory bruteforce
```bash 
gobuster dir -e -u http://10.0.2.5 -w /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt -x html,php -o gobuster-scan
```
```bash
dirb http://10.0.2.239 -r -o dirbuster-scan
```
Fuzzing Url
```bash 
ffuf -u http://10.0.2.5/blog-post/archives/randylogs.php?FUZZ=/etc/passwd -w /usr/share/wordlists/dirb/small.txt -fs 0
```
Automatic Vulnerability scanner
```bash 
nkito -host https://
```
Sending requests
```bash
curl -I HEAD http://mysite.com
curl -X POST http://mysite.com -H "Authorization: Bearer token" -D "{ 'data': 'something' }"
```

<a id="crack"></a>
## Crack and Bruteforce
Crack Zip
```bash
fcrackzip -D -p /user/share/wordlists/rockyou.txt user_backup.zip
```

<a id="payload"></a>
## Payloads
Payload with msfvenom
```bash 
msfvenom --list payloads
```
```bash 
msfvenom -p payloadname --list-options
```
```bash
msfvenom -p payloadname LHOST="" LPORT="" -f python -b "\x00\x0a\0d"
```

<a id="dns"></a>
## DNS 
Get all dns servers for a domain
```bash
dig +short ns zonetransfer.me
```
Get copy of zone from dns server
```bash
dig axfr zonetransfer.me @nsztm1.digi.ninja.
```
DNS zone transfer. "mysite.test" is the certificate name that target is using
```bash
host -l mysite.test 10.10.100.44
```


<a id="search"></a>
## Searching
```bash
find . -user userX -group groupY | cat 		===> show path
find . -user userX -group groupY | xargs cat	===> show content
```






<a id="file"></a>
## File and object analysis
binwalk: embedded inside exe and other file types
```bash
binwalk -D "png image:png" fidf.jpg
```
exiftool: meta information

objdump: copy important Info from a file like its assembler or execution code in assembly
```bash
objdump -D file.bin
```
```bash
strings -a
```
hexdump of a file
```bash
xxd
```
stat of a file
```bash
stat 
```

<a id="misc"></a>
## Misc
Copy to clipboard
```bash
cat my-ctf-password.txt | xclip
```
