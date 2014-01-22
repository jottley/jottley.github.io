---
title: Jeos and Initramfs
author: jared
layout: post
permalink: /general/jeos-and-initramfs
aktt_tweeted:
  - 1
categories:
  - General
tags:
  - IDE
  - initramfs
  - jeos
  - SCSI
  - ubuntu
  - VMWare
---
<div>
  When <a href="http://cdimage.ubuntu.com/jeos/releases/gutsy/release/" target="_blank">installing</a> <a href="http://www.ubuntu.com/" target="_blank">Ubuntu</a> <a href="http://en.wikipedia.org/wiki/Jeos" target="_blank">Jeos</a> in VMWare, you need to change the disk type in VMWare from SCSI to IDE, otherwise, when it goes to boot, it will appear to hang and then launch in to <a href="https://wiki.ubuntu.com/Initramfs" target="_blank">initramfs</a>, because it is not expecting SCSI disks, even though you may have installed on SCSI disks. This was very frustrating for a couple of hours.
</div>