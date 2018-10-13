+++
title = "What is TCP/IP?"
description = "TCP/IP is the suite of communications protocols used to connect hosts on the Internet. TCP/IP uses several protocols, the two main ones being TCP and IP. -- Webopedia"
tags = [
    "tcp",
    "ip",
    "arp",
    "mac",
    "packets",
    "transport",
    "transmission",
    "udp",
    "osi",
    "networking"
]
date = "2016-03-03"
highlight = "true"
+++

>  TCP/IP is the suite of communications protocols used to connect hosts on the Internet. TCP/IP uses several protocols, the two main ones being TCP and IP. -- <cite>[Webopedia][1]</cite>

# Internet Protocol (IP)
IP operates at the network layer of the OSI model, and is responsible for delivering packets of data to network enabled devices. A physical device address, known as the MAC address, is not routable across networks. The MAC address is made up of the manufacturer identifier and a serial, so although it may be globally unique (if not spoofed), there is no hierarchical information available to hop across networks. Every router would have to know about every MAC address, which would be a huge overhead. 

The **Address Resolution Protocol (ARP)** converts between the IP address and the MAC address within a network. The IP address consists of network and host segments, allowing packets to be sent to the next destination in the routing table, maintained by each router.

# Transmission Control Protocol (TCP)
TCP operates at the transport layer of the OSI model, and allows a device to reliably send packets. It is described as "connection-oriented", which means a connection is maintained until the applications on both end-points have finished sending and receiving data, or until there is an error that cannot be recovered from. 

TCP breaks up data into packets for transmission, and handles the re-sending of packets when they do not arrive, and ensures that packets are combined in the correct order after being received. 

TCP produces a one-to-one connection between two end-points. In order to broadcast to multiple devices, **User Datagram Protocol (UDP)** is used instead, but that protocol has different advantages/disadvantages.

[1]:http://www.webopedia.com/TERM/T/TCP_IP.html