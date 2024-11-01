---
title: "Packaging a LaunchAgent Script With the Luggage"
date: 2012-10-23T07:12:14-05:00
draft: false
tags: [macOS, Luggage, Packaging]
---

Previously, I showed how to [install]({{< ref "20120524-getting-started-with-the-luggage" >}}) Luggage, and how to [package a drag and drop app]({{< ref "20120814-packaging-a-drag-and-drop-application" >}}).  In this installment, we will look at how to package up a LaunchAgent script.

We will work with a simple script to mount a couple of file shares when a user logs into their computer.  To set it up as a LaunchAgent, we need two pieces.  First, the script itself (connectshares.sh) needs to be copied to /Library/Scripts/Myorg and everyone should have read and execute permissions.  Second, a plist (com.myorg.connectshares.plist) file which controls the execution of the script needs to be copied to /Library/LaunchAgents and and everyone should have read permissions.

So lets look at our Makefile.  We start with the basics that we need for every Makefile :

```
include /usr/local/share/luggage/luggage.make
TITLE=setup_connectshares
VERSION=1.0
REVERSE_DOMAIN=com.myorg
PAYLOAD=\
```

Now we need to get to the important part. We need to specify the payload stanzas to specify where the source files should be put when installation occurs. First, the plist file :

```
pack-Library-LaunchAgents-com.myorg.connectshares.plist
```

This one is easy, because Luggage has a built in rule for the destination /Library/LaunchAgents

The next part of the Makefile is going to be a little more complex as we want to copy our script file to a custom location.

```
1_Myorg:    l_Library
    @sudo mkdir -p ${WORK_D}/Library/Scripts/Myorg
    @sudo chown root:wheel ${WORK_D}/Library/Scripts/Myorg
    @sudo chmod 755 ${WORK_D}/Library/Scripts/Myorg
pack-connectshares.sh:  1_Myorg
    @sudo ${CP} ./connectshares.sh ${WORK_D}/Library/Scripts/Myorg/connectshares.sh
    @sudo chown root:wheel ${WORK_D}/Library/Scripts/Myorg/connectshares.sh
    @sudo chmod 755 ${WORK_D}/Library/Scripts/Myorg/connectshares.sh
```

The first part, 1_Myorg tells Luggage how to build the directory structure and what the permissions should be. We then tell Luggage that the file connectshares.sh is to be copied to that directory with proper permissions. The complete Makefile will look like this :

```
include /usr/local/share/luggage/luggage.make
 
TITLE=setup_connectshares
VERSION=1.0
REVERSE_DOMAIN=com.myorg
PAYLOAD=\
    pack-Library-LaunchAgents-com.myorg.connectshares.plist\
    pack-connectshares.sh
 
1_Myorg:    l_Library
    @sudo mkdir -p ${WORK_D}/Library/Scripts/Myorg
    @sudo chown root:wheel ${WORK_D}/Library/Scripts/Myorg
    @sudo chmod 755 ${WORK_D}/Library/Scripts/Myorg
pack-connectshares.sh:  1_Myorg
    @sudo ${CP} ./connectshares.sh ${WORK_D}/Library/Scripts/Myorg/connectshares.sh
    @sudo chown root:wheel ${WORK_D}/Library/Scripts/Myorg/connectshares.sh
    @sudo chmod 755 ${WORK_D}/Library/Scripts/Myorg/connectshares.sh
```

Place this Makefile in a directory with connectshares.sh and com.myorg.connectshares.plist and you can build your package.