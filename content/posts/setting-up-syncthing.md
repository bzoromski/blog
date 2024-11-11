+++
title = 'Setting Up Syncthing'
date = 2024-11-10T21:41:47-06:00
draft = false
Summary = "Configuring Syncthing to keep files in sync between computers."
description = "Configuring Syncthing to keep files in sync between computers."
toc = true
readTime = true
autonumber = false
math = false
tags = ["syncthing", "my-media-server-project"]
showTags = true
hideBackToTop = false
+++
## Intro
With everything else I did in [My Media Server Project](/posts/my-media-server-project/), I accomplished pretty much everything I set out to do -- my family could easily upload and access our photos and videos from their mobile devices.  But I wanted one last thing: an easy way to add files to our libraries on computers and synchronize folders between them.

## Syncthing
I looked a few different options but quickly latched onto [Syncthing](https://syncthing.net/), a really easy to set up, secure way to synchronize files and folders between multiple computers. 

The [Syncthing Getting Started guide](https://docs.syncthing.net/intro/getting-started.html) is well-written and walks you through how to install the tool and start syncing between computers.  

I found the process for setup and configuring a lot easier on Linux computers.  It installs as a service and you just configure it all using a GUI in your browser at `http://localhost:8384/`.  

On Windows computers the setup process was a little bumpier, and for configuring afterward I found it easiest to also install [SyncTrayzor](https://github.com/canton7/SyncTrayzor).

## Tips
A few things I ran into in my time using Syncthing to sychronize my Jellyfin media folders between computers:
* If you have more than one Jellyfin server running you will want to create a `.stignore` file in the root of each media library with the following:
```
**.nfo
```
This tells Syncthing to not try to synchronize any file with the extension `.nfo` -- I let Jellyfin create those files but they can always be re-created on a Jellyfin media library scan if they aren't there.  But I had them synchronizing and constantly getting conflict errors, so I put in the ignore rule.
* It's best to set up syncing for individual media libraries (e.g., `/media/jellyfin/Pictures` and `/media/jellyfin/Videos`) instead of the overall media directory (e.g., `/media/jellyfin`).  That way if you have some computers that don't need copies of certain libraries, or that have less free space than others you can be selective in which directories get synced to which computers.