---
layout: post
title: "KARMETASPLOIT"
date: 2013-06-12 04:49
comments: true
tags: [security, metasploit]
---
 Wireless networks have become very common in today's world, people are used to be connected to wireless networks in office, home, coffee shops etc. In order to facilitate the process of connecting to the wireless network, most of the operating systems often remember the previous networks connected to (often stored in Preferred Networks List) and send continuous probes looking for these networks. Once the network is found, the system automatically connects to the network. If more than one of the probed networks is found, it connects to the network with the highest signal strength (though it may vary sometimes on the operating system used).Since these clients send continuous probes, any hacker within the radio frequency range can listen passively and see the networks the client is probing for. Because of the vulnerabilities in the implementation of the algorithms for connecting to previous networks, it is possible for an attacker to set up a custom station (Access point) and have the victim connect to it. Once the victim is connected to the Fake AP the attacker has IP-level connectivity to the victim and can launch a bunch of attacks against the victim.

 <!--more-->

Dino Dai Zovi ann Shane Macaulay, 2 security researchers, wrote a set of wireless security tools developed as a Proof of concept for this vulnerability and called it Karma. It was later integrated with Metasploit and called Karmetasploit, so when a victim connects to the fake AP, karmetasploit launches all the suitable attacks available in the Metasploit framework against the vicitm. Karmetasploit also implements various evil services like DNS, POP3, FTP, SMB etc and responds to the client's requests for these services. That way, we can also capture passwords and other credentials. In this article we will look at a demo of Karmetasploit in action.

## Configuration

The first thing is to download the Karma resource file from Offensive security website as shown in the image below.

![1]({{site.baseurl}}/images/posts/Karmetasploit//1.png)

The next step is to set up a DHCP server in place. This is because clients will expect an IP-address to be handed over to them when they connect to the access point. We will need to install the dhcp3-server utility, and also specify a custom configuration file. Set up the configuration file as shown in the figure below.

![2 Actual]({{site.baseurl}}/images/posts/Karmetasploit//2 actual.png)

a) At the start of the configuration file is the place for setting your global parameters, as we can see the domain name server address has been set to 10.0.0.1\. Some things to note about the settings is that some lines start with the option keyword whereas some do not. The lines that start with the "option" keyword correspond to the options for the DHCP server, whereas the lines that do not start with the option keyword can either control the behaviour of the DHCP server, or specify some client parameters that are not optional in the DHCP protocol.

b) In the second and third line we have set up the default and the maximum lease time, this actually specifies the time after which the DHCP server will reclaim and reallocate the IP addresses. Usually the lease renews itself halfway through the set time. For e.g if the lease time is set to 2 hours, the lease will renew itself after every 1 hour. In this case the default and the maximum lease time has been set to 1 minute and 1.2 minutes respectively.

c) The fourth line states "ddns-update-style none;", this basically specifies the dynamic dns update style to be used. In this case we have set it to none.

d) In the fifth line the term "authoritative" means that the DHCP server is the official DHCP server for the local network. Obviously we should set it to authoritative to avoid any sort of suspicion.

e) The sixth statement "log-facility local7" specifies the DHCP server to do all its logging on the specified log facility (in this case local7). There are a number of log facilities available but not all log facilities are available on all systems. Some of the popular log facilities include auth, daemon, security, syslog etc.

f) In the next line the configuration for an internal subnet is set inside curly braces, this means that the options inside the braces specify only to the systems with subnet 10.0.0.1 and netmask 255.255.255.0 .The range parameter specifies the range in which the IP-Addresses will be handed over to the client. In this case the clients connected to the Fake AP will be assigned the IP-addresses in the range 10.0.0.25-10.0.0.254\. We have also set up the domain name server's and the router's IP-address inside the curly braces.

Let's have a quick look at the karma resource file, type "cat karma.rc" to show the contents of the file.

