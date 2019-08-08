Languages
=========
OCR supports various languages, such as English, French. But users need to know what language they want OCR to read before getting started. They need to download the trained data from tesseract-ocr website and upload it to this directory on the HTC: /home/*your_ID*/.linuxbrew/Cellar/tesseract/4.0.0_1/share/tessdata/.

Also, remember to change the parameter “lang” in the following line in the ocr.py. ::

    text = pytesseract.image_to_string(Image.open(img_path),lang='eng', config=r'--tessdata-dir "/home/your_ID/.linuxbrew/Cellar/tesseract/4.0.0_1/share/tessdata/"')



