.. OCR documentation master file, created by
   sphinx-quickstart on Wed Jul 10 18:12:59 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Welcome to OCR's documentation!
===============================
The main goal is to read a PDF file with the help of OCR and output the text in the PDF file. In order to increase efficiency, we run some steps of the process on high-performance computers.

There are several steps in the entire process:

1. Split the input PDF file into separated images
#. Enhance the contrast of each image and make it black and white. (Image Cleaning)
#. Create a checkpoint file to make a list of jobs to do. (Preprocess)
#. Read each image with the help of OCR and output text files.

Step 2-4 require slurm files to run on HTC.


.. toctree::
   :maxdepth: 2

   split
   slurm_1
   imageCleaning
   preprocess
   slurm_2
   ocr
   postprocess
   time

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
