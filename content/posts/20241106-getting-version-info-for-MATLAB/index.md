---
title: "Getting Version Info for MATLAB"
date: 2024-11-06T07:30:00-05:00
draft: false
tags: [macOS, Munki]
---

It can be challenging to get accurate version information for MATLAB on macOS.  The installer is not a package, so no package receipts to rely on.  

The next place we might logically look for version info is in the app bundle itself. 
```
	<key>CFBundleVersion</key>
	<string>24.2.0</string>
```
Well that at least looks promising...  This is an example from an installation of MATLAB 2024b.  The 24.x indicates 2024 and the x.2 indicates `b` version.  The problem with this is that the full version of that app is 24.2.0.2740171.  The fourth number differentates the updates to 2024b. 

So what to do?  If we look in the root of the app bundle folder, we see a file called `VersionInfo.xml`  Here are the contents from the same sample installation : 

```
<?xml version="1.0" encoding="UTF-8"?>
<!-- Version information for MathWorks R2024b Release -->
<MathWorks_version_info>
  <version>24.2.0.2740171</version>
  <release>R2024b</release>
  <description>Update 1</description>
  <date>Sep 20 2024</date>
  <checksum>4024810896</checksum>
</MathWorks_version_info>
```

Now we have the info we need!  The info in the `version` tag is what we need to correctly identify the version installed.  Here is a Python script that makes use of this info.  In my case it is a Munki installcheck_script (to determine if Munki should install or not) :

```
#!/usr/local/bin/managed_python3

import os
import xml.etree.ElementTree as ET

currentVersion = "24.2.0.2740171"
xmlPath = '/Applications/MATLAB_R2024b.app/VersionInfo.xml'

if os.path.isfile(xmlPath):
	tree = ET.parse(xmlPath)
	root = tree.getroot()
	installedVersion = (root[0].text)
else:
	installedVersion = "0"


if installedVersion < currentVersion:
	print("Installation/Upgrade required")
	exit(0)
else:
	print("MATLAB " + currentVersion + " or newer already installed")
	exit(1)
```

You of course may find this info useful for systems other than Munki, and of course there are methods other than Python to get the info as well.  