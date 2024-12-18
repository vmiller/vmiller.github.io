---
title: "Enabling and Disabling SSH in Munki Update"
date: 2024-12-19T04:00:00-05:00
draft: false
tags: [munki, macOS]
---
I previously [posted]({{< ref "20180227-enabling-and-disabling-ssh-using-munki" >}}) a munki nopkg script to enable SSH using `/usr/sbin/systemsetup`  This stopped working on newer versions of macOS at some point and I implemented a new method, but never posted it.  The new method simply loads the launch daemon to enable SSH.

The install_check script 
```
	<key>installcheck_script</key>
	<string>#!/bin/bash

#check to see if ssh is off
(/bin/launchctl list | grep ssh)

if [[ $? = 1 ]] 
then
	# exit 0 to tell Munki an install is needed
	exit 0
fi
# if not needed, exit 1
exit 1</string>
```

The postinstall_script :
```
	<key>postinstall_script</key>
	<string>#!/bin/bash

/bin/launchctl load -w /System/Library/LaunchDaemons/ssh.plist

(/bin/launchctl list | grep ssh)

if [[ $? = 0 ]] 
then
	exit 0
fi

exit 1</string>
```

And finally the uninstall_script :
```
	<key>uninstall_script</key>
	<string>#!/bin/bash

/bin/launchctl unload -w /System/Library/LaunchDaemons/ssh.plist</string>
```