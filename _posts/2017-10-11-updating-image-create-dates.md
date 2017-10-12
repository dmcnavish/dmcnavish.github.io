---
layout: post
title: Updating Image Create Dates
tags: [shell]
---

When I am not banging away at the keyboard, I enjoy photography. I have a DSLR Canon camera and usually shoot pictures in [RAW format](https://en.wikipedia.org/wiki/Raw_image_format) for a variety of reasons. The process for this is to export the RAW images from the camera onto my computer, edit them in Lightroom/Photoshop and then export them again into JPG format. This is all fairly standard stuff. 

One drawback to using this workflow is that the resulting images' create dates are the time when the JPG is exported and not when the picture was taken. This may seem like a minor detail, but when you take pictures over the course of a month and then go back and look to see that all of the images have a create date within seconds of each other, it is a little disappointing. 

But, all is not lost. It turns out that even though the images have the wrong create date, they still contain a ton of the original meta data. At first I thought all of my problems were solved, but then after looking a little harder, it turns out that each operating system displays different parts of the data and some ignore fields all together. Specifically, OSX's finder, doesn't allow you to see the create date. Back to square one.

After a little searching, I was able to find a tool named [exiftool](https://www.sno.phy.queensu.ca/~phil/exiftool/) that allows you to view meta data from the command line. Finally!

Below is a small script that iterates over all images in a directory, extracts the create date, and then updates the file with the original create date. It has been tested and works on OSX, if using on another OS, you may need to change the date format.

```shell
#!/bin/bash

for FILENAME in /Users/davidmcnavish/temp/pics/*.jpg; do
        DATE=$(exiftool -xmp:all  $FILENAME  | grep "Date Created" | sed 's/.*: //')
        FORMATTED_DATE=$(date -j -f "%Y:%m:%d %H:%M:%S.%N" "$DATE" +"%y%m%d%H%M")
        touch -mt $FORMATTED_DATE $FILENAME
done
```

You can find the original snippet [here](https://gist.github.com/dmcnavish/f88cd002b01f732ae27ed1b21afd674d).