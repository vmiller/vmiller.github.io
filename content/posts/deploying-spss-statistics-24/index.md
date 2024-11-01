---
title: "Deploying SPSS Statistics 24"
date: 2016-10-06T07:15:01-05:00
draft: false
tags: [macOS, Munki, SPSS]
---

I previously posted a method for scripting the activation of SPSS Statistics 23. My method has changed for version 24, so I thought it would be a good idea to write up my notes.

First up, repackaging. Due to the SPSS silent installer being notoriously bad, I had already decided I would just repackage version 24 and use a postinstall script to activate. It really is the best way to ensure that it will deploy properly. To add to this, I discovered that SPSS has its own Java bundled so that it does not rely on having the legacy system Java installed. The silent installer you download from IBM, though, still needs the system Java. Repackaging means I have one less dependency on the legacy system java.

Installing SPSS onto a clean test machine, I was able to see that it places files both under /Applications and /Library/Application Support. The files under Applications follow the usual convention of previous versions of SPSS, /Applications/IBM/SPSS/Statistics/24 The files dropped in Library follow a similar convention : /Library/Application Support/IBM/SPSS/Statistics/24 To repackage, grab both of these locations. Note that is it required that users have write access to /Library/Application
Support/IBM/SPSS/Statistics so you’ll need to make sure those permissions get set properly in your package.

I use the Luggage for packaging, and here is my Makefile :

```
include /usr/local/share/luggage/luggage.make
TITLE=SPSS
PACKAGE_VERSION=24
REVERSE_DOMAIN=org.domain
PAYLOAD=pack-from-applications-IBM \
    pack-Library-Application-Support-IBM \
    pack-script-pb-postinstall
 
pack-Library-Application-Support-IBM:   l_Library_Application_Support
    @sudo ${CP} -R /Library/Application\ Support/IBM ${WORK_D}/Library/Application\ Support/
    @sudo find ${WORK_D}/Library/Application\ Support/IBM/SPSS/Statistics -type d -print0 | sudo xargs -0 chmod 777
    @sudo find ${WORK_D}/Library/Application\ Support/IBM/SPSS/Statistics -type f -print0 | sudo xargs -0 chmod 666
```

I do the license activation in a postinstall script. The basis for the syntax of the script came from a sample script that SPSS ships with their program. The sample script is at /Applications/IBM/SPSS/Statistics/24/SPSSStatistics.app/Contents/bin/licsilent.sh Here is what my postinstall script looks like :

```
#!/bin/bash -x
 
VERSION=24
AUTHCODE=<redacted>
 
APPPATH="/Applications/IBM/SPSS/Statistics/$VERSION/SPSSStatistics.app/"
cd "$APPPATH/Contents/bin"
 
"$APPPATH/Contents/PlugIns/jre1.8.0_71.jre/Contents/Home/bin/java" -jar "$APPPATH/Contents/bin/licenseactivator.jar" SILENTMODE CODES=$AUTHCODE
```

If you have made it this far, perhaps you would consider doing me a favor. I have two open RFE requests with IBM. One is for them to ship a proper Apple installer package, and the other is to not require users have write access to the shared /Library/Application Support location. If you have an IBM login, please consider voting up these requests :

[RFE 91634](http://www.ibm.com/developerworks/rfe/execute?use_case=viewRfe&CR_ID=91634)

[RFE 91094](http://www.ibm.com/developerworks/rfe/execute?use_case=viewRfe&CR_ID=91094)