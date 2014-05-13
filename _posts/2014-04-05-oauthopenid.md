---
layout: post
title: "OAUTH和OPENID"
category: Reading Notes
tags: ["读文章", “算法”]
---
{% include JB/setup %}

To start with, here's what OAuth does have in common with OpenID:

- They both live in the general domain of security, identity and authorization
- They are open web standards. Created and evolved by people with an itch to scratch and evolved pragmatically by a loose, fluid, alliance. 
- They both celebrate decentralization. There is no central Open ID or OAuth server that holds all the security information in the universe.
- They both involve browser redirects from the website you're trying to use - the 'consumer' website - to a distinct 'provider' website. Meanwhile, those websites talk to each other behind the scenes to verify what just happened.
- The user can actively manage the provider website, exerting control over which websites can talk to it and for how long.

Open ID gives you one login for multiple sites. Each time you need to log into a site using Open ID, you will be redirected to your Open ID site where you login, and then back to the site. 

OAuth lets you authorise one website - the consumer - to access your data from another website - the provider. For instance, you want to authorize a printing provider - call it Moo - to grab your photos from Flickr. Moo will redirect you to Flickr which will ask you, for instance, "Moo wants to download your Flickr photos. Is that cool?" and then back to Moo to print your photos.



### Reference

http://softwareas.com/oauth-openid-youre-barking-up-the-wrong-tree-if-you-think-theyre-the-same-thing