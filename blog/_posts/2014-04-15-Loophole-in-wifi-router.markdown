---
layout: post
title: Loophole in WIFI routers
published: true
tags: WIFI security hacking router loophole SSID
---

>*DISCLAIMER*: This post is meant only for educational and demonstrative purposes
>and should not be used in unethical ways, and secondly I cannot be held responsible
>for any issues that arise in your lives if you follow this post. You cannot
>bind me into any legal matter regarding the information and knowledge given below.
>If you try to do so, you'll be quashed and will have to pay false infamy charges.

Recently I found a loophole in WIFI router's security. In this post, I will discuss this loophole and how to harden the security.

### Okay, Lets begin with SSID. What is SSID?

WIFI router broadcasts  wireless signal carrying the name of your network. This
name is known as SSID(**S**ervice **S**et **ID**entifier) of your router. It
allows other people to connect wireless to your network with ease. So if you
have an SSID that is called _truck_ and your neighbor's SSID is called _scooter_,
people can connect to either the _truck_ or _scooter_ wireless network. This
broadcast SSID feature can also be disabled to create hidden network.

Now a days WIFI routers can host multiple SSIDs(**S**ervice **S**et **ID**entifier).
And if only a year ago that could seem as a router/firmware limitation.

How could I find that my router is capable to host multiple SSID ?
You can find it under Wireless section of your router's configuration page.(Generally http://192.168.1.1)

[![SSID Setting](/images/ssid_setting.png =640x)](/images/ssid_setting.png)

### Got it, but where is the deal?
In some routers like binatone provided with Airtel broadband, Can host multiple
SSID but this feature is limited by firmware. You can see the SSID and passkey
for SSID index 2 on WIFI setting page of your binatone routers. The default SSID
is "Management" and passkey is "0987654321". Important thing is the SSID is in
hidden mode. Just connect this hidden network by adding hidden netwok into your
OS(Please Google it if you don't know how to connect hidden wifi network).

Okay great, I am connected. Now what ?
If you are connected. You have been asigneed an IP address. Get IP address and gateway
address using `ipconfig` command in your CMD terminal. Assume that  you got `192.168.1.10` IP
and your gateway is`192.168.1.1`

Open a web browser and open site "http://192.168.1.1", Enter "admin" as username and
"password" as password. You will see the restricted user interface of the router. If you
get this page then rest of the things is easy. Search on google to get the default username and password.

Now try to open "http://192.168.1.1/basic/home_wlan.htm". You will see wlan configuration
page. Scroll down and copy the passkey.

[![SSID Setting](/images/ssid_password.png =640x)](/images/ssid_password.png)

Connect to the wifi with the passkey and you are done.

That's all !!

### Holy Crap!!! My router has this limitation. How can I improve security of my WIFI?

1. First of all reset the router with factory restore image.
   (Why? To delete any other admin user/scripts created by the hacker on your router.)

2. Change the password of your admin user. Make it strong with at least 8 character long.

3. Choose WPA/WPA2 as your wifi security option.

4. Choose your passkey at least 10 character long including letters, digits and symbols. Don't use your SSID, mobile number or any dictionary words as your passkey

5. Randomly check your router's interface to find connected users' information.

6. By Default your router is set for 192.168.1.* LAN. Change it to some other range. For example 192.168.35.* or 10.1.24.*.

7. If you are PRO user and want to play with your router, You can try free open-source firmwares like [DD-WRT](http://www.dd-wrt.com).
