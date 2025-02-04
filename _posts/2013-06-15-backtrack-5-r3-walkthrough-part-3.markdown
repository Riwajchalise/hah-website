---
layout: post
title: "Backtrack 5 R3 Walkthrough part 3"
date: 2013-06-15 03:12
comments: true
tags: [security, backtrack]
---

This article is in continuation to part 2 of the Backtrack 5 r3 walkthrough series. In this article we will we looking at some of the other new tools that were added into Backtrack 5 with the release of its latest version R3\.


## Wifite

Wifite is probabaly one of the best tools out there for cracking wireless networks. It just makes the whole task so simple for you by hiding all the intricate details of cracking a wireless network and making the whole process automated. It can crack WEP/WPA/WPS encrypted networks in a row. Some of the features of Wifite are...

*   Automate the whole process of cracking wireless networks. Just run the python file wifite.py and it will start scanning for wireless networks nearby and will ask you to select which targets to attack.
*   Automatically detects hidden Essid's by deauthenticating the connected client and checking the association and authentication packets to figure out the Essid.
<!--more-->
*   Backs up all the cracked passwords as well as the WPA handshakes so you can use them later.
*   You can customize the attack by selecting the type of attack (for e.g arp request replay for cracking WEP), select the pps (packets per second), set the channel to sniff on etc.
*   Gives you options in the middle of an attack. Just press Ctrl+C and it will ask you whether you want to quit, move on to the next target, or start cracking (if you think enough info has been collected) etc.
*   Very easy to upgrade. Just type _./wifite.py -upgrade_ to upgrade.

In this example, we will use wifite to crack WEP. To have a look at all the commands that wifite has to offer, just type _./wifite.py -h_

![1]({{site.baseurl}}/images/posts/bt5r3/1.png)

Firstly, let's just try a generic attack. Just type _./wifite.py_ and you will see that it automatically puts a wireless interface into monitor mode and starts scanning for the nearby wireless networks.

![2]({{site.baseurl}}/images/posts/bt5r3/2.png)

As you can see from the figure below, it found 2 nearby networks. It also lets you know if it found any clients connected to it as it is important sometimes to have a client associated with the network too. Press Ctrl + C when you think wifite has found all the nearby wireless networks.

![3]({{site.baseurl}}/images/posts/bt5r3/3.png)

It will now ask you to select the target for the network which you want to attack. You can also specify multiple networks seperated by commas. In this case we are interested in cracking the network with the ESSID "Infosec Test" which i set up for testing purposes. So i just type 1 and press enter.

![4]({{site.baseurl}}/images/posts/bt5r3/4.png)

Once you have done this, you will see that it has started the attack against the network. In this case, it is using the arp-replay attack to crack the WEP key. The minimum number of IV's for wifite to being cracking the WEP key is 10000 by default. But you can always change that in wifite. Also, it is always a good option to specify the pps for every attack as sometimes wifite will try to capture packets at higher rates which might turn your wireless card into a denial of service mode and hence stop the attack. The following command shown below helps to set both the things that we just discussed.

![5]({{site.baseurl}}/images/posts/bt5r3/5.png)

Coming back to our previous attack against the "Infosec Test", we see that it is still capturing IV's.

![6]({{site.baseurl}}/images/posts/bt5r3/6.png)

Once it has captured sufficient IV's, you will notice that it cracked the WEP key. In this case, it found the WEP key to be "0987654321". This is one of the reasons why wifite is such an awesome tool out there for wi-fi cracking.

![7]({{site.baseurl}}/images/posts/bt5r3/7.png)

Wifite also offers some other cool customization options. For e.g the following command will ask wifite to endlessly attack the target WEP network. This means that the program will not stop until it has cracked the WEP key for the target network. This attack could be handy in case when you are near a network that does not have any connected client to it or has very less activity. Just use this command and forget about it, wifite will automatically crack the WEP key as soon as it gathers sufficient information. You will also notice that it informs you whether you have already cracked the network by looking at its database. Having a database is another handy feature as it stores all the cracked passwords for all the networks as well as any captured WPA handshakes so that you can carry out a bruteforce attack whenever you want.

![8]({{site.baseurl}}/images/posts/bt5r3/8.png)

The following command will scan for all nearby WPA networks and store the WPA handshakes without carrying out a bruteforce attack. This feature could come in handy when you want to gather the information as quickly as possible in a particular location. You can always crack the WPA key using the handshake somewhere else.

