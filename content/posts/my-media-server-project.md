+++
date = '2024-11-10T09:59:46-06:00'
draft = false
title = 'My Media Server Project'
description = "My journey to set up a media server my whole family could use."
toc = false
readTime = true
autonumber = true
math = false
tags = ["jellyfin", "caddy", "nextcloud", "syncthing"]
showTags = true
hideBackToTop = false
+++
Around 10 months ago I embarked on a project to create a media server for my family to use for our shared videos, photos and music.  I had the following goals:
1. Use my existing home computers as much as possible
2. Get away from using proprietary apps/SAAS (Plex, Dropbox, Amazon Photos, etc.)
2. Make it easy for everyone in the family to use on various devices
3. Synchronize the content on multiple computers

I'm happy to report that the project was a success and has worked even better than I thought it would, with only a few hiccups along the way.

The primary server software I'm using is: 
* [Jellyfin](https://jellyfin.org/) for the media server itself
* [Caddy](https://caddyserver.com/) to provide SSL certs for Jellyfin
* [Nextcloud Server](https://nextcloud.com/install/) for uploading new photos & videos
* [Nextcloud Desktop](https://nextcloud.com/install/) to download uploaded images
* [Phockup](https://github.com/ivandokov/phockup) in scripts to sort images & videos into folders by date
* [Syncthing](https://syncthing.net/) to synchronize/back up content between computers

For client software my family is using:
* Jellyfin iOS and FireTV apps for photos & video streaming
* Finamp iOS app for music streaming
* Nextcloud iOS app for uploading photos & videos from our phones

The basic idea is: we use the Nextcloud mobile app's built in Camera Uploads feature to upload media from our phones.  The Nextcloud Desktop app on one of my computers downoads those videos and uses a script that calls Phockup to sort that media into folders based on the date the videos or photos were taken.  Syncthing is synchronizing the media between our various computers so that we can add media to any one of them and it will automatically sync to the others, including the computer running the Jellyfin server.


I'm planning a series of blog posts describing how I set this all up, and will link to each article as I have it completed:
* Setting Up Syncthing 
* Setting Up Nextcloud for Media Uploads
* Configuring Jellyfin and Caddy