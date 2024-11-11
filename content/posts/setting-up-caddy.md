+++
title = 'Setting Up Caddy'
date = 2024-11-10T17:27:05-06:00
draft = false
description = "Configuring Caddy to use Jellyfin simply & securely."
summary = "Configuring Caddy to use Jellyfin simply & securely."
toc = true
readTime = true
autonumber = false
math = false
tags = ["jellyfin", "caddy", "my-media-server-project"]
showTags = true
hideBackToTop = false
+++
## The Goal
As part of [My Media Server Project](/posts/my-media-server-project/) I wanted to make logging in and using the server as easy as possible for my family.  So I didn't want anyone to have to worry about what port number the server is running on or trying to remember a complicated URL.  That meant setting up a reverse proxy service, a domain name, and some port forwarding on my home router.

The goal was to create a URL like `https://jellyfin.zoromski.com` and have my family members only have to remember that, and no port numbers.

## Caddy as Reverse Proxy
In looking around at my options, I found [Caddy](https://caddyserver.com/), which not only allows you to set up a reverse proxy but *also* handles all the SSL certificate generation and renewals.  It was perfect for what I want.  

Set up was initially pretty straightforward, just following the [Caddy installation doc](https://caddyserver.com/docs/install#install).  I installed it directly on my old laptop running Ubuntu (the same one that I installed Jellyfin on in the [Setting Up Jellyfin](/posts/setting-up-jellyfin) article).  

The real trick was figuring out what is supposed to be in the `/etc/caddy/Caddyfile` -- the main configuration file for Caddy -- to make it point to the Jellyfin server running on the same machine.  After investigation and some trial and error, this is what I ended up setting in the `Caddyfile` (note: this is the *entire* configuration file):
```
jellyfin.zoromski.com {
    reverse_proxy localhost:8096
}
```
Super simple, it just looks for any traffic that is coming in looking for `jellyfin.zoromski.com` and redirects it to the Jellyfin server on the localhost, running on port 8096.

## Additional DNS and NAT Forwarding Setup

I then created a DNS A record for that hostname that points to my home IP address.  (Technically, I created an A + Dynamic DNS Record in the Advanced DNS for my domain on namecheap.com, which I have set up to dynamically update if my home IP address ever changes.)  

To finish the setup, I then logged onto the admin for my home router, went to Advanced Settings/NAT Forwarding/Port Forwarding.  I created two rules, one for port 80 (standard HTTP traffic) and one for port 443 (HTTPS traffic) and told all of that traffic to be forwarded to the internal IP for the computer I have Jellyfin & Caddy running on.

The way this is set, I can add new host name A records (or change my existing host name records) to my DNS set-up and as long as I also add them to the `Caddyfile` that traffic will be redirected to whatever port I want.