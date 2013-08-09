---
layout: post
title: "Vagrant NFS mount failing on mac osx"
date: 2012-05-03 23:01
comments: true
categories: [Ruby]
---

## The Problem

Vagrant is running slow so you decide to try nfs shared folders since there is
a [documented](http://vagrantup.com/v1/docs/nfs.html) performance boost 
with it. However, running `vagrant up` fails with the error:

```
Mounting NFS shared folders failed. This is most often caused by the NFS
client software not being installed on the guest machine. Please verify
that the NFS client software is properly installed, and consult any resources
specific to the linux distro you're using for more information on how to
do this.
```

But I already had nfs on the host and guest machine. The only other clue I
had was a strange message during the modification of /etc/exports:

```
[default] Exporting NFS shared folders...
[vagrant] Preparing to edit /etc/exports. Administrator privileges will be required...
Password:
su: Sorry
su: Sorry
su: Sorry
```

Notice the `su: Sorry` showing up several times.

## The Solution

Turns out that even though I gave the right password, my host machine didn't
have root user enabled. For whatever reason, this was preventing vagrant from
modifying the guest machine's /etc/exports file (shown by `su: Sorry`). Once I
enabled the root user everything worked just fine.
