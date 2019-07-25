OCR
===
Processor ID
------------
First of all, we should let each processor get its own ID number so that it can read its corresponding child checkpoint file and start to work on the tasks assigned to it. ::

    import csv
    import time
    import sys
    from PIL import Image
    from PIL import ImageEnhance
    import pytesseract
    from optparse import OptionParser, Option, OptionValueError
    
    fileName = str(sys.argv[1])
  
    def list_callback(option, opt, value, parser):
            setattr(parser.values, option.dest, value.split(','))
    
    parser = OptionParser()
    parser.add_option("-p", action="store", type="str", dest="proc_id")
    
    (options, args) = parser.parse_args()
    
    proc_id = options.proc_id


Child checkpoint_DONE file
--------------------------
Second, each processor will create another child checkpoint file named checkpoint_i_DONE.csv to record the tasks that have been done. In the beginning, those files are empty.  ::

  with open(fileName + "/checkpoint_" + str(proc_id) + "_" + "DONE" + ".csv", "w") as c:
      writer = csv.writer(c)
      writer.writerow(["File Path"])



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
Next step is the most important part as OCR will start to read the images according to the lists in the checkpoint_i_TODO.csv. After OCR finishes reading, these tasks will be marked in the checkpoint_i_DONE.csv and the output text files can be found in the output/*fileName* folder.   ::

    time_label = []
    cur = 0
    with open(fileName + "/checkpoint_" + str(proc_id) + "_" + "TODO" + ".csv", "r") as c3:
        reader = csv.reader(c3)
        paths = [row for row in reader]
        cur = len(paths)-1
    
        for j in range(1, len(paths)):
            img_path = paths[j][0]
    
            start = time.time()
    
            img = Image.open(img_path)
            img = binarizing(img, 160)
            img.save(img_path)
    
            text = pytesseract.image_to_string(Image.open(img_path),lang='eng', config=r'--tessdata-dir "/home/yourID/.linuxbrew/Cellar/tesseract/4.0.0_1/share/tessdata/"')
    
            end = time.time()
            time_label.append(end-start)
    
            output = str(paths[j][0]).replace("/dataset/", "/output/")
            output = output.replace(".jpg", ".txt")
            out = open(str(output), "w")
            out.write(text)
            out.close()
    
            with open(fileName + "/checkpoint_" + str(proc_id) + "_" + "DONE" + ".csv", "a") as c4:
                writer = csv.writer(c4)
                writer.writerow([paths[j][0]])


Time
----
In order to do the benchmark, we also take the records of average time that OCR takes to read one image. The average time of each image-processing is written into a file named timing.csv. In the end, this file will show us how many images each processor has read and how long it takes. ::

    whole_time = 0
    print("Number of pages in the processor: " + str(cur))
    for i in range(cur):
        whole_time += time_label[i]
    
    with open(fileName + "/timing.csv", "a") as t1:
        writer = csv.writer(t1)
        writer.writerow([cur, whole_time/cur])


