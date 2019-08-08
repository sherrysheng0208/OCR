Problems encountered
====================
Image-cleaning
--------------
For now, image-cleaning is not good enough to output high-quality images. Users need to set their own parameters in the python program in order to get better results. 

pdfimages
---------
We failed on using *pdfimages* to split PDF files to images because we cannot install poppler on HTC. Using *pdfimages* in the command line can help us make the code simpler and reduce the amount of code.

Wand
----
It turns out that using *wand.image* to split PDF files into high-resolution images is really slow. The time it takes is so long that the process will be killed on the HTC.

