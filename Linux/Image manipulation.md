# Image manipulation

Notes on manipulating images. Mostly including ffmpeg and imagemagick.

### Converting from gif to to image series

coalesce means *merge a sequence of images*

```none
convert target.gif -coalesce Frames/output_%02d.png
```

### Converting from image series to a gif

```none
convert -delay 20 fullBody_*.png -loop 0 fullBody.gif
```
