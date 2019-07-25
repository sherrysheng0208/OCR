.. OCR documentation master file, created by
   sphinx-quickstart on Wed Jul 10 18:12:59 2019.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Getting started with OCR
===============================
Optical character recognition or optical character reader (OCR) is a technology that can help you convert different types of images, such as scanned documents, PDF files or photos of documents into machine-encoded text. When we want to use OCR to read extremely large files, it may take us much time to complete the entire process. In order to increase the efficiency, we may need the help of High Throughput Computing (HTC) cluster, which can support large work loads including single node, parallel and large memory multithreaded jobs. 

The program we use to run OCR on HTC mainly consists of four parts, including pre_process.py, ocr.py, post_process.py and a slurm script. 

In the pre-process, we split input PDF files into images and create master and child checkpoint files. In the OCR process, we do image-cleaning and convert the images into text. In the post-process, we take the record of the finished and unfinished jobs and update master checkpoint files.

What users need to do to run the program is:

1. Input the PDF files that they want to convert
#. Make directories to store output images, output texts and checkpoint files
#. Run the program in one command line and specify how many processors they want it to run on
#. Check the output texts and master checkpoint files


.. toctree::
   :maxdepth: 2

   instructions
   pre_process
   cleaning_ocr
   post_process
   timing
   language
   problems

Indices and tables
==================

* :ref:`genindex`
* :ref:`modindex`
* :ref:`search`
