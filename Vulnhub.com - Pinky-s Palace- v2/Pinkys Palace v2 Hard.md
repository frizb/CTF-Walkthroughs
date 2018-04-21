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

31337 - Elite


