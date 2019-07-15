Split PDF file into images
==========================
Assume the name of the input file is document.pdf. We can set the number of pages that we want to extract from the PDF file. For example, we can extract 260 images from document.pdf. ::

 import PyPDF2
 from PIL import Image

 if __name__ == '__main__':
     input1 = PyPDF2.PdfFileReader(open('document.pdf', 'rb'))
     for i in range(1, 260):
         page0 = input1.getPage(1)
         xObject = page0['/Resources']['/XObject'].getObject()

         for obj in xObject:
             if xObject[obj]['/Subtype'] == '/Image':
                 size = (xObject[obj]['/Width'], xObject[obj]['/Height'])
                 data = xObject[obj]._data
                 if xObject[obj]['/ColorSpace'] == '/DeviceRGB':
                     mode = "RGB"
                 else:
                     mode = "P"

                 if '/Filter' in xObject[obj]:
                     if xObject[obj]['/Filter'] == '/FlateDecode':
                         img = Image.frombytes(mode, size, data)
                         img.save(obj[1:] + ".png")
                     elif xObject[obj]['/Filter'] == '/DCTDecode':
                         img = open(obj[1:] + ".jpg", "wb")
                         img.write(data)
                         img.close()
                     elif xObject[obj]['/Filter'] == '/JPXDecode':
                         img = open(obj[1:] + ".jp2", "wb")
                         img.write(data)
                         img.close()
                     elif xObject[obj]['/Filter'] == '/CCITTFaxDecode':
                         img = open(obj[1:] + ".jpg", "wb")
                         img.write(data)
                         img.close()
                 else:
                     img = Image.frombytes(mode, size, data)
                     img.save(obj[1:] + ".png")
 else:
    print("No image found.")


