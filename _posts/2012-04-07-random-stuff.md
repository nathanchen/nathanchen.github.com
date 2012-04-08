---
layout: post
title: "Random Stuff"
category: 
tags: []
---
{% include JB/setup %}


	##One
	R. Paul, Exclusive: a behind-the-scenes look at Facebook release engineering

Moving a 1.5GB binary blob to countless servers is a non-trivial technical challenge. After exploring several solutions, Facebook came up with the idea of using BitTorrent, the popular peer-to-peer filesharing protocol. BitTorrent is very good at propagating large files over a large number of different servers.

Rossi explained that Facebook created its own custom BitTorrent tracker, which is designed so that individual servers in Facebook's infrastructure will try to obtain slices from other servers that are on the same node or rake, thus reducing total latency.

Many external resources are referenced from Facebook pages, including JavaScript, CSS, and graphical assets. Those files are hosted on geographically distributed content delivery networks(CDNs).

