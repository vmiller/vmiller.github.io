---
title: "More on Displaying the Bitlocker Wizard With Windows 8 and MDT 2012 U1"
date: 2013-05-11T07:13:30-05:00
draft: false
tags: [MDT, Windows, Bitlocker]
---

I a previous [post]({{< ref "displaying-the-bitlocker-wizard-pane-with-windows-8-pro-and-mdt-2012-update-1" >}}), I detailed how to get the Bitlocker wizard page to appear when deploying Windows 8 pro.

I recently confirmed another case where the wizard does not show up. I became aware of this [thread on Technet](http://social.technet.microsoft.com/Forums/en-US/w8itproinstall/thread/d56c8985-f763-4aea-bbc6-dd871b9f5a5e) because another user linked to my previously mentioned blog post.

I was able to confirm after a bit of testing, that when using the Windows 8 Enterprise evaluation media, MDT does not show the Bitlocker wizard page. At this point, I am not sure of the reason for this.
