---
title: "Stupid SSH Tricks"
date: 2024-12-18T09:16:19-05:00
draft: false
tags: [macOS]
---

I'm writing down these SSH configuration tweaks because I tend to change them infrequently enough that I forget them :-)  

## Configuring SSH keys

Sometimes it might be useful to use seperate SSH keys for seperate purposes.  For example, one for work and one for peprsonal.  On macOS this can be accomplished using `~/.ssh/config`

For each host that you want to use a specific key for, you can create an entry like this :
```
Host gitlab.home
	Hostname 192.168.1.50
	IdentityFile ~/.ssh/home_rsa
```
The `Host` line gives a user freindly name, the `Hostname` line specifies the IP address or DNS name of the desired host.  Lastly the `IdentityFile` line specifies the path to the ssh key to use for authentication.  

## Disabling Host Verification

Use cauton with this one...  In my case I need to ssh into lab computers for maintenance purposes.  These systems are on DHCP and may get new IP addresses at times.  Having to modify the `known_hosts` file everytime this happened was a bit of a pain.  To diasble host verification I added the following to `~/.ssh/config` 
```
Host *
	StrictHostKeyChecking no
	UserKnownHostsFile=/dev/nul
```
This completely disables host checking.  