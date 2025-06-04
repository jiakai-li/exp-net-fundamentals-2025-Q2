# Windows networking tools

Windows networking tools introduced:
- `ipconfig`: Check local network interfaces and informations
- `ping`: Test if a destination is reachable
- `tracert`: Check what is the routing between the source and the destination
- `nslookup`: Lookup the IP address for a DNS record
- `netstat`: Show active connections
- `route`: Routing table related functions
    - Persistent route: makes a route persistent across boots of system

## IPconfig

```text
# ipconfig output

Windows IP Configuration


Wireless LAN adapter Wi-Fi:

   Connection-specific DNS Suffix  . :
   IPv6 Address. . . . . . . . . . . : 2404:4400:5106:6200:f5a9:a39e:7f32:48c0
   Temporary IPv6 Address. . . . . . : 2404:4400:5106:6200:f8e6:508e:aaa7:ea15
   Link-local IPv6 Address . . . . . : fe80::d126:e148:82e:1a12%20
   IPv4 Address. . . . . . . . . . . : 192.168.1.6
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : fe80::6ecd:d6ff:fec1:cfb7%20
                                       192.168.1.1

Ethernet adapter vEthernet (WSL (Hyper-V firewall)):

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::2358:c671:cbce:b5de%54
   IPv4 Address. . . . . . . . . . . : 172.24.208.1
   Subnet Mask . . . . . . . . . . . : 255.255.240.0
   Default Gateway . . . . . . . . . :
```

I'm using my local computer to complete this home work. There are mainly two network interfaces that are of interesting in my local environment, one for the wifi connection and one for WSL. The wifi network interface is the one that will be used in this home work to connect to destinations using different commands in below

## Ping

```text
# ping www.google.com output

Pinging www.google.com [2404:6800:4006:809::2004] with 32 bytes of data:
Reply from 2404:6800:4006:809::2004: time=41ms
Reply from 2404:6800:4006:809::2004: time=30ms
Reply from 2404:6800:4006:809::2004: time=30ms
Reply from 2404:6800:4006:809::2004: time=30ms

Ping statistics for 2404:6800:4006:809::2004:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 30ms, Maximum = 41ms, Average = 32ms
```

I pinged google.com in this section, which shows:

- IPv6 is being used
- There are four packets that were being sent and received back successfully
- Each packet contains 32bytes data
- The RTT is about 30-40ms

## Trace Route

```text
# tracert www.google.com output

Tracing route to www.google.com [2404:6800:4006:809::2004]
over a maximum of 30 hops:

  1     3 ms     3 ms     3 ms  default-rdns.vocus.co.nz [2404:44xx:51xx:62xx:6axx:d6xx:fexx:cfxx]
  2    10 ms     6 ms     6 ms  default-rdns.vocus.co.nz [2xx0:4xx0::2x]
  3    32 ms    33 ms    29 ms  2620:107:4008:6ff::2
  4    30 ms    31 ms    30 ms  2001:4860:1:1::1510
  5     *       34 ms     *     2001:4860:0:1::7de5
  6    34 ms    32 ms    53 ms  2001:4860:0:1::6fb5
  7    27 ms    30 ms    27 ms  syd09s15-in-x04.1e100.net [2404:6800:4006:809::2004]

Trace complete.
```

The trace route result shows:

- There are 7 hops between my local environment and google.com
- The first two hops are likely my local ISP
- From the third hop onward the latency had increased
- There are some `*` entries, which might because the probe packet gets dropped or blocked

## Netstat
```text
# netstat -n output

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    127.0.0.1:65353        127.0.0.1:65375        ESTABLISHED
  TCP    127.0.0.1:65353        127.0.0.1:65376        ESTABLISHED
  TCP    127.0.0.1:65375        127.0.0.1:65353        ESTABLISHED
  TCP    127.0.0.1:65376        127.0.0.1:65353        ESTABLISHED
  TCP    192.168.1.6:52140      162.159.133.234:443    ESTABLISHED
  TCP    192.168.1.6:53530      35.174.127.31:443      ESTABLISHED
  TCP    192.168.1.6:53630      104.18.26.90:443       ESTABLISHED
  ....
```

Thre are mainly two types of connections:
- The local loopback connections (the ones with source address of `127.0.0.1`)
- The external https connections (the ones with source address of `192.168.1.6`)

Apart from the obove, we can see that:
- For the external connections, my local PC uses random chosen ports for connection
- Most of the connections are secure https connections

## Route

```text
# route print output
===========================================================================
Interface List
 20...fc xx xx xx xx c4 ......Intel(R) Wi-Fi 6 AX201 160MHz
 54...00 15 5d 47 ef 35 ......Hyper-V Virtual Ethernet Adapter
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0      192.168.1.1      192.168.1.6     30
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
     172.24.208.0    255.255.240.0         On-link      172.24.208.1   5256
     172.24.208.1  255.255.255.255         On-link      172.24.208.1   5256
   172.24.223.255  255.255.255.255         On-link      172.24.208.1   5256
      192.168.1.0    255.255.255.0         On-link       192.168.1.6    286
      192.168.1.6  255.255.255.255         On-link       192.168.1.6    286
    192.168.1.255  255.255.255.255         On-link       192.168.1.6    286
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link       192.168.1.6    286
        224.0.0.0        240.0.0.0         On-link      172.24.208.1   5256
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link       192.168.1.6    286
  255.255.255.255  255.255.255.255         On-link      172.24.208.1   5256
===========================================================================
Persistent Routes:
  None

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
 20   4126 ::/0                     fe80::6ecd:d6ff:fec1:cfb7
  1    331 ::1/128                  On-link
 20   4126 2404:4400:5106:6200::/64 On-link
 20    286 2404:4400:5106:6200:f5a9:a39e:7f32:48c0/128
                                    On-link
 20    286 2404:4400:5106:6200:f8e6:508e:aaa7:ea15/128
                                    On-link
 20    286 fe80::/64                On-link
 54   5256 fe80::/64                On-link
 54   5256 fe80::2358:c671:cbce:b5de/128
                                    On-link
 20    286 fe80::d126:e148:82e:1a12/128
                                    On-link
  1    331 ff00::/8                 On-link
 20    286 ff00::/8                 On-link
 54   5256 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```

The result shows IPv4 and IPv6 route tables in my local environment
- The default gateway in my case is `192.168.1.1`, combine with the fact that my local network submask is `255.255.255.0`, the router takes the first available IPv4 address
- There is no persistent routes in my local PC, simply because I've never touched it 💩
