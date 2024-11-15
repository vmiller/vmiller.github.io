---
title: "Logic Pro Rename"
date: 2024-11-14T11:23:22-05:00
draft: falso
tags: [macOS, Munki]
---

With the 11.1 update, Logic Pro has changed the app bundle from `Logic Pro X.app` to `Logic Pro.app`

But wait!  Didn't this change happen a few updates back?  Yes and no...  The file sustem name was `Logic Pro X.app` but it displayed in Finder as `Logic Pro.app` 

If installing Logic Pro from the App Store, none of this is important as it is all seemless to the user.  If like me you are managing this on managed endpoints, the path changing is important.  Updating with Munki will result in the two app bundles being side by side with both presenting to the user as `Logic Pro.app`  A simple pre or post install scrip to check for and delete `Logic Pro X.app` will clean that up.  If you have Logic in a managed Dock profile, don't forget to change that path as well!