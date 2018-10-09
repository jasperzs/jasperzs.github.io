---
layout:     post
title:      What is Proxy Server?
subtitle:   Understanding how proxy servers work
date:       2018-10-29
author:     Mr.Humorous 🥘
header-img: img/network/header.jpg
catalog: true
tags:
    - Proxy Server
    - Network
---

## 1. Overview
![How Proxy Servers Work Illustration]({{ site.url }}/img/network/proxyServer/how-proxies-work.png)

You may not know it, but every time you reach out to a website or connect with anyone online, your online connection gives your computer "address" to the site/person you're connecting with.

Why? So that the other end knows how to send information (a Web page, email, etc.) back to your computer. That address is your public IP address. IP stands for Internet Protocol.

Without an IP address, you wouldn't be able to do any Internet/online activity and others online wouldn't be able to reach you. It is how you connect to the world.

## 2. Where does your IP address come from?
Internet Service Providers (e.g. Bell and Rogers) provide IP address at home and as well as Internet connection. Smart device also uses an IP address when you're browsing the web or using an app.

There are few realities about public IP addresses:
+ Your IP address identifies where you are in the world, sometimes to the street level.
+ It can be used by websites to block you from accessing their content.
+ It ultimately ties your name and home address to your IP address, because someone is paying for an Internet connection at a specific location.

Fortunately, there are few ways you can get around those realities, and one of them is to use a proxy service/server (people simply say "proxy").

## 3. Proxy means "substitute."
A proxy lets you go online under a different IP address identity.

You don't change your Internet provider; you simple go online and search for "free proxies" or "list of proxies" and you will get several websites that provide lists of free proxies.

## 4. How a proxy operates.
A proxy server is a computer on the web that redirects your web browsing activity. Here's what that means.
+ Normally, when you type in a website name (Amazon.com or any other), your Internet Service Provider (ISP) makes the request for you and connects you with the destination—and reveals your real IP address, as mentioned before.
+ When you use a proxy your online requests get rerouted.
+ While using a proxy, your Internet request goes from your computer to your ISP as usual, but then gets sent to the proxy server, and then to the website/destination. Along the way, the proxy uses the IP address you chose in your setup, masking your real IP address.

## 5. Why you might want to use a proxy.
+ A school or local library blocks access to certain websites and a student wants to get around that.
+ You want to look at something online that interests you, but you would prefer it couldn't be traced back to your IP address and your location.
+ You're traveling abroad and the technology set up in the country you're in prevents you from connecting to a website back home.
+ You want to post comments on websites but you do not want your IP address to be identified or your identity tracked down.
+ Your employer blocks access to social media or other sites and you'd like to bypass those restrictions.

## 6. Why you might not want to use one
You should keep in mind that your employer, your ISP and other networks might object to you using a proxy. Just because you can do it, doesn't mean you should. And in some cases, websites will blacklist IP addresses they suspect or know are from a proxy.

## 7. Not all proxies are alike.
Even though all proxies help you access websites you might not otherwise get to, not all proxies behave the same way. A proxy can fall into one of four categories:
+ __Transparent proxy__. It tells websites that it is a proxy server and it will pass along your IP address anyway.
+ __Anonymous proxy__. It will identify itself as a proxy, but it won't pass your IP address to the website.
+ __Distorting proxy__. It passes along an incorrect IP address for you, while identifying itself as a proxy.
+ __High Anonymity proxy__. The proxy and your IP address stay a secret. The website just sees a random IP address connecting to it that isn't yours.
