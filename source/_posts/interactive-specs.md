---
title: Interactive Specs
date: 2016-05-22 00:31:05
tags:
---

It's 2016 and specifications and communication protocols are still hard to understand. I find this weird if one of the main intentions of a spec is for it to be understood and implemented by others.

I had to extract some metadata from a JPG through the EXIF headers recently. The EXIF spec was written in 1995 and has been implemented in all major languages, yet in order to understand the spec, I ended up having to spend several hours looking at a JPEG hexdump, manually trying to match bytes to the spec.

In my ideal world, all specs would have some element of interactivity. I attempted to create what an "interactive" spec would look like, with the goal being to help someone gain a more intuitive understanding of the spec.

Here's an example of an interactive EXIF spec showing how the GPS metadata works:

![exif gif](/images/exif.gif)

It's online here: [http://jiahuang.github.io/exif-viewer/](http://jiahuang.github.io/exif-viewer/).

The binary of an image can be highlighted to explain what part of the EXIF spec corresponds with the resulting data. This would have saved me hours of time. It doesn't even have that much *more* additional data than the [EXIF spec](http://www.kodak.com/global/plugins/acrobat/en/service/digCam/exifStandard2.pdf).

I can list off dozens of JavaScript frameworks for building interactive user-facing websites, but I can only list two documentation frameworks (Swagger, and Gitbooks -- if Gitbooks even counts). Pages and pages of pdf specs is outdated and ineffective. Developers need better documentation tools. 
