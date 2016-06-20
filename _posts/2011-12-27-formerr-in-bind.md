---
layout: post
title: FORMERR in Bind
---

Recently I was configuring Bind 9 on Centos 6.2. The setup was pretty straight forward: caching name server with one authoritative local domain. But the problem was that it was resolving my local authoritative domain but getting SERVFAIL for all external queries! I tried using forwardars, forward only, forward first but nothing helped. "dig @127.0.0.1 yahoo.com" was giving a SERVFAIL and "dig @192.168.1.1 yahoo.com" was working fine.

This problem almost made me crazy and was about to give up on bind until I found the problem. I tried every thing: I was configuring bind on a KVM virtual machine, I thought may be its the Bridge or the KVM's internal network bug. I tried bind on a physical machine, but the problem was same! I tried it on Fedora 14, same problem! Next I tried it on different versions of bind, but the error was the exact same!

Here are the errors I was getting in the named.run logs:

> DNS format error from 128.63.2.53#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 128.63.2.53#53  
> DNS format error from 202.12.27.33#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 202.12.27.33#53  
> DNS format error from 192.5.5.241#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.5.5.241#53  
> DNS format error from 192.36.148.17#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.36.148.17#53  
> DNS format error from 128.8.10.90#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 128.8.10.90#53  
> DNS format error from 193.0.14.129#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 193.0.14.129#53  
> DNS format error from 192.112.36.4#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.112.36.4#53  
> DNS format error from 199.7.83.42#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 199.7.83.42#53  
> DNS format error from 192.33.4.12#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.33.4.12#53  
> DNS format error from 192.203.230.10#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.203.230.10#53  
> DNS format error from 198.41.0.4#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 198.41.0.4#53  
> DNS format error from 192.228.79.201#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.228.79.201#53  
> DNS format error from 192.58.128.30#53 resolving yahoo.com/A for client 127.0.0.1#39224: reply has no answer  
> error (FORMERR) resolving 'yahoo.com/A/IN': 192.58.128.30#53  

So basically bind was getting a malformed response from all the root level servers it was trying in the hint file named.ca

## The Solution:

After enabling the debug mode and monitoring traffic with tcpdump, I found the problem. The culprit was my DSL router (AzTech 605EW). And apparently most home dsl routers will behave abnormally with large udp packets. The large udp packets are because bind uses EDNS when querying other DNS servers.

Adding the following configuration in named.conf disabled edns and solved the problem!

>server ::/0 { edns no; };  
>server 0.0.0.0/0 { edns no; };
