---
layout: post
title: "Sample Cyber Security Interview Questions and Answers"
description: "Answers may be incorrect"
date: 2018-03-05
category: security
excerpt_separator: <!--more-->
image: /assets/images/cardimages/fingerprint.png
type: BlogPosting
---

This is a collection of technical interview questions collected from around the web, with **potentially innacurate answers**. <!--more-->

- Does TLS use symmetric or asymmetric encryption?

The answer in modern systems is both. The original connection is set up securely using asymmetric encryption, then a shared secret is used for the rest of the conversation. The reason for this is simply performance. Asymmetric key encrytion carries a performance burden, as exposing your public key to the world means your related private key must still be difficult to deduce. As factoring algorithms and computing power has increased, this has meant we need *huge* numbers to ensure our private keys are safe.
I found another explaination of the added size of asymmetrically encrypted messages on stackexchange:

>Another efficiency issue with asymmetric encryption is network bandwidth. This one is an absolute limitation. Public key encryption is public: anybody, including the attacker, can use the public key to encrypt arbitrary messages. This means that if the encryption is deterministic, then the attacker can run an exhaustive search on the encrypted data (i.e. encrypting potential messages until one matches). The data which is encrypted is still "useful data": it has structure, it is subject to such a search. Therefore, an asymmetric encryption scheme must include extra randomness. This, in turn, necessarily implies a data size increase. For instance, with RSA as described by PKCS#1, with a 1024-bit key, you can encrypt a data element only up to 117 bytes, yielding a 128-byte value. Therefore, if you were to encrypt a big file with "only RSA", you would end up with an encrypted message about 9% bigger than the plaintext: that's 900 extra megabytes for a 10 GB message. On the other hand, symmetric encryption only incurs a constant size overhead (say, at most +32 bytes for AES-CBC, including room for the initial value). There are many contexts where network bandwidth is a scarcer resource than CPU.

- Describe the process of a TLS session being set up when someone visits a secure website.

A secure website over HTTPS will usually run immediately over port 443 and thus the TCP and TLS connections will be done immediately in sequence. This means, after the initial TCP connection is complete and the client sends the SYN-ACK, the client will send a ```Hello``` to initiate the TLS connection, before any data is sent. The client sends with that hello, the TLS version it is using, a list of appropriate ciphersuites and any other TLS options such as compression. The server then responds with the TLS verison, selected ciphersuites and the certificate. Should the client accept hte certficiate, it begins the RSA key exchange or a Diffie-Hellman exchange to establish the shared key that is used for the remainder of the session.

