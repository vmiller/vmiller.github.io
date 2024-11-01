---
title: "Packaging a Drag and Drop Application"
date: 2012-08-14T07:11:20-05:00
draft: false
tags: [macOS, Luggage, Packaging]
---

In a previous [post]({{< ref "20120524-getting-started-with-the-luggage" >}}), I showed how to get started by getting Luggage setup.  In this post we will create a simple package based on a .app bundle.

Drag and drop applications are really user friendly to install, the user simply needs to drag them to the Applications folder.  They are not as nice, however, for the system administrator who wishes to deploy the application via some automated tool.  In some cases it is desirable to repackage these application into an installer package.  Fortunately this is easy to do with the Luggage.

For this demonstration, I will use my favorite text editor TextWrangler.app.  When we download TextWrangler it is downloaded as a disk image.  Mount the image file by double clicking on it, and we see that we have an app bundle and a link to the Applications folder.  The application bundles are actually a special directory, so our first step is to make a compressed tar of the app.

Open Terminal and execute the following commands (be sure to scroll to the right to see all of the second command) :

```
cd /Volumes/TextWrangler\ 4.0.1/
 
/usr/bin/tar cvjf ~/Downloads/TextWrangler.app.tar.bz2 TextWrangler.app
```

This will create a tar file in the Downloads directory.  Now, create a directory for our files that Luggage will need.  Move the tar file into this directory and  then create a text file with the file name of Makefile (make sure an extension does not get added to the filename when it is saved).  The contents of Makefile should be as follows :

```
include /usr/local/share/luggage/luggage.make
TITLE=install_TextWrangler
REVERSE_DOMAIN=com.example.corp
PAYLOAD=unbz2-applications-TextWranger.app
```

The “include” line tells Luggage where to find the luggage.make file and includes the built in rules that we will use.

The TITLE line is where we specify the title that we want our resulting installer to have.  You can change this to suite your own taste.

The REVERSE_DOMAIN line should be changed to the domain that is applicable to your organization.

The PAYLOAD line is the one that tells The Luggage what to do with the files we provide.  In this example we are using a built in rule that will unpack a bz2 tar file into the /Applications folder.

In Terminal, change into the directory where we have our tar file and our Makefile and run the following command :

`sudo make pkg`

The result of this command will be the creation of a .pkg file that will install TextWrangler.app into /Applications If we want that installer wrapped in a disk image we can run the following command instead :

`sudo make dmg`

This is a minimal example, and a fairly painless way to get started with The Luggage and Makefiles.  All make files must have these four lines at minimum.  Next, go here to learn how to package a Launch Agent.