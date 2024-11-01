---
title: "Renewing SPSS for Mac License Silently Update"
date: 2017-10-26T07:16:06-05:00
draft: false
tags: [SPSS, macOS]
---

I previously wrote about a [script to license SPSS for Mac silently]({{< ref "renewing-spss-for-mac-license-silently" >}} "Link").  In version 25 of SPSS Statistics, the path to the bundled Java has changed. It now no longer has the version of the JRE in the path. As a result the script now looks like this :

```
#!/bin/bash -x
 
VERSION=25
AUTHCODE=<redacted>
 
APPPATH="/Applications/IBM/SPSS/Statistics/$VERSION/SPSSStatistics.app/"
cd "$APPPATH/Contents/bin"
 
"$APPPATH/Contents/JRE/bin/java" -jar "$APPPATH/Contents/bin/licenseactivator.jar" SILENTMODE CODES=$AUTHCODE
```
If SPSS keeps a consistent path to the bundled Java, it will make it a little bit easier to update the script for each new version of SPSS Statistics.