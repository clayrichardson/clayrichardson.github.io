---
layout: post
title: "ImageMagick autocrop scanned documents"
date: 2013-12-27 03:09:55 -0600
comments: true
categories: 
---
I needed to autocrop a bunch of handwritten 14.8cm x 21cm notes I scanned.
This requires ImageMagick:
```
brew install imagemagick
```
Use the convert command:
```
convert ./*.png -fuzz 1% -trim +repage -set filename:f "$(date)-%f" "./CroppedScans/Cropped-%[filename:f]"
```

