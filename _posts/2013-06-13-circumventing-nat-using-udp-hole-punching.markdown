---
layout: post
title: "Circumventing NAT using UDP hole punching"
date: 2013-06-13 22:35
comments: true
tags: [networking, security]
---

A lot of the networks use NAT (Network Address Translation) these days. This allows the systems on the same network to have a single global IP address. This also assures enhanced security but at the same time adds complications specially while connecting to P2P (Peer to Peer) networks. This is because at the time of initiating a connection in a Peer to Peer network, it is not possible to determine which packet coming from the peer is intended for which host on the network simply because they have one global IP address. Also, most of the networks with NAT may drop incoming packets simply because it cannot figure out which client on the NAT the packet is directed to, or may recognize it as an unauthorized packet etc. Some of the common Peer to Peer applications are Skype, Spotify etc.

<!--more-->

UDP hole punching is one of the most common techniques used to establish UDP connections with systems behind NAT. It is called UDP hole punching because it punches a hole in the firewall of the network which allows a packet from an outside system to successfully reach the desired client on a network using NAT. Though this process needs a third party host to establish connection between the clients, previous research has shown that this can be done without using third party hosts too.

## Problems while performing UDP Hole punching

UDP Hole punching is a technique via which it is possible to traverse NAT's and establish P2P connections. This requires the use of a third party host. It is important to first discuss the problems which arise while performing UDP hole punching. Let's assume the two systems who want to communicate with each other are A and B and the third system which helps them in establishing a connection is C.

1) In order to establish connection between 2 systems, the systems should know each other's global IP address and the port to which it wants to establish the connection . However when A sends an outbound packet, then it is possible that while traversing the NAT, the NAT changes the port number of A (often called port randomization). This makes UDP Hole punching almost impossible. The IP address is also rewritten to resemble the IP address of the NAT.

2)The firewall at the NAT may not allow inbound connections from outside.

3)Both the systems A and B must know the IP address of the third party server C.

## A Small experiment

First let's do a small experiment. Make sure your system is behind a NAT. Then go to the terminal and do a ping on any website.

![1]({{site.baseurl}}/images/posts/nat-traversal/1.png)

Open up Wireshark to see the packets flowing through as shown in the figure below. We will see responses coming from the destination. But since our system is behind a NAT, how does the incoming packet know that the response is directed to us ? Well, the answer is simple. When our first packet went out of the network, NAT A noticed it and realized that some sort of communication was being established. Hence it decided that any sort of response which corresponds to this outgoing request will be forwarded to the system A. It then sees that the response had the content of the initial outgoing request and realized that it was meant for A, and hence forwarded the packet to A. What this did was that it basically punched a hole in the firewall, through which the firewall now allows incoming response which correspond to the request from A.

![2]({{site.baseurl}}/images/posts/nat-traversal/2.png)

## How UDP Hole punching works.

We now have sufficient background to understand how UDP hole punching actually works. The process works in the following way. We assume that both the systems A and B know the IP address of C.

a) Both A and B send UDP packets to the host C. As the packets pass through their NAT's, the NAT's rewrite the source IP address to its globally reachable IP address. It may also rewrite the source port number in which case UDP hole punching would be almost impossible.

b) C notes the IP address and port of the incoming requests from A and B. Let the port number for A is X and the port number for B is Y.

c) C then tells A to send UDP packet to the global IP address of the NAT for B at port Y, and similarly tells B to send UDP packet to the global IP address of the NAT for A at port X.

d) The first packets for both A and B get rejected while entering into each other's NAT's. However as the packet passes from the NAT of A to the NAT of B at port number Y, the NAT A makes note of it and hence punches a hole in its firewall to allow incoming packets from the IP address of the NAT of B from port number Y. The same happens with the NAT of B and it makes a rule to allow incoming packets from the IP address of the NAT of A from port number X.

e) Now when A and B send packets to each other, these get accepted and hence a P2P connection is established.

From a penetration tester's point of view, it is important to deduce some strategy to communicate with the victim behind the NAT.Based on the different scenarios, there can be 4 possible cases, we will assume the attacker to be the client and the victim to be the server we are trying to communicate with

Case 1: The victim and the client are both behind seperate NAT's.

Case 2: The victim and the client are both behind same NAT's.

Case 3: The victim is behind a NAT whereas the client isn't and has a global IP address.

Case 4: The client is behind a NAT whereas the victim isn't and has a global IP address.

The UDP hole punching technique discussed above will work on almost all those cases. I am using the word almost because UDP hole punching does not work on all the NAT implementations, simply because their is no standard procedure in which a particular NAT operates. In the cases where it doesn't work, the third party host is used to relay the whole traffic between A and B.

## How Skype Does it

