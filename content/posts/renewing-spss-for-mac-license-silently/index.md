---
title: "Renewing SPSS for Mac License Silently"
date: 2015-09-28T07:14:44-05:00
draft: false
tags: [macOS, SPSS, Munki]
---

It’s not hard to find discussions online about installing SPSS silently. I have a working install based on this [discussion](https://groups.google.com/forum/#!searchin/munki-dev/SPSS/munki-dev/qySQdMNEItU/uWmQZLpdup8J). I needed to re-license a number of machines with a new authorization code and had a bit of trouble, so I thought I’d write it up…

I built a payload free package with The Luggage to distribute with Munki, and my first attempt at the postinstall script looked like this :

```
#!/bin/bash -x
 
VERSION=23
ACTIVATORPATH="/Applications/IBM/SPSS/Statistics/$VERSION/SPSSStatistics.app/Contents/bin/"
AUTHCODE=REDACTED
 
cd $ACTIVATORPATH
/usr/bin/java -jar "$ACTIVATORPATH/licenseactivator.jar" SILENTMODE CODES=$AUTHCODE

```

This worked to activate SPSS, but the script was failing if no user was logged in. I opened a ticket with IBM support and included a copy of my script and the relevant portion of install.log. After being asked for my authcode (which was already in the script I supplied) and being told the code was good (which I already knew) I was passed off to someone else an received this reply :

> The ‘java.lang.InternalError: Can’t connect to window server – not enough permissions’ message included in the failure message you provide is not unique to IBM SPSS.
>
> I found multiple references to the message among several web sites. The propse solution was ‘Add JVM options -Djava.awt.headless=true’. 

The -Djava.awt.headless=true looked familiar from my SPSS installation scripts like the ones in the Munki-Dev thread I linked above. On my second try, I landed on this script :

```
#!/bin/bash -x
 
VERSION=23
ACTIVATORPATH="/Applications/IBM/SPSS/Statistics/$VERSION/SPSSStatistics.app/Contents/bin/"
AUTHCODE=REDACTED
 
cd $ACTIVATORPATH
/usr/bin/java -Djava.awt.headless=true -jar "$ACTIVATORPATH/licenseactivator.jar" SILENTMODE CODES=$AUTHCODE
```

This appears to be working fine even with no logged in user. I simply set this package as an update to SPSS and the clients with SPSS installed updated their license.

#### Update
After the response I got from SPSS above, I replied requesting a sample command that would work. Here is the response :

>I think the problem may be due to the invocation of ‘licenseactivator.jar’ instead of ‘licenseactivator.exe’.
>
>Our documentation for unattended license updating provides the following example:
>
>licenseactivator authcode1[:authcode2:…:authcodeN] [PROXYHOST=proxy-hostname][PROXYPORT=proxy-port-number][PROXYUSER=proxy-userid] [PROXYPASS=proxy-password]
>
>Notice that the command example refers to ‘licenseactivator’ (which is the same as ‘licenseactivator.exe’) rather than ‘licenseactivator.jar’. Since ‘licenseactivator’/’licenseactivator.exe’ is not a Java Runtime file (i.e. does not have a ‘jar’ extension), ‘licenseactivator’/’licenseactivator.exe’ does not require Java to run. Since the error message presented in the error logs (‘java.lang.InternalError’) contains elements specifically referring to Java, the cause of the behavior may be due to (a) the failure of Java to launch when no user is logged in and (b) the invocation of Java due to the use of ‘licenseactivator.jar’ instead of ‘licenseactivator’.
>
>Try replacing ‘licenseactivator.jar’ with ‘licenseactivator’ (or possibly ‘licenseactivator.exe’) and see if this resolves the issue.

So now he is referencing some documentation (why didn’t he refer to that earlier?) that suggests we should be using bin/licenseactivator. So we could try a script like this :

```
#!/bin/bash -x
 
VERSION=23
ACTIVATORPATH="/Applications/IBM/SPSS/Statistics/$VERSION/SPSSStatistics.app/Contents/bin/"
AUTHCODE=REDACTED
 
"$ACTIVATORPATH/licenseactivator" $AUTHCODE
```

This appears to work fine, even without a console user logged in.

#### Update 2

The script and a Luggage Makefile can be found here : 

https://github.com/vmiller/SPSS_License