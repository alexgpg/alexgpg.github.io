---
title: "What HTTPS took from us? Federated CDN"
date: "2024-09-07"
summary: "In the past we can optimize own networks without asking big corps, now we don't."
description: "Article: How CDN-like works on early days of web"
toc: false
readTime: true
autonumber: true
tags: ["cdn", "internet", "internet-history"]
showTags: true
hideBackToTop: false
---

In the early days of the web, when HTTP without SSL (HTTPS) was common, there was a way to speed up
the Internet and reduce traffic if you were a system administrator of a network. You could set up
a cache proxy server like [Squid](https://www.squid-cache.org/) in the network. If a page were in the cache
(after someone visited it), it would load from the local cache server instead of the upstream server,
drastically reducing load time (better for users) and decreasing traffic (better for the network).
It was possible to configure it in transparent mode, so the user didn't even need to configure anything.
They could just use the Internet, and caching would work automatically.

![squid proxy connects 3 computers and pass traffic to the Internet caching it](/squid-proxy-diagram.png "Squid works as Cache Server")

In fact, it's a **CDN node**, but controlled not by the content host, but by any network administrator.
It's like a **distributed federated CDN** with an extremely relaxed policy for participation.
It was a nice feature; you didn't need to be a large service provider to host some vendor's CDN nodes.

Is it secure? Absolutely not. By today's standards, the security of this solution is terrible.

There are problems here: non-encrypted traffic is visible by a network administrator, proxy admin could see
what are you looking at and even change data(like adding an advertisement). Content owners might be not
happy is anyone can just cache their content without notice. So yeah, TLS is needed.

Is it possible now? In practice, no, at least not without
[violating](https://wiki.squid-cache.org/ConfigExamples/Intercept/SslBumpExplicit#intercept-https-connect-messages-with-ssl-bump) security practices.

If you don’t have the key and you aren’t the owner of the domain, you can’t simply send cached data because
you don’t even know what the user is viewing. It is possible to install the provider's self-signed
root certificate, but it's not a good idea.

So now, as an administrator, you can’t optimize your network. _It’s a common trend to remove any ability
to control your network, leading to networks becoming just dark pipes to the cloud._
Centralization of DNS (moving DNS from ISPs) is part of this trend.

I believe that returning it in some form without compromising security would be useful. There is no better
place for a CDN node than right inside your local network. Maybe a P2P solution like
[IPFS](https://ipfs.tech/) could help, but I’m not sure about that.

There’s food for thought here.

