---
title: Principle of CDN Acceleration
date: 2021-03-11 00:54:15
tags:
- CDN
---

## What is CDN
The full name of CDN is content delivery network. Its purpose is to add a new layer of cache in the existing Internet to publish the content of the website to the nodes closest to the "edge" of the user's network, so that users can obtain the required content nearby and improve the response speed of users when visiting the website. In terms of technology, it can solve the problems of small network bandwidth, large user visits and uneven network distribution, so as to improve the response speed of users visiting websites.

In short, the working principle of CDN is to cache the resources of your source station to CDN nodes located all over the world. When users request resources, they will return the resources cached on the nodes nearby, instead of requiring each user's request to go back to your source station to obtain them. This can avoid network congestion, relieve the pressure on the origin station, and ensure the speed and experience of users' access to resources

![CDN](https://app.yinxiang.com/shard/s57/res/5cea6e26-eb53-4a5c-a20d-e3a62817d906/CDN_Muvi.png)

The network optimization function of CDN is mainly reflected in the following aspects:

- Solve the "first kilometer" problem on the server side
- The impact caused by the bottleneck of interconnection between different operators is alleviated or even eliminated
- The pressure of export bandwidth of each province has been reduced
- It relieves the pressure of backbone network
- Optimize the distribution of hot content on the Internet

## How CDN works

Traditional access process
![Traditional access process](https://app.yinxiang.com/shard/s57/res/d3007aa3-b61d-4758-8a85-1d42ef712c11/%E6%97%A0%E6%A0%87%E9%A2%98.jpg)

1. The user enters the domain name to visit, and the operating system queries the IP address of the domain name to localdns
2. Local DNS queries root DNS for the authorization server of the domain name (assuming that the local DNS cache has expired)
3. Root DNS responds the DNS record of domain name authorization to local DNS
4. After getting the authorized DNS record of the domain name, the local DNS continues to query the IP address of the domain name from the DNS authorized by the domain name
5. DNS of domain name authorization queries domain name records and responds to local DNS
6. Local DNS will get the domain name IP address and respond to the client
7. After getting the IP address of domain name, the user visits the site server
8. The site server answers the request and returns the content to the client

CND access process
![CND access process](https://app.yinxiang.com/shard/s57/res/a2fd0a90-4efe-4b6c-b4a5-6ac62bd09277/1.jpg)

1. The user enters the domain name to visit, and the operating system queries the IP address of the domain name to localdns
2. Local DNS queries root DNS for the authorization server of the domain name (assuming that the local DNS cache has expired)
3. Root DNS responds the DNS record of domain name authorization to local DNS
4. After getting the authorized DNS record of the domain name, the local DNS continues to query the IP address of the domain name from the DNS authorized by the domain name
5. After DNS queries the domain name record (generally CNAME), it responds to localdns
6. After the local DNS gets the domain name record, it queries the IP address of the domain name from the intelligent dispatch DNS
7. Intelligent scheduling DNS responds the most suitable IP address of CDN node to local DNS according to certain algorithms and strategies (such as static topology, capacity, etc.)
8. Local DNS will get the domain name IP address and respond to the client
9. After getting the IP address of domain name, the user visits the site server
10. The CDN node server answers the request and returns the content to the client. (on the one hand, the cache server saves it locally for future use; on the other hand, it returns the acquired data to the client to complete the data service process)

From the above analysis, we can get that in order to achieve transparent access to ordinary users (the user client does not need to make any settings after using the cache), it is necessary to use DNS to guide users to access the cache server, so as to achieve transparent acceleration service, Therefore, it is the most simple and effective way to guide user access by modifying DNS.

## Elements of CDN network

For ordinary Internet users, each CDN node is equivalent to a web server placed around it. By taking over DNS, users' requests are transparently directed to the nearest node. The CDN server in the node will respond to the user's request like the original server of the website. Because it is closer to the user, the response time must be faster.

The area circled by the dotted line in the above figure is the CDN layer, which is located between the client and the site server.

### CNAME record
CNAME is the alias (canonical name). It can be used to resolve a domain name to another domain name. When the DNS system queries the name on the left side of CNAME, it will turn to the name on the right side of CNAME and query again. It will track down to the last PTR or a name. If the query is successful, it will respond. Otherwise, it will fail.
For example, you have a server that stores a lot of data that you use docs.example.com To access these resources, but also through the documents.example.com If you can also access these resources, you can add a CNAME record to your DNS resolution service provider, and documents.example.com point docs.example.com After the CNAME record is added, all the access documents.exa mple.com Will be transferred to docs.example.com , get the same content.

### CNAME domain name
When accessing the CDN, you will get a CNAME domain name assigned to you by CDN after adding the accelerated domain name on the CDN provider console. You need to add a CNAME record in your DNS resolution service provider to point your accelerated domain name to this CNAME domain name, so that all requests for the domain name will be directed to the CDN node to achieve the acceleration effect.

### DNS
DNS is domain name system, which means domain name resolution service. Its role in the Internet is to convert domain names into IP addresses that can be recognized by the network. People are used to memorizing domain names, but machines only recognize IP addresses. Domain names and IP addresses are one-to-one corresponding. The conversion between them is called domain name resolution. Domain name resolution needs to be completed by a special domain name resolution server. The whole process is automatic. For example: input when surfing the Internet www.baidu.com It will automatically convert to 220.181.112.143.

Common DNS resolution service providers include: alicloud resolution, Wannet resolution, DNSPod, Xinnet resolution, route53 (AWS), dyn, cloudflare, etc.

### Back to source host
The back to source host determines that the back to source request accesses a specific site on the source station.

> Example 1: the origin site is a domain name, and the origin site is a domain name www.a.com The return source host is www.b.com , then the actual return source is the request to www.a.com The resolved IP address is the site on the corresponding host www.b.com

> Example 2: the source station is IP, the source station is 1.1.1.1, and the return source host is www.b.com , then the site on the host corresponding to 1.1.1.1 is actually returned to the source www.b.com

### Agreement back to source
The protocol used when referring back to the source is consistent with that used by the client when accessing the resource. That is, if the client requests the resource in the way of HTTPS, when the resource is not cached on the CDN node, the node will use the same HTTPS method to get the resource; similarly, if the client uses the HTTP protocol, the CDN node will also use the HTTP protocol when returning to the source.