![9]({{site.baseurl}}/images/posts/bt5r3/9.png)

If you want to know more about the wifite tool, i recommend you have a look at its official website [http://code.google.com/p/wifite/](http://code.google.com/p/wifite/).

## Iphone Analyzer

Iphone Analyzer is an IOS device forensics tool. Unlike its name, Iphone Analyzer works for all IOS devices. It is used to analyze data from iTunes backups and provides a rich interface to explore the content of the device as well as recover them. In case of Mac OS X, Iphone analyzer automatically detects the location of the backup file. However, while using it with Backtrack 5 R3, you will have to provide it with the location of the backup file. It also allows you to analyze your IOS device over SSH, which is a very handy feature. Though this feature of Iphone analyzer is still in the beta version but this feature can be very useful especially when performing penetration tests on jailbroken IOS devices. Iphone analyzer allows you to see your text messages, photos, call records etc. IOS uses sqlite for managing its database. Iphone analyzer also allows you to analyze the various sqlite files, the schema which is used to enter data into the database as well as the contents of the file. It also allows you to browse the file structure like you would normally do via a terminal on a jailbroken device.

To analyze a jailbroken device using Iphone Analyzer, click on File --> Open: Other --> Open over SSH. Then enter the IP address of the device and the ssh username and password.

![15]({{site.baseurl}}/images/posts/bt5r3/15.png)

Let's use Iphone Analyzer to analyze a backup file. As you can see from the figure below, on a MAC OS, it automatically detects the locations of the backup files. While running it on Backtrack 5, you will have to give it the location of the backup file. Click on the backup file you want to analyze and then click on "Analyze Iphone backup"

![11]({{site.baseurl}}/images/posts/bt5r3/11.png)

Once this is done, you are presented with this beautiful interface that allows you to explore the contents of your backup file. At the top-center, you will see a lot of information about the IOS device like GUID, Serial Number, UDID, the last backup date, the phone number etc. On the bottom of this, you will see a detail section which contains all the information that Iphone Analyzer could obtain from the info.plist file. On the left side, you will see a Bookmarks and File System section. On the right side is the Manifest section. This gives you a lot of the information about the actual path of your applications in the directory structure. Please note that since this is not a jailbroken device, most of the information will be non-readable.

![12]({{site.baseurl}}/images/posts/bt5r3/12.png)

As shown in the figure below, the Bookmarks section will allow to find out the information which you are most likely to be looking for. It includes information about your Address Book, Voicemail, Location Map, Facebook Friends, Messages, incoming and outgoing calls etc.

![13]({{site.baseurl}}/images/posts/bt5r3/13.png)

Similarly, if you want, you can explore the filesystem of the device by clicking on the Filesystem tab. This will allow you to look at the various sqlite and plist files. The figure below shows information about a plist file that apple stores on your system named "com.apple.wifi.plist". Using this file, it is possible to figure out the latest networks you have connected to. As you can see from the image below, the plist file tells me information about a network with the ESSID "Buzz@Barista".

![14]({{site.baseurl}}/images/posts/bt5r3/14.png)

You can also analyze various sqlite files using Iphone Analyzer. The figure below shows the database structure of a sqlite file named ocspcache.sqlite3\.

![10]({{site.baseurl}}/images/posts/bt5r3/10.png)

Alternatively, if you just want to look at the content stored in the sqlite file, you can click on "Browse Data" and this will show you the all the database information about the sqlite file.

![17]({{site.baseurl}}/images/posts/bt5r3/17.png)

If you will at the very bottom, you will see a button named "Deleted fragments". Another cool feature of Iphone analyzer is to recover deleted items from the database, though it is not as effective but still very useful under certain circumstances.

![16]({{site.baseurl}}/images/posts/bt5r3/16.png)

## References:

*   Wifite official website  
    [http://code.google.com/p/wifite/](http://code.google.com/p/wifite/)

*   Iphone Analyzer user guide  
    [http://www.crypticbit.com/files/ipa_user_guide.pdf](http://www.crypticbit.com/files/ipa_user_guide.pdf)

This article was originally published on the [resources](http://resources.infosecinstitute.com/) page at [Infosec Institute](http://infosecinstitute.com/). For more information, please visit my author [page](http://resources.infosecinstitute.com/author/prateek/).