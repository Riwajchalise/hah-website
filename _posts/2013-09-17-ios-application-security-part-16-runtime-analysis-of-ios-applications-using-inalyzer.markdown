---
layout: post
title: "iOS Application Security Part 16 – Runtime Analysis of iOS Applications using iNalyzer"
date: 2013-09-17 13:12
comments: true
tags: security
---
In the previous article, we looked at how we can perform static analysis of iOS Applications using iNalyzer. In this article, we will look at how we can use iNalyzer to perform runtime analysis of iOS applications. We can invoke methods during runtime, find the value of a particular instance variable at a particular time in the app, and basically do anything that we can do with Cycript.

In the last article, we were successfully able to generate the html files via Doxygen and open it up to view class information and other information about the app. For runtime analysis, we will be using the Firefox browser. The developer of this tool has personally recommended me to use Firefox as this may not work on other browsers. However, it seemed to be working fine for me on Chrome as well.

<!-- more -->

**To open up the runtime interpreter, first of all open up the index.html file generated by Doxygen for the app that you want to analyze, then just double tap the left arrow key.**

![1]({{site.baseurl}}/images/posts/ios16/1.png)

You will see a console come up on the top as shown in the figure above where we can type commands. The first thing to do is to tell iNalyzer the ip address of your device, which in this case is 10.0.1.23\. So let me just enter that on the box in the middle and press enter.

![2]({{site.baseurl}}/images/posts/ios16/2.png)

Once the IP address has been set, make sure that the app that you want to analyze is open (i.e on foreground) on your device and your device is not in sleep mode. This is important because if your app is in the background or the device is in sleep mode, then your app is temporarily paused by the operating system and hence it is not possible to perform any kind of runtime analysis on the app.

Once the app is open, just type any command on the console, just like you would type on Cycript.

![3]({{site.baseurl}}/images/posts/ios16/3.png)

As we can see, we get a response. We can now type any cycript command that we want here.

Let's hide the status bar from the app. We can do this with the command [[UIApplication sharedApplication] setStatusBarHidden:YES animated:YES];

![4]({{site.baseurl}}/images/posts/ios16/4.png)

We see that we don't get a response. Its because the response type of this method is void.

![5]({{site.baseurl}}/images/posts/ios16/5.png)

However, the status bar has been hidden in the app. Note that we no longer see the time on the top.

![6]({{site.baseurl}}/images/posts/ios16/6.PNG)

Similarly, we can also find the delegate class of this app.

![7]({{site.baseurl}}/images/posts/ios16/7.png)

We can also set the application icon badge number. In this case, let us set it to 9000.

![8]({{site.baseurl}}/images/posts/ios16/8.png)

And it works.

![9]({{site.baseurl}}/images/posts/ios16/9.png)

Since this is exactly similar as having a cycript console, we can enter javascript code as well or any other command from Cycript's documentation. Here is a command i entered from the [Cycript tricks](http://iphonedevwiki.net/index.php/Cycript_Tricks) page.

![10]({{site.baseurl}}/images/posts/ios16/10.png)

Similarly, i can create a function using both Objective-C and javascript syntax. If you are not following cycript here, please refer to the earlier parts on this series that talk about Cycript and its usage in detail.

![11]({{site.baseurl}}/images/posts/ios16/11.png)

I can then use that method whenever i like.

![12]({{site.baseurl}}/images/posts/ios16/12.png)

In part 9 on this series, we had discussed about an application named Snoop-it. iNalyzer is very similar to Snoop-it. However both have their advantages and disadvantages. At the time of writing of the article on Snoop-it, it didn't allow for method swizzling, whereas iNalyzer does. Similarly, iNalyzer doesn't allow us to monitor api calls whereas Snoop-it does have that feature. Hence, both these applications have their pros and cons.

**Conclusion**

In this article, we looked at looked at how we can leverage the power of iNalyzer to perform runtime analysis of iOS applications. iNalyzer is a great tool in the arsenal for anyone interested in learning iOS application security as it makes our task much more easier and efficient.

**References**

*   iNalyzer  
    [https://appsec-labs.com/iNalyzer](https://appsec-labs.com/iNalyzer)
