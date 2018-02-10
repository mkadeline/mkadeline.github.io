---
layout: post
title: "Kioptrix 4 Walkthrough and Analysis"
description: A walkthrough and analysis of the methods involved in Kioptrix 4.
date: 2018-01-02
category: security
excerpt_separator: <!--more-->
image: /assets/images/cardimages/fingerprint.png
type: BlogPosting
---

Once again, we begin with a scan of the local network using ```nmap -sP -sV 10.0.0.*``` to find the VM IP is ```10.0.0.15```.

Then, a TCP port scan:<!--more-->

![nmapscan](/assets/images/postimages/security/screenshots/nmap-port-scan-kio-4.png)

which shows us four open ports. The usual OpenSSH, Web port and this time two Samba ports. I should have recognised immediately this was something to scan and investigate before spending so much time on the web interface.

Opening the IP in the browser gives us a login page. I immediately begin attempting some SQL injection. This time, I'm avoiding anything not allowed in the exam, so SQLMap and OWASP ZAP are out. I immediately tried several (incorrect) variations of ```'OR '1=1;#``` in both the username and password field. On the first few attempts, with my injection syntax wrong, I caused the program to malfunction but did not get any other output except:

![error](/assets/images/postimages/security/screenshots/kio-4-error.png)

Interested by this, I tried several other SQLi commands that should produce some output but only ever saw this error. After some googling, it seems this is a case of blind SQL injection, where the program has been set up to hide the output and deliver error messages on failed or malformed queries. Getting back to the task of bypassing the authentication, I eventually remembered the correct syntax and was authenticated. However, as I'd essentially forced my was in, I was presented with:

![error-two](/assets/images/postimages/security/screenshots/kio-4-error2.png)

GOing back to the blind SQLi tactics, and seeing the url bar was ```10.0.0.15/member.php?username=xxx```, I spent a while trying out several queries to elicit a true and false response from the database. However, as the variable was username, it seems as soon as an incorrect username was presented, the query automatically failed and delivered the same error above.

I had to change my tactics and find some usernames. In a move I should have made at the very beginning, I looked further into samba and found an nmap script to enumerate the samba users: 

![smb-enum](/assets/images/postimages/security/screenshots/nmap-smb-enum-kio-4.png)

Success - plugging those values back into the web app give me two users and two passwords. SSHing into their account gave me a *very* limited shell, with only a few commands available.

![smb-enum](/assets/images/postimages/security/screenshots/limited-shell-kio-4.png)

It took a long time and a lot of googling to find a way out of the limited shell. At first I tried to manipulate an echo command into reading and editing the .bashrc and .bash_profile files, as John is an owner, using variations of echo "$(cat my_file.txt)". However, the shell limited the syntax used and would kick me out after one strike. I saw several escape directions using commands I was unable to use such as ```wget```ting a malicious file, ssh'ing into another shell, using ```vi``` or some other text editor to edit the bash files and various uses of the ```>``` operator and finally, uses of python, perl and other scripting languages. Eventually, I came across one method that was permitted: ```echo os.system('/bin/bash')``` from https://netsec.ws/?p=337

I couldn't quite understand why this would work, especially as it doesn't work on my system. An explanation from stackexchange from user234931:

>This seems to have been an issue that existed in LShell 0.9.15, a restricted shell implemented in python.
>The vulnerable function was called ```check_path()```, which was used to check if a user was allowed to access the given path on the command line.
>The problem was that this function used ```eval()``` as a mean to strip quotes from the command line, and this would happily execute any valid python expression as well.
>```
>    for item in line:
>        # remove potential quotes
>        try:
>            item = eval(item)
>        except:
>            pass
>```
>This issue was later fixed in the following commit by replacing the eval() call with a regex substitution.

Regardless, the command worked and I was now in an unrestricted, although not root, shell. A quick check of John's permissions led to nothing I could immediately see. Not sure of where to go next, I decided to check out the MySQL database. In previous challenges I saw some MySQL passwords were stored in local files in plain text. First, I ran ```ps aux``` to confirm MySQL was running:

Before looking for a password I decided to check whether there even was one, and there wasn't. I also noticed MySQL was running as root. My initial thought was to explore the database and find different users with hopefully higher, if not close to root privileges and move up from there. However, exploring the databases showed only John and Robert:

At this stage, I actually doubled back and forgot about MySQL running as root. Attempting and googling for more Samba and Netbios-ssn exploits. Finally, after some time googling MySQL running as root, I came across an explanation of a MySQL plugin that executes commands at the same permission level as itself, in this case as root. If I can execute a /bin/bash command as root, I'll get the root shell.

https://www.adampalmer.me/iodigitalsec/2013/08/13/mysql-root-to-system-root-with-udf-for-windows-and-linux/ will take you through the full exploit.

Essentially. MySQL has the ability to run User Defined Functions which are stored in the database and called with a special syntax. The instructions that worked on my machine are as follows (note: before I completed the instruction I explored the database and found the .so files already stored, so only had to execute the function).

> ```create function sys_exec returns integer soname 'lib_mysqludf_sys.so';```
> ```select sys_exec('/bin/bash')```

That command unfortunately didn't work. Maybe because MySQL gets the shell then closes it. Either way, it doesn't come to John. Unfortunately, also simply adding john to the sudo group using ```SELECT sys_exec('sudo adduser john sudo)``` did not work. From previous Kioptrix VM's, I knew could get sudo permission by editing the /etc/sudeors file as root. So using ```SELECT sys_exec('echo "john ALL=(ALL) ALL" >> /etc/sudoers')``` was successful and I was able to log into root.

Happy days.
