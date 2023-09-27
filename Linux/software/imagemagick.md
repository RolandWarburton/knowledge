# Image manipulation

Notes on manipulating images. Mostly including ffmpeg and imagemagick.

## Converting from gif to to image series

coalesce means *merge a sequence of images*

```none
convert target.gif -coalesce Frames/output_%02d.png
```

## Converting from image series to a gif

```none
convert -delay 20 fullBody_*.png -loop 0 fullBody.gif
```

## Concatenating PDFs

Sometimes you just need to string PDFs together. For example making a mega document of all your uni lectures for revision purposes.

Firstly, collect all your PDFs into one folder. You can then use the renaming tool on most file managers (like Thunar) To rename each file numerically (1.pdf, 2.pdf, etc).

Then run the following command from the Poppler Library.

```none
pdfunite 1.pdf 2.pdf 3.pdf output.pdf
```

You can also collect a list of PDFs in a text file to copy paste into the pdfunite command with the following find command.

```none
find . -type f -iname "*.pdf" -exec echo \"{}\" \\ >> pdfs.txt \;
```

## applying and removing passwords to/from PDFs

There are two types of passwords that a PDF can have.

#### Owner Password

This is a **Read Only** password for a pdf that **prevents a user from editing** a pdf document, but they can still view it. This type of password is weak and can be removed easily.

##### Encrypting Owner Password

Encrypting can be done using pdftk. [source](https://www.pdflabs.com/docs/pdftk-cli-examples/).

```none
pdftk input.pdf output encrypted.pdf owner_pw mypassword
```

##### Decrypting Owner Password

To crack an owner password is very straightforward using qpdf. This isnt actually *cracking* the pdf, more like its just stripping out parts of the pdf that prevent editing from happening (no encryption is taking place).

```none
qpdf --decrypt password.pdf decrypted.pdf
```

You can use the find command to recursively check for every PDF and run the qpdf command above with the `--replace-input` tag to intentionally overwrite the input file

```none
find . -type f -iname "*.pdf" -exec qpdf --decrypt --replace-input {} \;
```

#### User Password

This is a more restrictive password that **removes all read and write access** from a pdf, requiring the owner and user to enter it before any type of reading or modification can be made. This is a more permanent method of locking a pdf down, as such it encrypts the entire document resulting in the PDF being undecipherable without the password.

##### Encrypting User Password

Encrypting can be done using pdftk. [source](https://www.pdflabs.com/docs/pdftk-cli-examples/).

```none
pdftk input.pdf output encrypted.pdf owner_pw mypassword
```

In addition you can combine both owner and user passwords on the same document, they can be unique or the same to provide access control to the PDF.

```none
pdftk input.pdf output encrypted.pdf owner_pw foo user_pw bar
```

##### Decrypting User Password

The process used to crack this can done using either John The Ripper, or hashcat. I prefer JTR for this as it comes with a pdf2john.pl script that makes hashing the pdf easier to perform password extraction.

For installing JTR and a hint for a common gotcha refer to the linux debugging section in either my notes repo [here](https://github.com/RolandWarburton/knowledge/blob/master/Linux/Debugging.md#setting-up-john-the-ripper-in-arch) or on my website under [/Notes/Linux/Debugging](/Notes/Linux/Debugging).

Using JTR create a hash of the encrypted PDF.

```none
/usr/share/john/pdf2john.pl encrypted.pdf > pdf.hash
```

Next run john on `pdf.hash`. If you want to use a special wordlist you can do so with the `--wordlist=words.txt` flag.

```none
john pdf.hash --progress-every=3
```

You after waiting (a long time) you should get an output from JTR after it has finished running. You can retrieve the output again at a later date by using the `john pdf.hash --show --format=PDF` command.

```none
password.pdf: password

1 password hash cracked, 0 left
```

#### Verifying PDF user/owner password

Using either the properties tab of your pdf viewer, you can see the state of the PDF (encrypted/unencrypted).

You can also use a CLI tool provided by the Poppler library `pdfinfo` to list the details regarding the pdf, including lots of valuable information (more than just for password/encryption purposes).
