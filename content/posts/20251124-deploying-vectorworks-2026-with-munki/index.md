---
title: "Deploying Vectorworks 2026 With Munki"
date: 2025-11-24T08:50:25-05:00
draft: false
tags: [Vectorworks, macOS, Munki]
---

## Overview
In a [previous post]({{< ref "/posts/20251119-deploying-vectorworks-2026/index.md" >}}) I documented the behavior of the new (for version 2026) Vectorworks installer package.  In this post I will show how I make this all work with Munki (and AutoPkg) to deploy it to my endpoints.

### Structure
To accomplish everything that needs to happen, I have three packages.  These are then chained together using Munki's `requires` and `update_for` logic.  The packages and their order look like this : 
1. LDF file package
2. Vectorwors package
3. Preference and License package

*LDF file package* - this package stages the `.ldf` file to disk in a location that can be referenced later by the Vectorworks package.  I obtain a valid `.ldf` file by performing a manual install of Vectorworks.  I believe you can also obtain this from support.  I then build a package whose payload deploys the file to `/Library/MyOrg/vwfiles/`

*Vectorworks package* - this "package" is actually two items wrapped in a disk image.  At the root of the disk image is the installer package downloaded from Vectorworks and the vw2026.plist file.  The vw2026.plist file contains your serial number and the path to the LDF file that the prior package deploys.  The disk image is imported into Munki and The *LDF file package* is put in a `requires` array in the pkgsinfo of the *Vectorworks package*

*Preference and License package* - this package does two things.  First, it deploys the `net.nemetschek.vectorworks.2026.plist` to `/Library/Preferences` as mentioned in my prior post.  In addition, I deploy a custom `LoginDialog.xml` file to `/Applications/Vectorworks 2026/Settings/SeriesG`. This file points the endpoints to a license server.  This pkgsinfo for this package lists it as an `update_for` the *Vecttorworks package* 

### Automating some of this with AutoPkg
The 1st and 3rd package above do not update frequently, typically just with a new major version of Vectorworks.  I builld those packages with munki-pkg and keep with source files in a git repository.

The 2nd package, onn the other hand, will update with each Vectorworks update.  For each update we'll need to download an updated package, wrap it in a disk image with our plist file and then import that into munki.  This sounds like a good task for AutoPkg to handle.  I have published [download](https://github.com/autopkg/vmiller-recipes/blob/master/Nemetschek/VectorworksPkg.download.recipe) and [munki](https://github.com/autopkg/vmiller-recipes/blob/master/Nemetschek/VectorworksPkg.munki.recipe) recipes to help with this.  AutoPkg can also handle sorting out the correct version as the package receipt version is just `1.0.0`  The download recipe takes care of that by unpacking the package and getting the version info from the app bundle in the payoad.