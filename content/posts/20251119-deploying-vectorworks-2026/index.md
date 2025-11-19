---
title: "Deploying Vectorworks 2026"
date: 2025-11-19T09:48:25-05:00
draft: false
tags: [Vectorworks, macOS]
---
I was happy to see Vectorworks shipping a package installer.  I did run into issues getting it to work as documented [here](https://forum.vectorworks.net/index.php?/articles.html/articles/how-to/installation/command-line-installation-of-vectorworks-2026-r945/)


I created a vw2026.plist alongside the installer package and here is what I observed: 

When no user is specified in the vw2026.plist file, no serial number preference is written anywhere.  

When spcifying a user in the plist file, the serial number is written to `<userhome>/Library/Preferences/net.nemetschek.vectorworks.license.2026.plist` 

Specifying `root` as a user results in the serial number being written to `/Library/Preferences`.  

In all cases, launching Vectorworks results in a message that no valid serial numbers have been found.

Contents of net.nemetschek.vectorworks.license.2026.plist :
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>NNA User License</key>
	<string>EMXDDM-XXXXXX-XXXXXX-LLLLLL</string>
</dict>
</plist>
```

I was unable to find any combination where the package installer correctly wrote the serial number in a way that allowed Vectorworks to recognize it.


## Getting this to work

Installing Vectorworks manually with the install manager app I see that the installer wrote the serial number to `<userhome>/Library/Preferences/net.nemetschek.vectorworks.2026.plist`  Note that is a different domain than the pkg installer

Contents of net.nemetschek.vectorworks.2026.plist :
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>NNA Licenses</key>
	<array>
		<string>EMXDDM-XXXXXX-XXXXXX-LLLLLL</string>
	</array>
</dict>
</plist>
```
In addition to the domain being different, I notice that the key is different "NNA Licenses" vs "NNA User License"



To get this all working, I just needed to deploy a correct `net.nemetschek.vectorworks.2026.plist` to `/Library/Preferences` post installation.  I am already deploying a `LoginDialog.xml` to point to our license server, so it's not much extra work to also deploy the serial number preference.  It just would have been nice to have accurate documentation to save me some work.

I hope writing this unofficial documentation helps others.