Skype uses the UDP hole punching technique to allow communication between users who are behind NAT. However, Skype does not use a separate server to act as a third party host. Rather it uses its users computers to act as a third party host. Any client which has a publicly reachable IP can become the third party host. Hence this may increase the load on Skype's users as they are responsible for initiating connection between the users who are behind NAT. Sometimes UDP hole punching may not be possible due to various reasons like port randomization by the NAT etc. In the cases where UDP hole punching is not possible, the third party host (i.e a Skype user's system having a globally reachable IP address) is used to relay the whole communication between the users who are behind NAT.

## P2P communication without a Third party host

Some of the problems with UDP hole punching is that a third party server is always required to relay traffic or to establish UDP states between the machines. This increases the load on the third party server. There has been research to determine whether it is possible to establish P2P connection between systems without a third party host. Researchers Andreas Muller, Nathan Evans, Christian Grothoff and Samy Kamkar have discussed some theoretical methods in their paper "Autonomous NAT Traversal" which could help in achieving this goal. Their approach uses fake replies to establish connection between peers. As always, we have the 4 cases.

Case 1: The victim and the client are both behind seperate NAT's.

Case 2: The victim and the client are both behind same NAT's.

Case 3: The victim is behind a NAT whereas the client isn't and has a global IP address.

Case 4: The client is behind a NAT whereas the victim isn't and has a global IP address.

According to their paper, the technique proposed by them works quite well for the cases 3 and 4 but doesn't work quite well for the cases 1 and 2\. A proof of concept tool named "pwnat" was also released which is also available in Backtrack. We will discuss the techniques proposed by them now for cases 3 and 4 and also look at a demo. We assume that the server is behind a NAT whereas the client is trying to initiate a connection from a globally reachable IP address.

a) Open up pwnat in Backtrack. We can see that it can be run in server as well as client mode. We will run both on our local machine just to understand how the communication happens.

![3]({{site.baseurl}}/images/posts/nat-traversal/3.png)

b) Type in the following command as shown below to start the server. 10.0.2.15 stands for the IP address of our local machine and 2222 is the port from which our packets will generate.

![4]({{site.baseurl}}/images/posts/nat-traversal/4.png)

c) Now open up Wireshark and start sniffing packets. We notice that some ping requests are going to the non-existent IP address 3.3.3.3\. Note that since the packets are being passed from the NAT, the NAT will allow incoming packets from random IP address which appear to be a response for this request. Note that i am saying random IP address because the response could be from a hop on the way to the destination 3.3.3.3 (which is non-existent) and if it can prove to the NAT that it was a response for the initial request, then it will be allowed into the network and directed to our local machine.

![5]({{site.baseurl}}/images/posts/nat-traversal/5.png)

d) Now we will start the client. Type in the following command as shown in the figure below to start the pwnat client. 8000 is the port on the client from where we want the connection to be initiated. localhost 80 is the host and port to which we want to be redirected to through the communication channel.

![6]({{site.baseurl}}/images/posts/nat-traversal/6.png)

On the server side, we can see that it has got a connection request and is regularly receiving packets from the client. This confirms that a communication channel has been established. The packets that the server is regularly receiving are the keep alive packets sent by the client in order to maintain the connection state.

![7]({{site.baseurl}}/images/posts/nat-traversal/7.png)

If we look at the wireshark trace, we can see "TTL exceeded" packets being sent to the server. If we look at the content of one of the packets, we can see that the content contains the original request packet. This packet is actually a fake reply to the server indicating that the TTL has expired for the ping request to 3.3.3.3\. This packet is actually sent by the client and when it reaches the NAT of A, the NAT recognizes it as a response to the initial ping request to 3.3.3.3 sent by A and hence routes it to A. Once the packet reaches A, A figures out the IP address and port of the client and hence a bidirectional communication channel can now be established.

![8]({{site.baseurl}}/images/posts/nat-traversal/8.png)

In cases where both the systems are behind NAT, the payload of the reply from the client to the server could contain the IP address of the client as well as the port number on which it wants to initiate the connection. One of the other issues with this could be that the fake ICMP replies sent by the client could be rejected by the client's NAT. Hence this many not work in all cases. The pwnat tool though has support for communicating between NAT to NAT peers.

## Conclusion

P2P applications are very popular these days mainly because they don't require use of third party servers to relay traffic between them. However this makes it tough for systems behind NAT's to establish P2P networks between them. From a penetration tester's point of view also, it is important to establish a direct communication channel with a victim which is behind a NAT. In this article we discussed some of the techniques like UDP hole punching which use third party hosts to establish communication states between the client. Once the communication state has been established, these client can establish a bi-directional communication channel between each other. We also looked at some of the techniques of establishing a communication between two systems behind NAT without the use of third-party hosts and looked at some of the scenarios in which they could be used.

## References:

1.  Autonomous NAT Traversal  
    [grothoff.org/christian/pwnat.pdf](http://grothoff.org/christian/pwnat.pdf)

2.  UDP hole punching  
    [http://en.wikipedia.org/wiki/UDP_hole_punching](http://en.wikipedia.org/wiki/UDP_hole_punching)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).