![3 Actual]({{site.baseurl}}/images/posts/Karmetasploit//3 actual.png) ![4 Actual]({{site.baseurl}}/images/posts/Karmetasploit//4 actual.png)

First of all it loads the database "db_connect postgres:toor@127.0.0.1/msfbook" and then loads the browswer_autopwn server, the browser_autopwn is useful in launching a number of browser based exploits against the client as soon as the client opens up the browser. It then sets the server's IP, port and the URL. After that, a number of servers are started on different ports. One of the notable ones is the fakedns server which listens on UDP port 53 (which is used by default for handling DNS requests), receives the requests from the clients for different domain names and hands out an incorrect IP-address for that domain.

## Launching the Attack

Now it's time to set up the Fake-AP, we need a good wireless card for that, i am using "Alfa AWUS036H" 1000mW USB Wireless WiFi Adapter, it's a very popular wireless card and supports packet injection. One of the other cool things about it is that Backtrack already has drivers installed for this card and hence we don't need to do any custom setup or installation. Just plug it in and it will work. We will start by setting our wireless card in monitor mode, which allows us to sniff all the packets in our RF range, hence we can see what networks the clients nearby are probing for. Type in the "iwconfig" command to see the wireless interface. Type in "airmon-ng start interface-name" to set up the card in monitor mode.As we can see a virtual interface named mon0 has been created on top of wlan0 interface and it is currently in monitor mode.

![5]({{site.baseurl}}/images/posts/Karmetasploit//5.png)

Let's do a simple hack now to increase the signal strength of our wireless card. This is because we want the nearby clients to connect to us and hence our card should provide good signal strength which will increase the chances of clients connecting to us. Different countries have different range of power levels under which the wireless card will have to operate, for e.g in my case (INDIA), the maximum allowed strength for a wireless card is 20 DBm (approximately 100mW), however my card supports upto 30 DBm (1000 mW). The command "iw reg set IN" sets the regulatory domain to India (IN).

![6]({{site.baseurl}}/images/posts/Karmetasploit//6.png)

Linux however allows us to change the regulatory domain, and hence we can bypass the regulatory restriction. But which country should we change the regulatory domain to? One of the answers is "Bolivia", as it allows our card to operate at the maximum power level i.e 30 DBm. Type in the commands as shown in the figure below to set the transmission power to 30DBm

![7]({{site.baseurl}}/images/posts/Karmetasploit//7.png)

As we can see the tx-power is now set to 30 DBm.

We will now set up our Fake AP. We will use the utility airbase-ng which is available by default in Backtrack. Type in the following command as shown in the figure below to set up the Fake AP. The name of the AP is set by the -e option, the -P option asks airbase-ng to respond to all probes, the "-C 30" option asks airbase-ng to send Beacon frames with the ESSID of all the probed networks after every 30 seconds, so that a client in the RF range probing for the same network will be fooled and will connect to the Fake AP set up by the attacker if we are able to provide a better signal strength than the actual network. Finally we specify the interface name which is mon0.

![8]({{site.baseurl}}/images/posts/Karmetasploit//8.png)

We can also see that airbase-ng has created a virtual interface named "at0". This interface will be used by Karmetasploit.

We can verify from airodump-ng that a network named "InfoSecInstitute" has been created, also in the lower section under the probes section we can also see that some clients are probing for networks and the name of the network is given under the Probes section. Type in the command "airodump-ng mon0" to see a similar output, where mon0 is the interface which is currently set up in monitor mode.

![9]({{site.baseurl}}/images/posts/Karmetasploit//9.png)

The next step is to set up the DHCP server. First install the dhcp3-server by using the following command as shown below, as you can see i already have the latest version. Also don't forget to backup the original dhcpd.conf file when creating a new one for our DHCP server.

![10]({{site.baseurl}}/images/posts/Karmetasploit//10.png)

Now we are going to start up the DHCP server, first we assign an IP-address and a netmask to the at0 interface and set it up. Next we start the DHCP server by typing the command as shown in the figure below. As we can see from the command that we are passing our previously created dhcpd.conf file as an input.

![11]({{site.baseurl}}/images/posts/Karmetasploit//11.png)

To verify that the DHCP server is running, let's do a quick "ps aux | grep dhcpd " and check the output. As we can see the dhcpd service is up and running.

![12]({{site.baseurl}}/images/posts/Karmetasploit//12.png)

Also it would be a good idea to see the messages log file to see the IP addresses being handed out when the DHCP server is responding to our clients. It is also evident from the output that the dhcpd3 service is listening for requests.

![13]({{site.baseurl}}/images/posts/Karmetasploit//13.png)

Now it's time to start up Karmetasploit, go to the terminal and type in "msfconsole -r karma.rc". We can see that the karma resource file is being given as an input to Metasploit. You will get a somewhat similar output as shown in the figure below.

![14]({{site.baseurl}}/images/posts/Karmetasploit//14.png)

In this image, Karmetasploit is creating different tables to store various info about the client like credentials, email address etc.

![15]({{site.baseurl}}/images/posts/Karmetasploit//15.png)

In this image, Karmetasploit is setting LHOST to 10.0.0.1, and setting other options like Server Port (SRVPORT) , path of the url (URIPATH) etc. It is also starting up services like POP3 , Ftp .

![16]({{site.baseurl}}/images/posts/Karmetasploit//16.png)

Here it is loading all the autopwn exploits, and setting up their corresponding payloads. Most of these payloads are reverse payloads, which can also work if the victim is behind a NAT, as the connection is made being from the client to the attacker and not from the attacker to the client.

![17]({{site.baseurl}}/images/posts/Karmetasploit//17.png)

Finally, a payload handler is started by Karmetasploit, and it starts listening for connections from the client. If all goes well, you will get a "Server started" message in the end.

It's now time for the client to connect to our Fake Access point, we can wait for some time for the client to connect or we can simply carry out a deauthentication attack against the client. After being deauthenticated from the network the client will again try to reconnect to the network, however if our signal strength is more, it will connect to our Fake AP as our Fake AP will be the first to respond to their probes. The figure below shows how to carry out a broadcast deauthentication attack. Note that it is essential to first put your card on the same channel as the access point. This attack will basically tell all the clients associated with the Access point to disconnect from it. Hence we have to give the bssid of the access point using the "-a" option which we can find easily using Airodump.

![18]({{site.baseurl}}/images/posts/Karmetasploit//18.png)

Once the client connects to us, an IP-address will be handed over to it from our DHCP server, we can find this out by looking at the logs file, as we can see, a number of IP-addresses are being handed over to clients that connect to our Fake AP.

![19]({{site.baseurl}}/images/posts/Karmetasploit//19.png)

Also in the Airbase-ng output we can also see that some clients have connected to us.

![20]({{site.baseurl}}/images/posts/Karmetasploit//20.png)

When the use opens up his browser, all he sees is this page showing the Loading sign, this page however can be modified by the hacker to make a custom web page as the html file is present in the Metasploit directory. In this case i connected 2 systems running Mac OS and Windows 7 respectively.

![21]({{site.baseurl}}/images/posts/Karmetasploit//21.png)

In the background however, a lot of action is happening as is evident from the karmetasploit output below. We can see two Javascript reports in the output. Karma has identified the operating systems running on these systems as well as the browser and their versions. Based on that it has identified some exploits and is starting to drop the payloads on the systems. In this case i am using fully patched Mac OS and Windows 7 systems, hence it is not able to get a shell on the system.

![22]({{site.baseurl}}/images/posts/Karmetasploit//22.png)

The results are different when working with an unpatched windows box, we get a meterpreter session on the system, also note that we must migrate to another process as soon as we get a shell because the user may close the browser and hence our connectivity may be lost.

![23]({{site.baseurl}}/images/posts/Karmetasploit//23.png)

## Conclusion

Wireless networks have become very common these days, however due to the vulnerabilities in the algorithms used to connect to a wireless network, an attacker can force a client to connect to a Fake AP after which the attacker will have IP-level connectivity to the client and can launch a bunch of exploits against it. Karma was developed as a POC for this vulnerability and later integrated with the Metasploit framework and called Karmetasploit. Karmetasploit uses the vulnerability in the 802.11 protocols and the exploits present in metasploit framework to carry out a directed attack at the victim. One of the good features about this kind of attack is that it will even work against secured wireless networks.

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).