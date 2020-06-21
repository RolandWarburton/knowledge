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

### Concatenating PDFs

Sometimes you just need to string PDFs together. For example making a mega document of all your uni lectures for revision purposes.

Firstly, collect all your PDFs into one folder. You can then use the renaming tool on most file managers (like Thunar) To rename each file numerically (1.pdf, 2.pdf, etc).

Then run the following command from the Poppler Library.

```none
pdfunite 1.pdf 2.pdf 3.pdf output.pdf
```
