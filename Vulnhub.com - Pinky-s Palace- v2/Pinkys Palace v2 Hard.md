# Pinky's Palace: v2

## Identify the target server IP address using Nmap

'''
root@kali:~/Desktop# nmap -sP -PA21,22,25,3389 192.168.126.1/24
---SNIP---
MAC Address: 00:50:56:F3:96:7E (VMware)
Nmap scan report for 192.168.126.131
Host is up (0.00022s latency).
Nmap done: 256 IP addresses (5 hosts up) scanned in 2.02 seconds
'''

Our target is 192.168.126.131

## Kick things off with Vanquish to automate our enumeration process in the background.
https://github.com/frizb/Vanquish

'''
root@kali:~/Documents# cd Vanquish/
root@kali:~/Documents/Vanquish# echo 192.168.126.131 >> pinky.txt
root@kali:~/Documents/Vanquish# python Vanquish2.py -hostFile pinky.txt -verbose -logging
'''

With Vanquish running in the background, I begin to review the website content manually while collecting the HTTP requests and responses through BurpSuite.

## Host file shenanigans
The website looks quite broken at first and I notice that it references a host named pinkydb, so I use VI to add that to my /etc/hosts file.
'''
root@kali:/etc# cat hosts
127.0.0.1	localhost
127.0.1.1	kali
192.168.126.131 pinkydb

# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
'''


