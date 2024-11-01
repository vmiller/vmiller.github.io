---
title: "Problem Updating Deployment Share"
date: 2012-01-19T07:09:00-05:00
draft: false
tags: [Windows, MDT]
---

I have had an issue where I was no longer able to update my deployment share to update my boot images.  If I installed MDT on another computer and connected to the share, I could then do the update, so the problem is not with the share itself but with the computer doing the updating.

I had tried uninstalling WAIK and then reinstalling it, but that did not help.  I did some googling and did not come up with anything either…  I was suspecting that some temp files were a problem, so I started poking around.  When I looked in `Users\username\AppData\Local\Temp` I found some directories named `MDTUpdate.XXX` that were rather large.

For some reason I could not delete these files.  I finally had to boot to a live CD to delete the files.  Once I deleted these files, I could update my Deployment Share again.