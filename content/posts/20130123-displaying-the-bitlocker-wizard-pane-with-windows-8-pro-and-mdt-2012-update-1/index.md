---
title: "Displaying the Bitlocker Wizard Pane With Windows 8 Pro and MDT 2012 Update 1"
date: 2013-01-23T07:12:34-05:00
draft: false
tags: [Windows, MDT, Bitlocker]
---

Using MDT 2012 Update 1 with ADK, I built and captured a Windows 8 Pro image to enable my institution to more easily do some testing with Windows 8. After setting up a task sequence to deploy this reference image I noticed something unexpected. When choosing the Win 8 task sequence, I was not presented with the Bitlocker wizard pane. The wizard pane was showing up fine for my Win 7 task sequences.

I posed the question on the [MDT-OSD](http://www.myitforum.com/absolutenm/EmailLists.aspx#MDT/OSD) email list, and Michael Niehaus to the rescue! Turns out there is logic in the MDT scripts to determine if the edition of Windows is a “Premium SKU” to determine what features are available. Since I was using Windows 8 Pro instead of Enterprise it was not getting marked as a premium SKU.

Of course, since Win 8 Pro does support Bitlocker (Win 7 Pro did not), this is a bug. A user posted a bug report to Microsoft along with some work around code. If you have a Connect account, you can see the bug report [here](https://connect.microsoft.com/site14/feedback/details/773608/bitlocker-and-windows-8-pro).

The work around :

In ZTIUtility.vbs add the following two lines of code after line 3846 :

```
</p><p></p><p>case "PROFESSIONAL", "PROFESSIONALE", "PROFESSIONALN"</p><p> If Left(oEnvironment.Item("OSCurrentVersion"), 3) &gt;= 6.2 Then IsHighEndSKUEx = True</p><p></p><p>
```

**Update 1-14-2014**

Hat tip to this [Post](http://mdt2012.com/content/windows-81-pro-enable-bitlocker-mdt) on MDT2012.com. This bug still exists in MDT 2013 and Windows 8.1 has been released since the writing of this post. The above lines of code have been changed to allow for Windows 8.1 (changed from equal to greater than or equal).