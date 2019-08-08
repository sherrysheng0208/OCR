OCR
===
Get Processor ID
----------------
Same as pre-process, we should let each processor get its own ID number so that it can read its corresponding child checkpoint file and start to work on the tasks assigned to it. ::

    from optparse import OptionParser, Option, OptionValueError

    def list_callback(option, opt, value, parser):
            setattr(parser.values, option.dest, value.split(','))

    parser = OptionParser()
    parser.add_option("-p", action="store", type="str", dest="proc_id")
    (options, args) = parser.parse_args()
    proc_id = str(options.proc_id)


Image cleaning
--------------
Before we do the OCR, we want the images to be clearer and easier to read so that OCR can output better results. Therefore, we need to do some image cleaning, such as enhancing contrast of images and changing the colour of images to black and white. All the parameters in the following code can be changed based on different requirements and quality of images. ::

    def binarizing(img, threshold):
        img = img.convert("L")
    
        enh_con = ImageEnhance.Contrast(img)
        contrast = 3
        img = enh_con.enhance(contrast)
    
        pixdata = img.load()
        w, h = img.size
    
        for y in range(h):
            for x in range(w):
                if pixdata[x, y] < threshold:
                    pixdata[x, y] = 0
                else:
                    pixdata[x, y] = 255
        return img


Read images with OCR
--------------------
Next step is the most important part of the entire program. Each processor will first create another child checkpoint file named checkpoint_i_DONE.csv to record the tasks that have been done. They are empty in the beginning. Then, the processors will start to read images according to the lists in the checkpoint_i_TODO.csv. After OCR finishes reading, these tasks will be recorded in the checkpoint_i_DONE.csv and output text files can be found in the *your_directory_output/file_name_Folder* folder.

In order to test benchmark and plot, the function will print how long OCR takes to read one image and its end time. ::

    def ocr(file_path, proc_id):
        with open(file_path + "_Folder/checkpoint_" + str(proc_id) + "_DONE.csv", "w") as c:
            writer = csv.writer(c)
            writer.writerow(["File Path"])
    
        with open(file_path + "_Folder/checkpoint_" + str(proc_id) + "_TODO.csv", "r") as c3:
            reader = csv.reader(c3)
            paths = [row for row in reader]
    
            for j in range(1, len(paths)):
                img_path = paths[j][0]
    
                start = time.time()
    
                img = Image.open(img_path)
                img = binarizing(img, 160)
                img.save(img_path)
    
                text = pytesseract.image_to_string(Image.open(img_path), lang='eng', config=r'--tessdata-dir "/home/your_ID/.linuxbrew/Cellar/tesseract/4.0.0_1/share/tessdata/"')
    
                output = str(paths[j][0]).replace("input", "output")
                output = output.replace(".jpg", ".txt")
                out = open(str(output), "w")
                out.write(text)
                out.close()
    
                with open(file_path + "_Folder/checkpoint_" + str(proc_id) + "_DONE.csv", "a") as c4:
                    writer = csv.writer(c4)
                    writer.writerow([paths[j][0]])
    
                end = time.time()
                print("Time for OCR:", end - start, ",", end)


ocr.py
------
We run ocr.py on all the processors to increase efficiency.  ::

    from PIL import Image
    import os
    import sys
    import csv
    import time
    from PIL import ImageEnhance
    import pytesseract
    from optparse import OptionParser, Option, OptionValueError
    
    directory = str(sys.argv[1])

    def ocr_all_files(rootDir, proc_id):
        for lists in os.listdir(rootDir):
            if not lists.startswith("."):
                path = os.path.join(rootDir, lists)
                if os.path.splitext(lists)[-1] == '.pdf':
                    file_path = os.path.splitext(path)[0]
                    print(file_path)
                    if os.path.exists(file_path + "_Folder/checkpoint.csv"):
                        ocr(file_path, proc_id)
    
                if os.path.isdir(path) and not path.endswith("_Folder"):
                    ocr_all_files(path, proc_id)

    ocr_all_files(directory + '_input', proc_id)


