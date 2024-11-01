---
title: "Deploying SPSS 27"
date: 2020-08-17T07:16:36-05:00
draft: false
categories: [Mac OS]
tags: [macOS, Munki, SPSS]
---
Good news! As of SPSS 27 IBM is finally shipping an Apple installer package for macOS instead of their Java based silent installer. Looking at the installer, it has a few minor issues but is for the most part sane. The package can be imported into Munki without much fuss.

As in the past, activation will need to be scripted post install to license SPSS. The format of the path to the application has changed a bit, my postinstall script now looks like :

```
#!/bin/sh
 
SERVERPATH="server.myorg.org"
ACTIVATIONPATH="/Applications/IBM SPSS Statistics 27/Resources/Activation"
 
cd "$ACTIVATIONPATH"
./licenseactivator LSHOST=$SERVERPATH COMMUTE_MAX_LIFE=7
```