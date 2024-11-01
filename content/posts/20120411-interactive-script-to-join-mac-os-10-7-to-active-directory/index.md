---
title: "Interactive Script to Join Mac OS 10.7 to Active Directory"
date: 2012-04-11T07:09:48-05:00
draft: false
tags: [macOS]
---

According to this document from Apple : http://support.apple.com/kb/TS4176  You may have trouble using the Directory Utility if your domain is setup such that the Netbios name and host name are not the same.  This is the situation I find myself in, so I thought I would post a sample script to help others out…

The script can be found [here](https://github.com/vmiller/vmiller_scripts/tree/master/Interactive_AD_Bind)