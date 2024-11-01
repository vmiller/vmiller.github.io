---
title: "Enabling and Disabling SSH Using Munki"
date: 2018-02-27T07:16:21-05:00
draft: false
tags: [macOS, Munki]
---

I recently needed to be able to test connectivity from client systems in a number of locations to some new infrastructure. Obviously being able to remotely login to these systems is a lot more efficient than traveling around to all the locations. We do not have SSH enabled by default on our managed macOS clients.

I wanted to use Munki to enable SSH on select systems for testing, and then be able to disable it after testing was completed. I chose to do a nopkg installer to do this.

First up, the installcheck_script to check if SSH is enabled :

```
#!/bin/bash
 
#check to see if ssh is off
if [[ $(/usr/sbin/systemsetup -getremotelogin) = 'Remote Login: Off' ]] 
then
    # exit 0 to tell Munki an install is needed
    exit 0
fi
# if not needed, exit 1
exit 1
```

Next, the postinstall_script to enable SSH :

```
#!/bin/bash
 
/usr/sbin/systemsetup -setremotelogin On
 
if [[ $(/usr/sbin/systemsetup -getremotelogin) = 'Remote Login: On' ]] 
then
    exit 0
fi
 
exit 1
```

Finally, the uninstall_script to disable SSH:

```
#!/bin/bash
 
/usr/sbin/systemsetup -f -setremotelogin Off
```
To build these into a suitable munki pkgsinfo :

```
makepkginfo --nopkg \
 --installcheck_script installcheck_script.sh \
 --postinstall_script postinstall_script.sh \
 --uninstall_script uninstall_script.sh \
 --unattended_install \
 --unattended_uninstall \
 --uninstall_method uninstall_script \
 --name Enable_SSH \
 --pkgvers 1.0 > Enable_SSH-1.0.plist
 ```

 I then had to edit Enable_SSH-1.0.plist to add an uninstallable key with a value of True. The plist was copied into our munki repo and makecatalogs was run. I now have an item that when added to managed_installs will enable SSH on the next munki run. Conversely I can add it to managed_uninstalls to disable remote login.

 Source files can be found in this [GitHub repo](https://github.com/vmiller/enable-ssh)