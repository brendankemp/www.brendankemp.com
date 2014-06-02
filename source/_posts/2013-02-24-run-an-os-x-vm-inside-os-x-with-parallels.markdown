---
layout: post
title: "Run an OS X Virtual Machine inside OS X with Parallels"
date: 2013-02-24 19:32
comments: true
categories: osx vm workflow
---

I was able to do this with Parallels 8. There was some conflicting information on the forums, and no clear answer, so I thought I would write it up.
It's easy to do, but not obvious from the Parallels interface or help:

### Get a disk image of the OS X installer

With the new versions of OS X, you need to find the disk image inside the installer you download from the App Store.

1. Download the installer from the App Store
2. Right click the installer and select 'Show Package Contents':
    
  ![Install OS X > Show Package Contents](/images/posts/install-osx-show-package-contents.png)

3. Inside 'Contents', go to 'SharedSupport' > 'InstallESD.dmg'
    
  ![Contents > SharedSupport > InstallESD.dmg](/images/posts/install-osx-contents-installesd.png)

  This is the OS X Installer disk image. Copy it to some place you can find it easily.


### Create a new VM in Parallels and select "Install Windows or another OS from DVD or image file"

1. File > New
2. Select 'Install Windows or another OS from DVD or image file'
3. Select the OS X installer disk image that you just found and press continue

### Go through the regular OS X installation process select the 'Macintosh OSX' HD

The normal OS X installation process boots up in a new window.
  
1. Select your language
2. Select Reinstall OS X
3. Press 'Continue', 'Agree', and 'I agree'
4. Select 'Macintosh HD' for the target install disk

  {% img /images/posts/install-osx-macintosh-hd.png 600 1000 Install on Macintosh HD %}  

  This one scared me a little—it looks like it is going to force to overwrite your current Macintosh HD. 
  In fact, it is on a virtual disk, so you can install there with no problem.