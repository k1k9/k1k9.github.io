---
title: Smiling ASCII - CTFlearn
author: k1k9
categories: [CTFlearn, forensics]
tags: [ctf, ctflearn]
---
Points: 40
Level: Medium

Find the flag on the smiling face.
![smiling_face](/assets/posts/smiling-ascii/entry.png)


## Let's go
First of all, let's check what file has inside:
```sh
strings smiling.png | less
```
On the end of file we see:
```RGlkIHlvdSBrbm93IHRoYXQgcGl4ZWxzIGFyZSwgbGlrZSB0aGUgYXNjaWkgdGFibGUsIG51bWJlcmVkIGZyb20gMCB0byAyNTU/Cg==``` with is a base64 message: 
```sh
$ echo RGlkIHlvdSBrbm93IHRoYXQgcGl4ZWxzIGFyZSwgbGlrZSB0aGUgYXNjaWkgdGFibGUsIG51bWJlcmVkIGZyb20gMCB0byAyNTU/Cg== | base
64 -d

> Did you know that pixels are, like the ascii table, numbered from 0 to 255?
```
So we need dig deeper but first let's take a look of image. After zooming in we see various shades of gray, but in top left corner we have something like this:
![dum_dum](/assets/posts/smiling-ascii/top_left.png)

## Ending
We know that we should pay attention to the image and it is all about encoded characters. So let's write a quick program in python that will read us these colors and convert them to ascii.
```python
from PIL import Image

image = Image.open("smiling.png")
width, height = image.size
for y in range(height):
    for x in range(1):
        pixel = image.getpixel((x,y))
        if pixel == (255, 255, 255, 255): break
        print(chr(pixel[3]), end="")
```
And we have the flag