+++
title = 'Setting Up Jellyfin'
date = 2024-11-10T14:48:58-06:00
draft = false   
summary = "Configuring Jellyfin and directories for its media libraries."
description = "Configuring Jellyfin and directories for its media libraries."
toc = true
readTime = true
autonumber = false
math = false
tags = ["jellyfin","jellyman","bash-scripting", "my-media-server-project"]
showTags = true
hideBackToTop = false
+++
## Installing Jellyfin & Jellyman
Setting up Jellyfin is pretty straightforward -- for [My Media Server Project](/posts/my-media-server-project/) originally I set it up on an old laptop running Ubuntu by just following the install instructions on the [Jellyfin Downloads](https://jellyfin.org/downloads/server) page.

I became a little worried about how easily it would be to back things up and port my server over to another computer (say, if the old laptop I installed it on decides to die).  That sent me looking around for something that would automate a bunch of things -- installing Jellyfin, backing it up, restoring to a new computer, etc.  My search led me to [Jellyman](https://github.com/Smiley-McSmiles/jellyman), the Jellyfin Manager.  

[Jellyman](https://github.com/Smiley-McSmiles/jellyman) automates a lot of things for me, like downloading the latest version of Jellyfin (`jellyman -vd`) and switching to it (or switching back to an older version, `jellyman -vs`), backing up Jellyfin and its database (`jellyman -b`), and just having a handy way to restart Jellyman (`jellyman -r`).

## Jellyfin Libraries
When setting up my content libraries in Jellyfin I made a conscious decision to put them in a directory structure that both I and the `jellyfin` user have read/write access to.  Through trial and error I found that the parent directory (in my case `/media/jellyfin`) the `jellyfin` user needs to have read/write access to that directory, as well as any directories under that.  (If you don't give that ser read/write access, things *seem* to work but new media that you add will end up giving errors at some point -- like videos won't transcode and thumbnail images don't get created.)

### Group Access
If you set Jellyfin up with defaults, a user named `jellyfin` gets created, along with a group called `jellyfin` with just that user in it.  The files I'm sharing are going to be owned by my user account (`brian`) so need a group that I could use it for file & folder permissions. I wrote a little script that adds my user account to that `jellyfin` group and kept it so I can re-run it on any computer I end up installing Jellyfin on and put myself and the `jellyfin` user in it: 
```
#!/bin/bash
# make sure my user is in jellyfin group
username="brian"
groupname="jellyfin"

# Check if the user is already in the group
if ! groups "$username" | grep -q "\<$groupname\>"; then
    # If not, add the user to the group
    echo "Adding $username to group $groupname"
    sudo usermod -aG "$groupname" "$username"
    echo "User $username added to group $groupname"
else
    echo "User $username is already in group $groupname"
fi
```

So the setup of my Jellyfin media libraries has me creating the `/media/jellyfin` directory, and then setting permissions on it so I own it with `jellyfin` as the group, like this: 
```
# Set permission & ownership for parent directory 
sudo chown brian:jellyfin /media/jellyfin
sudo chmod 775 /media/jellyfin
```
### Individual Libary Directory Permissions
Each directory I create under there I just need to make sure they also have the same group ownership and file permissions.  Because I found that my file permissions occasionally get set incorrectly, I ended up creating a bash script that I can run anytime I set up Jellyfin on a new computer or just need to fix the permissions. Here is the full script:
```
#!/bin/bash

# Define the directory path
media_directory="/media/jellyfin"

# make sure user brian is in jellyfin group
username="brian"
groupname="jellyfin"

# Check if the user is already in the group
if ! groups "$username" | grep -q "\<$groupname\>"; then
    # If not, add the user to the group
    echo "Adding $username to group $groupname"
    sudo usermod -aG "$groupname" "$username"
    echo "User $username added to group $groupname"
else
    echo "User $username is already in group $groupname"
fi

# Check if the media directory exists
if [ ! -d "$media_directory" ]; then
    # If the directory doesn't exist, create it and set permissions
    sudo mkdir -p "$media_directory"
    sudo chown $username:$groupname "$media_directory"
    sudo chmod 775 "$media_directory"
    echo "Directory created: $media_directory"
else
    echo "Directory already exists: $media_directory"
fi

# Set permission & ownership for parent directory 
sudo chown $username:$groupname $media_directory
sudo chmod 775 $media_directory

cd $media_directory
# Loop through the directories to set permissions and ownership
for destination_dir in */ ; do
   echo "destination_dir: ${destination_dir}"
   dir_name="$media_directory/"$(basename "$destination_dir")
   echo "setting ownership"
   sudo chown $username:$groupname -R "$destination_dir"
   sudo chown $username:$groupname "${dir_name}"
   echo "changing file permissions"
   sudo find "${destination_dir}" \( -type f -exec chmod ug=rw,o=r   {} + \) -o \
   \( -type d -exec chmod ug=rwx,o=rx {} + \)
#    read -p "Press Enter to continue" </dev/tty
done

echo "Setup completed for all directories."
```

