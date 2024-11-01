---
title: "Building a Signed Package With the Luggage"
date: 2015-05-25T07:14:27-05:00
draft: false
tags: [macOS, Luggage, Packaging]
---

Recently I wanted to be able to sign a package that I was building with The Luggage.

Since The Luggage is calling pkgbuild to build the package, I took a look at the [pkgbuild documentation](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man1/pkgbuild.1.html) and determined that the following argument was needed :
`--sign "Common name of signing cert"`

The question then became : how to add this to my Makefile? Taking a look at [luggage.make](https://github.com/unixorn/luggage/blob/master/luggage.make) I saw that `PB_EXTRA_ARGS` is the variable used to contain the arguments for the pkgbuild command.

To add my signing argument, I simply added this line :
`PB_EXTRA_ARGS+= --sign "Developer ID Installer: John Doe (ID12345678)"`
This appends my `–-sign` argument to the list of pkgbuild arguments and can be placed anywhere after the statement that includes luggage.make.

