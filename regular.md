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
**Advanced:**
 - [Android](#android)
 - [Web](#web)
 - [Reverse Engineering and Binary Analysis](#binary)


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
Search strings inside files
```bash
grep -rli './' -e 'search_string'
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



<a id="android"></a>
# Android
- Application-based permissions
- `Broadcast Receiver`, `Activity`, `Content Provider`, `Intent`, `Service`: background process
- `APK` installation in android architectures: `AndroidRunTime`, `Delvik`
- `Frida` (Server -client) **dynamic** android analysis. use it for calling functions that you want anywhere

  frida server:**x86.64** vs **ARM**

  frida client: pip

  Frida is like a Hook that can bypass: **SSL pinning**, **root detection**, **encryption**, **emulation detection**, ....

  **WARN**: Android 8 or 7(not so much newer). The device should be rooted.
  
  Frida allows us to inject JS code into processes, where your js gets executed with full access to memory, hooking functions and even calling native functions inside the process. There is a bi-directional communication channel that is used to talk between your app and the JS running inside the target process.

  - usefull commands:
    `frida-ps -U`: Zygot process and others...

    ```bash
    adb push frida data/local/temp
    ```
    ```bash
    adb shell
    ```
   Show all system logs
   ```bash
   logcat | grep 1171
   ```

  Frida allows to run JS codes inside native apps(Windows,mac,lin,android). and we can also call C functions. It is like a debugger

  In static analysis, if you need to change something, you should change it in `smali` file and then compile it. but in Frida, it is much easier and no need to recompile and just use js scripts.

  We can use it to identify informations that may be hidden inside memory that they are not easy to identify when the developer use `Obfusction`. We can use `frida` to run automate tasks: Brute forcing, fuzzing.

  It is better to run Frida on Jailbreaked (rooted) devices but it can be run `unrooted` devices using Frida `gadets`(refer to Github page)

  How to reverse using Frida?
   - identify problems and the goal and how Frida will help us
	  - decompile app using `JadX` or ...
	  - find and analyze methods that we gonna modify and hook
	  - manipulate app to desired behaviour using **JS**

  `Objection` is a tool for bypassing **SSL pinning** that is based on Frida.

  Running frida:
	  ```bash
  frida -U -l .\Desktop\frida.js -f jakhar.aseem.diva
   ```
   ```
   %resume
  ```

- `APKTools`(APK easytool APKtool GUI) and `JadxGUI` for **static** analysis (decompilation any app written in any language to **JAVA**).

  You need Java for `Jadx`

  If `JadX` can not convert to Java, use `APKtools` to convert to `Smali`.

  In some APKs, we may have more than one `classes.dex`.

  You can not protect your APK from decompilation.
  

- Java and Smali + (Js for Frida for writing scripts)
- We can use `strings` command on APK file.
- **Hardcode** means the things that can not be obfuscated. (eg: keys, urls, special strings,...)
- shared preferences, each application has this folder and each application has an space in `/data/data`.
  application's data may be stored in `/sdcard`
  ```bash
  sqllite3 db.db
  ```
- Intents
 - Create and Start directly an `intent` Action: `#am start -a jakhar.aseem.diva.action.VIEW_CREDS`
 - Intent with extra:
 	```bash
  am start -n jakhar.aseem.diva/.APICreds2Activity -a jakhar.aseem.diva.action.VIEW_CREDS2 --ez check_pin false
  ```










