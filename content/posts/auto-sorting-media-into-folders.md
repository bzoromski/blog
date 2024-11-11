+++
title = 'Auto Sorting Media Into Folders'
date = 2024-11-10T20:46:18-06:00
draft = false
Summary = "Setting up scripting to sort photos and videos into date-specific folders with Phockup."
description = "Setting up scripting to sort photos and videos into date-specific folders with Phockup."
toc = true
readTime = true
autonumber = false
math = false
tags = ["phockup", "bash-scripting", "cron-jobs", "my-media-server-project"]
showTags = true
hideBackToTop = false
+++
## Intro
For years my wife and I had been manually sorting our digital photos and videos -- into year folders and then month folders under those -- but with the adoption of this process in [My Media Server Project](/posts/my-media-server-project/) that sorting has all been automated.

## Phockup
Searching around I found a command line utility that does everything I was looking to do: [Phockup](https://github.com/ivandokov/phockup).  You pass it an input directory and an output directory and it then renames and moves the files in whatever name format you provide, based on the creation date information stored in the image or video file.  

I followed the installation instructions on the [Phockup Github page](https://github.com/ivandokov/phockup) to install the tool.  Initially I had set it up as a Docker container running on a Windows machine but everything ran really slowly and it seemed like a lot of overhead for some simple shell scripts so I instead installed Phockup directly on a Ubuntu machine (the same one I have running Jellyfin).

## Scripting
With Phockup handling the renaming and folder creation, all I needed to do was write some very simple shell scripts with the settings/format I wanted everything in.

I ended up creating three shell scripts that I set to run on a schedule:
* `move-media-files.sh`
* `sort-image-files.sh`
* `sort-video-files.sh`

## The Move Media Files Script
The first script, `move-media-files.sh`, has the simple purpose of moving the uploaded files out of the original folder, the synchronized Nextcloud "Camera Uploads" folder.  I do this for a couple reasons... one is that if I move the files out of Nextcloud I am making space on the server for more photos/videos to be uploaded.  Another reason is so my other scripts can deal with a more static list of files and I don't have to worry about new images coming in while my sorting scripts are running.

This script just sets up the folders I'm going to move the files from and to, and runs an `rsync` command to do it.  For my source directory I'm pointing to the "Camera Uploads" directory I set up in the [Nextcloud setup](/posts/setting-up-nextcloud/).  For my destination directory I am pointing it to a "General Uploading" directory in my Jellyfin pictures library.
```
#!/bin/bash

# Set source and destination directories
source_dir="/home/brian/Nextcloud/Camera Uploads"
destination_dir="/media/jellyfin/Pictures/General Uploading"

# Use rsync to move files from source to destination
rsync -a --remove-source-files --timeout=5 -v "$source_dir/" "$destination_dir/"
```

## The Sort Image Files Script
The next script, `sort-image-files.sh`, runs Phockup to sort the images from the "General Uploading" folder that the `move-media-files.sh` script put them in.  

It will sort them into folders named with the year and subfolders with year-month (e.g. 2024/2024-11) -- that's done with a `--date YYYY/YYYY-MM` setting.  A `--file-type=image` setting tells this script to only process the images, not videos -- that's because on my Jellyfin server I have separate libraries set up for my photos and videos so I process them into different media folders.

Here's the whole script:
```
#!/bin/bash

# Set source and destination directories
source_dir="/media/jellyfin/Pictures/General Uploading"
destination_dir="/media/jellyfin/Pictures"

phockup "$source_dir" "$destination_dir" --file-type=image --date YYYY/YYYY-MM --move
```

## The Sort Video Files Script

The last script, `sort-video-files.sh`, is nearly identical to the `sort-image-files.sh` script -- the only difference is it uses a `--file-type=video` parameter and is sorting the files into a video-specific destination folder.

Here's the full script:
```
#!/bin/bash

# Set source and destination directories
source_dir="/media/jellyfin/Pictures/General Uploading"
destination_dir="/media/jellyfin/Videos"

phockup "$source_dir" "$destination_dir" --file-type=video --date YYYY/YYYY-MM --move
```
## Set the Scripts to be Executable
I've set the scripts to be executable by me, by doing the following in the directory they are in:
```
chmod 744 *.sh
```

## Cron Jobs
The last bit of all of this is to set up cron jobs to run these scripts on a regular basis.  

I have the `move-media-files.sh` script running every 15 minutes, the `sort-image-files.sh` script running every 20 minutes, and the `sort-video-files.sh` script running every 25 minutes.

Here's the `crontab`: 
```
*/15 * * * * "/home/brian/Scripts/media/move-media-files.sh"
*/20 * * * * "/home/brian/Scripts/media/sort-image-files.sh"
*/25 * * * * "/home/brian/Scripts/media/sort-video-files.sh"
```