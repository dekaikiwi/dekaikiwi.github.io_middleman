---
title: List of Useful Network Terminal Commands
date: 1555692647
---


# List of Useful Network Terminal Commands

Here are some useful utilities for working with networks on a UNIX system.

## ping

The quintessential "are you alive" utility. Uses ICMP protocol to see if server at the address given as an argument is responding.

Usage:

~~~
$ ping google.com
PING google.com (172.217.161.78): 56 data bytes
64 bytes from 172.217.161.78: icmp_seq=0 ttl=55 time=13.959 ms
64 bytes from 172.217.161.78: icmp_seq=1 ttl=55 time=11.498 ms
64 bytes from 172.217.161.78: icmp_seq=2 ttl=55 time=13.012 ms
~~~

## traceroute

Tracks the path a packet takes through to the server.

Useful for..

- Seeing how many "hops" a packet must go through to get the the host server.
- Finding any network defects between you and the host server.

Traceroute exploits the implementation of the TTL (Time to Live) field and ICMP protocol. Typically when a router decrements the TTL field value to 0 it will drop the packet and send some ICMP message to the ip address listed in `src` (source) to say that the packet was dropped.

A usual TTL value would be some value of 60 (for example), traceroute starts with a TTL of 1. Which means the first router that receives the packet will decrement the counter to one and drop the packet. traceroute uses the information in the recieved ICMP "dropped" packet. To find the identify of the first "hop" router. Then sends a new packet with a TTL of 2 and so and and so forth until it gets to the destination server.


Usage:

~~~
$ traceroute google.com
traceroute to google.com (172.217.26.46), 64 hops max, 52 byte packets
 1  192.168.1.1 (192.168.1.1)  1.447 ms  1.063 ms  0.999 ms
 2  xxx.xxx.xxx.xxx (xxx.xxx.xxx.xxx)  3.427 ms  3.533 ms  4.229 ms
 3  xxx.xxx.xxx.xxx (xxx.xxx.xxx.xxx)  3.932 ms  3.700 ms  3.206 ms
 4  xxx.xxx.xxx.xxx (xxx.xxx.xxx.xxx)  4.641 ms  4.283 ms  4.708 ms
 5  133.208.191.163 (133.208.191.163)  5.172 ms
    133.208.191.35 (133.208.191.35)  10.472 ms  5.747 ms
 6  72.14.205.0 (72.14.205.0)  5.984 ms  4.537 ms  4.719 ms
 7  * * *
 8  108.170.233.18 (108.170.233.18)  8.257 ms
    108.170.233.20 (108.170.233.20)  12.567 ms
    108.170.236.4 (108.170.236.4)  7.003 ms
 9  72.14.238.75 (72.14.238.75)  6.151 ms
    72.14.234.199 (72.14.234.199)  5.153 ms  4.661 ms
10  nrt12s17-in-f14.1e100.net (172.217.26.46)  4.444 ms  4.683 ms  4.447 ms
~~~

Note that a `*` in the output is a hop where the packet was dropped by no ICMP response packet was received before a preset timeout. In the case you get these I would extend the timeout with the `-w` option and see if it improves the situation at all.

## mrt (My Traceroute)

A combination of the `ping` and `traceroute` commands. `mrt` gives a constant update on the path your packets will take to the destination server and gives useful information on specific hops where you experience an unexpected packet loss and how long on average each response takes. Useful for identifying problematic nodes that aren't down, but perhaps performing badly.

Usage:

~~~
$ mrt google.com
~~~

## tcpdump

Provides the raw dump of packets being sent and received by any of your network interfaces. Despite the name, displays packets from other IP protocols. Quite useful for seeing what is going on under the hood of some command line tools.

For example if I run `ping google.com` and then `tcpdump -nni en0 icmp` (-nn to not convert ip address to host names and -i to specify network interface) I get the following output.

~~~
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on en0, link-type EN10MB (Ethernet), capture size 262144 bytes
00:04:10.085754 IP 192.168.1.9 > 172.217.161.78: ICMP echo request, id 47710, seq 0, length 64
00:04:10.091676 IP 172.217.161.78 > 192.168.1.9: ICMP echo reply, id 47710, seq 0, length 64
00:04:11.089104 IP 192.168.1.9 > 172.217.161.78: ICMP echo request, id 47710, seq 1, length 64
00:04:11.094271 IP 172.217.161.78 > 192.168.1.9: ICMP echo reply, id 47710, seq 1, length 64
00:04:12.092213 IP 192.168.1.9 > 172.217.161.78: ICMP echo request, id 47710, seq 2, length 64
~~~

Similar with `traceroute`

~~~
00:09:54.790974 IP (tos 0x0, ttl 1, id 58126, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 1, length 52
00:09:54.792225 IP (tos 0xc0, ttl 64, id 13892, offset 0, flags [none], proto ICMP (1), length 100)
    192.168.1.1 > 192.168.1.9: ICMP time exceeded in-transit, length 80
	IP (tos 0x0, ttl 1, id 58126, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 1, length 52
00:09:54.792789 IP (tos 0x0, ttl 1, id 58127, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 2, length 52
00:09:54.793711 IP (tos 0xc0, ttl 64, id 13893, offset 0, flags [none], proto ICMP (1), length 100)
    192.168.1.1 > 192.168.1.9: ICMP time exceeded in-transit, length 80
	IP (tos 0x0, ttl 1, id 58127, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 2, length 52
00:09:54.793775 IP (tos 0x0, ttl 1, id 58128, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 3, length 52
00:09:54.794691 IP (tos 0xc0, ttl 64, id 13894, offset 0, flags [none], proto ICMP (1), length 100)
    192.168.1.1 > 192.168.1.9: ICMP time exceeded in-transit, length 80
	IP (tos 0x0, ttl 1, id 58128, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 3, length 52
00:09:54.794754 IP (tos 0x0, ttl 2, id 58129, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 4, length 52
00:09:54.798466 IP (tos 0x0, ttl 254, id 0, offset 0, flags [none], proto ICMP (1), length 56)
    133.205.148.163 > 192.168.1.9: ICMP time exceeded in-transit, length 36
	IP (tos 0x0, ttl 1, id 58129, offset 0, flags [none], proto ICMP (1), length 72)
    192.168.1.9 > 13.78.34.172: ICMP echo request, id 58125, seq 4, length 52
00:09:54.799114 IP (tos 0x0, ttl 2, id 58130, offset 0, flags [none], proto ICMP (1), length 72)
~~~

See how TTL is incrementing gradually and we receive `ICMP time exceeded in-transit` from route when TTL is decremented to 0 on their end.
