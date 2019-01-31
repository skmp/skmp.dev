---
layout: post
title:  "Updated Reicast CI builds page"
categories: jekyll update
img: reicast-ci.png
---

I realized yesterday the Reicast builds page was not working for some odd reason. It turns out amazon prefers to be called via https, and I decided to do it that way instead of fiddling with the S3 settings. While I was at it, I replaced that random GPL listing code with something nicer.

Lo and behold, the new automated builds page

image

Using the magic power of jquery, underscorejs & regex this looks much nicer and is more functional.

Builds are sorted by branches, with the "testing", and "master" branches appearing on top. 

At some point some bootstrap-based ui, or proper "backend" might be in order, but this will have to do for now.

*update*

The code is up at github