- If someone steals the server’s private key can they decrypt all previous content sent to that server?
Potentially, depending on how the keys are generated. With Ephemeral Diffie Hellman, or EDH, the shared secret is changed and re-generated for each message. 
An example of the mechanics of the communcation are detailed on the wikipedia page for forward secrecy:
> The following is a hypothetical example of a simple instant messaging protocol that employs forward secrecy:
> 1. Alice and Bob each generate a pair of long-term, asymmetric public and private keys, then verify public-key fingerprints in person or over an already-authenticated channel. The only thing these keys will be used for is authentication, including signing messages and signing things during session key exchange. These keys will not be used for encryption of any kind.
> 2. Alice and Bob use a key exchange algorithm such as Diffie–Hellman, to securely agree on an ephemeral session key. They use the keys from step 1 only to authenticate one another during this process.
> 3. Alice sends Bob a message, encrypting it with a symmetric cipher using the session key negotiated in step 2.
> 4. Bob decrypts Alice's message using the key negotiated in step 2.
> 5. The process repeats for each new message sent, starting from step 2 (and switching Alice and Bob's roles as sender/receiver as appropriate). Step 1 is never repeated.

- What are some common ways that TLS is attack, and/or what are some ways it’s been attacked in the past?
Examples of SSL/TLS attacks:
- FREAK was an attack that involved forcing connections to downgrade to previous versions of TLS which used weaker 512 bit RSA keys that are easily broken with todays computing power. The current recommended key size for asymmetric keys is 2048 bits until 2030 and 2072 bits thereafter.
- Logjam likewise forces the downgrade to attack weak 512bit DH groups. The attacked then uses the groups to determine the shared key and decryp the traffic.
- DROWN attack was an attack on the servers that used up to date TLS but still offered SSL v2. If the same public keys were used on both connections, the server was susceptible to a chosen-ciphertext attack.
- Heartbleed 

 - What is Forward Secrecy?
Forward secrecy is the idea that is a current key is compromised, previous encrypted messages would be secure. It is achieved through the regenertion of keys with each message or at least limited reuse.

- Are open-source projects more or less secure than proprietary ones?
There are pros and cons of each. For example, the risk of having future bugs exposed and clear to see, with the benefit of many eyes looking at the problem and attempting solutions. The security will likely depend on who's looking and the skill of the developers/hackers that have access and the inclination to access the code.

- What’s the difference between encoding, encryption, and hashing?
Encoding is designed to protect data integrity during communication. It provides some obscurity to humans only and is, by design, completely transalatable by computers. Encryption is a reversible function used to protect the privacy of data and designed to be reversible only with the correct key/s. Hashing is designed to be a one way function and is done usually to authenticate identity or integrity of the user, file or data transferred.

- What’s more secure, SSL or HTTPS?
HTTPS is HTTP over SSL/TLS.

- Can you describe rainbow tables?
A rainbow table is a pre-computed table of hashes that is used to speed up hash cracking, usually for passwords or credit cards. 

- What is salting, and why is it used?
Salting is the act of adding randow strings of characters to the end of data that is then hashed. The practice makes the use of a rainbow table less effective and reduces the risks posed by insecure passwords.

- Who do you look up to within the field of Information Security? Why?
Hmmm

- Where do you get your security news from?
Potential sources include:
    - Reddit infosec pages
    - Infosec professionals on twitter
    - Dark Reading
    - Krebs on security

- If you had to both encrypt and compress data during transmission, which would you do first, and why?
Compress then encrypt. Encryption will turn data into random bytes making compression difficult. Compression will not affect encryption.

- What’s the difference between symmetric and public-key cryptography
Symmetric key uses a single shared key to encrypt and decrypt data, while public-key cryptography uses two keys. One to encrypt and one to decrypt. The recipient's public key is used to encrypt the message, which can then only be decrypted by their private key. 

- In public-key cryptography you have a public and a private key, and you often perform both encryption and signing functions. Which key is used for which function?
You must encrypt with their public key, and therefore can only be decrypted by their private key. You sign with your own private key, as no one else should be able to sign data with your private key, and thus the data is authenticated as yours.

- What kind of network do you have at home?
My particular network is a main computer triple booted with windows 10, kali and ubuntu and a second victim laptop running windows 10 for wireless attack practice, in addition to several bootable usbs and VMs for further practice. In addition, several networking devices and routers, as well as wifi adapters for different network attacks.
I have been told this question is a good indicator of whether you truly enjoy security, an incredibly important point in hiring given how fast the industry moves.

- What are the advantages offered by bug bounty programs over normal testing practices?
There is an ethical focus on high value bugs, as hunters will seek larger rewards. The added incentives will alsos attract a large number of hackers ethically targeting your system.

- What are your first three steps when securing a Linux server?
Ports, Permissions, Packages.
1. What ports are open and what services are attached to those. Any open ports extend the attack surface so extraneous ports should be closed. In addition, if ports must remain open their listening processes should be hardened and the firewalls present to protect them.
2. What user and group permissions are present. Improper user configuration is an easy way to escalate privileges or move laterally through a network. By tightening up privileges, moving critical services, file and directory access to root only and limiting permissions of other users to only what is absolutely necessary, you limit the avenues for escalation or post-exploitation. 
3. Update the kernel and OS to apply lateset security patches. Assess the currently installed packages, particularly those that commuicate over the network or WAN and remove any packages that aren't required for operation.

What are your first three steps when securing a Windows server?
This list is largely the same as the above, with more Windows specific actions.

Who’s more dangerous to an organization, insiders or outsiders?
Good risk assessment asks what is at risk. Outisiders attacks may be more common but insider attacks more damaging. So it depends which is more dangerous to the company in question.

Why is DNS monitoring important?
Malware may communicate with or send data over the network, take commands from the network or other actions that require the use of a URL or FQDN, which would be tracked and logged by the DNS monitoring. 

- What port does ping work over
Ping is an ICMP function that operates over layer 3 and therefore doesn't use a port.

- Do you prefer filtered ports or closed ports on your firewall?
There are benefits and drawbacks for both. If the ports are filtered for illegitimate traffic rather than closed, it is clear the the attacker the ports are open and simply protected by a firewall. The attacker can then attempt to tunnel through or evade the firewall. However, by dropping the packets, you also add protection against DDoS attacks, as attacking computer must timeout each request before sending another. If the immediate response is closed, the attack can resend packets and block the port if they know it is infact open and simply being filtered by a firewall.

- How exactly does traceroute/tracert work at the protocol level?
Windows uses ICMP packets and Linux uses UDP datagrams, however can be set to use any transport protocol and ICMP too. The packets are sent out with an increasing TTL or hop limit to routers attached to the host. The receiving router decrements the TTL, forwards any packets with a new TTL of 1 or greater and drops any packets with a TTL == 0 and returns an ICMP Time exceeded reply. If the packet eventually reaches the destination, the host will receive an ECMP Echo Reply. The originating host will receive all replies and eventually build a picture of the route taken by the packets.

- What are Linux’s strengths and weaknesses vs. Windows?
Windows has some excellent proprietary software. Linux can be run very lean and therefore potentially reduce attack vectors. Many skilled programmers may make linux more secure being open source. 

- Cryptographically speaking, what is the main method of building a shared secret over a public medium?
Diffie-Hellman is the preferred method although a shared secret can be built using RSA by first establishing the asymmetric encryption connection then encrypting a key to be shared using the recipients public key.

- What’s the difference between Diffie-Hellman and RSA?
Diffie-Hellman is a key-exchange protocol. The two parties publically share two parameters, ```p``` and ```g```. Then each party chooses a secret value, say ```a``` and ```b``` and releases the result of ```g^a mod p``` and ```g^b mod p``` publically. Each party takes the result their counterpart released and raises it to the power of their secret number mod p, i.e. ```(g^a mod p)^b mod p and vice versa. Both parties with have the same result as:
>(g<sup>b</sup> mod p)<sup>a</sup> mod p = g<sup>ba</sup> mod p
>(g<sup>a</sup> mod p)<sup>b</sup> mod p = g<sup>ab</sup> mod p
Thus the shared key is generated together without compromising security. Using sufficiently large numbers makes deriving the secrets, a and b, infeasible with current computing power.
RSA however uses a public-private key pair to encrypt and sign messages. Messages are signed with your own private key, meaning they are authenticated as yours and encrypted using the recipient public key, meaning they can only be decrypted using their private key. Both are called public-key cryptography as one portion of the key is shared.

- What kind of attack is a standard Diffie-Hellman exchange vulnerable to?
With RSA, signing the message with your privaet key authenticates it as yours as only someone with your private key, which should be no one but you, can do so. A Diffie-Hellman exchange is suceptible to a man in the middle attack, as neither party is authenticated and the exchange cannot provide any authentication. The attack would be carried out by an attacker by intercepting each parties publically released values and substituting there own before sending them on. That way the attacker has a key for each party and simply decrypts, reads, re-encrypts and sends on the information.
This attack can be mitigated by using a public-private key pair to sign the messages. An attacker can intercept the messages but cannot forge the signatures.

> - Describe the last program or script that you wrote. What problem did it solve?
> All we want to see here is if the color drains from the guy’s face. If he panics then we not only know he’s not a programmer (not necessarily bad), but that he’s afraid of programming (bad). I know it’s controversial, but I think that any high-level security guy needs some programming skills. They don’t need to be a God at it, but they need to understand the concepts and at least be able to muddle through some scripting when required.

- How would you implement a secure login field on a high traffic website where performance is a consideration?
A TLS connection should have been set up, particularly as a high traffic website, and all communication past the login must be encrypted as well. Some security features on login pages include input sanitisation, CSRF tokens and increasing timeouts to prevent brute force login attakcs.

What are the various ways to handle account brute forcing?
Account lockouts, long period timeouts and increasing timeout.

> - What is Cross-Site Request Forgery?
> Not knowing this is more forgivable than not knowing what XSS is, but only for junior positions. Desired answer: when an attacker gets a victim’s browser to make requests, ideally with their credentials included, without their knowing. A solid example of this is when an IMG tag points to a URL associated with an action, e.g. http://foo.com/logout/. A victim just loading that page could potentially get logged out from foo.com, and their browser would have made the action, not them (since browsers load all IMG tags automatically).
Further examples:
For example, a typical GET request for a $100 bank transfer might look like:
```GET http://netbank.com/transfer.do?acct=PersonB&amount;=$100 HTTP/1.1```
A hacker can modify this script so it results in a $100 transfer to their own account. Now the malicious request might look like:
```GET http://netbank.com/transfer.do?acct=AttackerA&amount;=$100 HTTP/1.1```
A bad actor can embed the request into an innocent looking hyperlink:
```<a href="http://netbank.com/transfer.do?acct=AttackerA&amount;=$100">Read more!</a>```
If the site only uses post requests, a form can be embedded and executed on page load or embedded in XSS.

- How does one defend against CSRF?
One mitigation is to use double submission of cookies, where a token is assigned to both a cookie and included as a request parameter and any mismatch indicates the request did not come from the user, as the request will always be sent with the correct cookie, but an attacker cannot generate the cookie to use as a request parameter. 
Another method is to generate random tokens and include them as parameters in each request. The tokens are verified by the server and so an attacker should not be able to include them in malicious CSRF attacks. While effective, tokens can be exposed at a number of points, including in browser history, HTTP log files, network appliances logging the first line of an HTTP request and referrer headers, if the protected site links to an external URL. These potential weak spots make tokens a less than full-proof solution.

- If you were a site administrator looking for incoming CSRF attacks, what would you look for?
> This is a fun one, as it requires them to set some ground rules. Desired answers are things like, “Did we already implement nonces?”, or, “That depends on whether we already have controls in place…” Undesired answers are things like checking referrer headers, or wild panic.

- What’s the difference between HTTP and HTML?
One is a communication protocol, the other is a markup language.

- How does HTTP handle state?
HTTP itself cannot handle state, thus we use cookies.

- What exactly is Cross Site Scripting?
XSS is the injection of malicious javascript code that is then executed by the victims browser. It is an attack on an application's users, not the system itself.

- What’s the difference between stored and reflected XSS?
Stored XSS is stored in the database and executre upon page load, once the browser parses what it sees are regular HTML or code. Reflected comes from the user in the form of a request (usually constructed by an attacker), and then gets run in the victim’s browser when the results are returned from the site.

- What are the common defenses against XSS?
Input validation where code symbols are not appropriate, but output sanitisation is more important.

> - What is the primary reason most companies haven’t fixed their vulnerabilities?
> This is a bit of a pet question for me, and I look for people to realize that companies don’t actually care as much about security as they claim to–otherwise we’d have a very good remediation percentage. Instead we have a ton of unfixed things and more tests being performed.
>
> Look for people who get this, and are ok with the challenge.

> - What’s the goal of information security within an organization?
> This is a big one. What I look for is one of two approaches; the first is the über-lockdown approach, i.e. “To control access to information as much as possible, sir!” While admirable, this again shows a bit of immaturity. Not really in a bad way, just not quite what I’m looking for. A much better answer in my view is something along the lines of, “To help the organization succeed.”
>
> This type of response shows that the individual understands that business is there to make money, and that we are there to help them do that. It is this sort of perspective that I think represents the highest level of security understanding—-a realization that security is there for the company and not the other way around.

> - What’s the difference between a threat, vulnerability, and a risk?
> As weak as the CISSP is as a security certification it does teach some good concepts. Knowing basics like risk, vulnerability, threat, exposure, etc. (and being able to differentiate them) is ?important for a security professional. Ask as many of these as you’d like, but keep in mind that there are a few differing schools on this. Just look for solid answers that are self-consistent.

>- If you were to start a job as head engineer or CSO at a Fortune 500 company due to the previous guy being fired for incompetence, what would your priorities be? [Imagine you start on day one with no knowledge of the environment]
> We don’t need a list here; we’re looking for the basics. Where is the important data? Who interacts with it? Network diagrams. Visibility touch points. Ingress and egress filtering. Previous vulnerability assessments. What’s being logged an audited? Etc. The key is to see that they could quickly prioritize, in just a few seconds, what would be the most important things to learn in an unknown situation.


- What is the difference between threat, vulnerability, and a risk?
A threat is from an attacker that will use a vulnerability that was not mitigated because someone forgot to identify it as a risk.

- What is more important for cybersecurity professionals to focus on, threats or vulnerabilities?
Apparently no right answer but I'm inclined to say address vulnerabilities and assume threats are always lurking.

- What are your first steps when securing a server?
Ports, Permissions, Packages, Passwords:
    - Close unnecessary ports and secure all others with firewall and secure authorisation (avoid those that do not support authorisation/encryption - telnet, tftp). Set all possible services to run on localhost.
    - Set strict root only access on all critical files. Set strict low privileges on other users and avoid privilege escalation via misconfigured permissions. Refuse root login via SSH.
    - Update the OS. Assess all packages and remove any unecessary ones to reduce attack vector.
    - Ensure all passwords are changed and secured.

- What could you do to prevent a man-in-the-middle attack?
Man in the middle attacks are primarily about a lack of authentication. Storing accepted public keys from official sources before connection can be one method. Do not send traffic over unencrypted channels and ensure the SSL channels use both an encryption scheme and the public-private key pair for authentication. The messages should be signed with the other parties private key, a signature that cannot be forged.
Another common instance is session hijacking through redirection away from HTTPS sites. Many sites will redirect from their HTTP to their HTTPS version and begin the encryption process. An attacker can hijack the session at this point and force the client to remain on the HTTP site, impersonating them and forwarding on all traffic. With no certificate involved, the user in unaware. This can be mitigated through HTTP Strict Transport Security (HSTS) which is a mechanism sent through headers that means connections are only accepted on HTTPS ports.

- How would you harden user authentication?
Two-factor authentication is the major method. For protocls like SSH, the public key is uploaded to the server you would like to log in to. Then the SSH session can use a challenge-response action to test the presence of the *private* key, thus authenticating the user further. Other methods include Bcrypt hashing, 

- Are there any special considerations for securing services in the cloud?
Remote access for users means remote access for hackers, so data stored in the cloud should be done only when physical storage options are limited or non-existant. Added risks include:
    - Remote logins mean the attacker does not need physical access to the server to access the data. By simply aquiring logins they now have access.
    - Authentication tokens can be hijacked and used without needing login credentials. This would not be possible if access to the physical device or SMB network was required.

- What is an IPS and how does it differs from IDS?
IPS - Intrustion prevention system will actively prevent intrusion through IP blacklisting or packet dropping. An IDS will detect it but not act.

- What is a Security Misconfiguration?
Security misconfiguration is the risk posed by misconfigured files, credentials or setups of services etc.

- What is a firewall?
A firewall is a device or software that blocks traffic based on a pre-defined set of rules or criteria.

- HIDS vs NIDS and which one is better and why?
HIDS is host intrusion detection system and NIDS is network intrusion detection system. Both the systems work on the similar lines. It’s just that the placement in different. HIDS is placed on each host whereas NIDS is placed in the network. For an enterprise, NIDS is preferred as HIDS is difficult to manage, plus it consumes processing power of the host as well.

- Various response codes from a web application?
1xx - Informational responses
2xx - Success:
    - 200 - OK
    - 204 - No Content -> The request was successful and the server returned no content
3xx - Redirection
4xx - Client side error
    - 400 - Bad Request -> The server cannot or will not process the request due to an apparent client error (e.g., malformed request syntax, size too large, invalid request message framing, or deceptive request routing)
    - 401 - Unauthorised -> Similar to 403 Forbidden, but specifically for use when authentication is required and has failed or has not yet been provided.
    - 403 - Forbidden -> The request was valid, but the server is refusing action. The user might not have the necessary permissions for a resource, or may need an account of some sort.
    - 404 - Not Found
    - 405 - Method Not Allowed -> A request method is not supported for the requested resource; for example, a GET request on a form that requires data to be presented via POST, or a PUT request on a read-only resource.
5xx - Server side error
    - 500 - Internal Server Error -> General Error
    - 503 - Service Unavailable

- When do you use tracert/traceroute?
In case you can’t ping the final destination, tracert will help to identify where the connection stops or gets broken, whether it is firewall, ISP, router etc.

- What are the three ways to authenticate a person?
Something they know (password), something they have (token), and something they are (biometrics). Two-factor authentication often times uses a password and token setup, although in some cases this can be a PIN and thumbprint.

- What is the CIA triangle?
Confidentiality, Integrity, Availability. As close to a ‘code’ for Information Security as it is possible to get, it is the boiled down essence of InfoSec. Confidentiality- keeping data secure. Integrity- keeping data intact. Availability- keeping data accessible.

- What is the Three-way handshake? How can it be used to create a DOS attack?
The three-way TCP handshake is a SYN, let's connect, a SYN-ACK, yes ok let's connect, and a final ACK, to acknowledge the connection. By sending a SYN and the following a SYN-ACK with another SYN, the server will wait the full timeout period for the final ACK from the first SYN. By sending many back to back SYN packets, a server may be overloaded and deny service.

- Where are windows user passwords kept
The password hashes are stored in the Windows SAM file. This file is located on your system at C:\Windows\System32\config but is not accessible while the operating system is booted up. These values are also stored in the registry at HKEY_LOCAL_MACHINE\SAM, but again this area of the registry is also not accessible while the operating system is booted.

- What is ARP?
ARP determines the mapping of an IPv4 address to the underlying MAC address. The implementation of the MAC protocol decodes the MAC PDU and delivers the User-Data to the IP-layer. To transfer packets between nodes via an IP address, the layer 3 must use layer 2, likely ethernet to perform the transfer. Ethernet requires a MAC address and so this must be handled by the ARP. Theoretically, Layer 2 and Layer 3 could be combined into a single layer, avoiding the need for ARP, however any changes to Ethernet would then require changes in IP and vice versa.

- What is ARP Cache poisoning
Address Resolution Protocol is used to link layer 3 and layer 2, essentially resolving an IP address to the relevant MAC and therefore device. When data is sent to an IP, an ARP request is sent throughout the network by the networks router to find which IP owns which MAC address. These addresses are stored in a cache or local storage. ARP Cache Poisoning takes advantage of the fact that ARP is a stateless protocol without authentication. Network hosts will automatically cache any ARP replies they receive, regardless of whether network hosts requested them. Even ARP entries that have not yet expired will be overwritten when a new ARP reply packet is received. Thus the ARP is 'poisoned' with the attackers MAC.
Common mitigations include static ARP entries although this is very manual. Another method involves intelligent monitoring and notifications of a changed address.

- What is a WAF and how does it differ from a normal firewall?
Web application firewall or WAF is an application firewall for HTTP conversations, applying a set of rules to these conversations. A WAF may be calibrated to protect against common attacks such as XSS or SQLi. While a normal filewall sits between servers, blocking and inspecting packets based on their source, destination, frequency etc. a WAF will inspect the HTTP packets and attempt to block the common attacks above. The WAF may do that by inspecting packets for illegal characters or requests.

- What is SSRF?
In a typical network, there may be one public facing application, e.g. 10.0.0.1 or server that communicates to protected internal servers, e.g 10.0.0.2. It may be set up that only communications from the authorised server 10.0.0.1 are able to request resources or make changes to 10.0.0.2. However, incorrect configurations in 10.0.0.1 may mean it is possible to attack 10.0.0.2 indirectly.
If the public facing server consumes and forwards information or input without sanitsation or performs requests based on user input without sanitsation or verification, it may be liable to a SSRF attack.
An example is an HTTP request, sent to 10.0.0.1 that is then forwarded to 10.0.0.2 without or with limited checking. The attacker may be able to request sensitive information or access protected services via this pass through request.

### Potential Networking Questions ###
- What is a router? What is routing?
A router allows internetwork communication between directly connected networks and communication between indirectly connected networks through the process of routing.  

- What is NAT?
NAT is Network Address Translation is a method of remapping one IP address space into another by modifying network address information in IP header of packets while they are in transit across a traffic routing device. **IP Masquerading** is a technique that hides an entire IP address space, usually consisting of private IP addresses, behind a single IP address in another, usually public address space. The term NAT has become virtually synonymous with IP masquerading.
Essentially the practice involves having one public facing IP for a network, the router, and all incoming and outgoing requests and responses and then mapped to that internal address by the router.
E.g Request => Internal Host:<To: 132.452.564.324, From: 10.0.0.1> 
                    --> Router:<To: 132.452.564.324, From: Public IP> 
                            --> Web Server<132.452.564.324>

Response    => Web Server: <To: Public IP, From: 132.452.564.324>
                    --> Router:<To: 10.0.0.1, From: 132.452.564.324>
                            --> Internal Host<10.0.0.1>
This means incoming requests or responses are dropped by the NAT or Firewall unless ports are forward. **(To be fact checked)**

- Describe each layor of the condensed OSI Model
    - Layer 1: The physical layer allows the transmission of raw bit streams on phyiscal mediums. I.e. electronic signals over sensors, switches and wires. Important concepts here include encoding, timing, pin layout, voltages etc. A physical transmission medium can include electrical cable, fibreoptic and radio signal.
    - Layer 2: The Link layer allows for higher level transmission of data *frames* between two nodes connected by a physical layer. The nodes are directly connected. It detects and potentially corrects error occurring in the physical layer and controls the flow between the nodes. It operates using frames over ethernet, Wifi or Zigbee.
    - Layer 3: The Network layer allows for the transferring of **packets** between 'different networks'. This is usually done through the Internet Protocol which specifies the addressing of hosts. 
    - Layer 4: The transport layer provides the functional and procedural means of transferring variable-length data sequences from a source to a destination host, while maintaining the quality of service functions. Although not developed under the OSI Reference Model and not strictly conforming to the OSI definition of the transport layer, the Transmission Control Protocol (TCP) and the User Datagram Protocol (UDP) of the Internet Protocol Suite are commonly categorized as layer-4 protocols within OSI.
    - Layer 5: The application layer is the OSI layer closest to the end user, which means both the OSI application layer and the user interact directly with the software application.

- What is a DHCP server?
A DHCP dynamically assigns IP addresses to a host joining the network. It is used to efficiently allocate IP addresses to hosts that need it. A router or a residential gateway can be enabled to act as a DHCP server. Most residential network routers receive a globally unique IP address within the ISP network. Within a local network, a DHCP server assigns a local IP address to each device connected to the network.

- Private IP Space
24-bit block	10.0.0.0 – 10.255.255.255	16,777,216
20-bit block	172.16.0.0 – 172.31.255.255	1,048,576
16-bit block	192.168.0.0 – 192.168.255.255	65,536

- Subnetting
Subnetting is the act of splitting a network into two or more subnets to make the intra-network communication more secure or efficient. 
A walkthrough of subnetting:
Suppose we had a network with an IP allocation of 10.x.x.x. This gives us a default subnet mask of 255.0.0.0 which tells us that the first three octets are the network number, and the last, the host number. This allows up to 16,777,216 hosts.
The subnet mask is 255.0.0.0 because this is the value used in a routers AND operation to determine whether the traffic is intra or inter network. This is also represented as a /8 subnet, as only 8 bits are reserved for the network address.
Breaking this up into a class B network means we take a further 8 bits into the subnet mask, to 255.255.0.0 or /16. The logic follows for a /24 and /32 mask network. The increases in masking don't need to be in jumps of 8 either. A /17 is valid and represents a mask of 255.255.128.0. 



What are the primary design flaws in HTTP, and how would you improve it?
If you could re-design TCP, what would you fix?
What is the one feature you would add to DNS to improve it the most?
What is likely to be the primary protocol used for the Internet of Things in 10 years?
If you had to get rid of a layer of the OSI model, which would it be? -->