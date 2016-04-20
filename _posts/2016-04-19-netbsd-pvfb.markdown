---
layout: post
title: "xen pv framebuffer support in netbsd"
date: 2016-04-20
tags: netbsd xen programming
---
Recently installed 7.0 on a xen VM. There is no paravirtual driver video driver for it yet,
but there is a [project proposal](http://wiki.netbsd.org/projects/project/xen-domu-pvfb/).

After some work (took me considerably more than the estimated 16hs, truth be told)
got something usable.  The patch is [here]({{site.url}}/patches/pvfb.patch)

To apply the patch, run from the kernel' src dir

    patch -p1 < pvfb.patch  

It will patch the default XEN_DOMU kernel configuration to add the new video/keyboard/mouse drivers.
The code is basically a glue between xenbus and genfb | wskbddev | wsmousedev

base functionality is in place, check the [sample screencast](https://sendvid.com/6q7j2yju), 
but there are several aspect that need review/fixes, among others:

* Xen suspend/resume seems broken, but so do without the driver, too.
* No way to modify the screen resolution without recompiling. (default is set to 1280x800, 16bits) 
* Keyboard under X do not recognize arrows/most of keypad keys. These *are* recognized when in terminal, so I think the problem is in wskbd driver in X
* Framebuffer do not implement feature-update, that is, it do not try to detect what areas did change. This imposes additional work on the qemu side

If you want to give it a try and run X,  you likely will need to configure xorg.conf by hand. Use this as a template:

    Section "InputDevice"
    	Identifier "Mouse0"
        Driver	"mouse"
        Option	"Protocol" "wsmouse"
        Option	"Device" "/dev/wsmouse0"
    EndSection
    Section "InputDevice"
        Identifier "Keyboard0"
        Driver	"kbd"
        Option	"Protocol" "wskbd"
        Option	"Device" "/dev/wskbd0"
        Option	"XkbModel" "pc102"
    EndSection
    Section "Device"
        Identifier	"driver"
        Driver	"wsfb"
        Option	"device"	"/dev/ttyE3"
    EndSection
    Section "Screen"
        Identifier	"screen"
        Device	"driver"
    EndSection
    Section "ServerLayout"
        Identifier	"Builtin Default Layout"
        Screen	"screen"
    EndSection

Note this was done from someone -me- that works on erlang for a living, not C, has been a decade since the last time he used a lower level language, and never had written kernel code before.
Feedback is appreciated. 
