Pre-process
===========
Get Processor ID
----------------
First of all, we should let each processor get its own ID number because we only want to run pre-process on one specific processor. ::

    from optparse import OptionParser, Option, OptionValueError

    def list_callback(option, opt, value, parser):
            setattr(parser.values, option.dest, value.split(','))

    parser = OptionParser()
    parser.add_option("-p", action="store", type="str", dest="proc_id")
    (options, args) = parser.parse_args()
    proc_id = str(options.proc_id)


Make directories
----------------
Next, we make two directories respectively for processed input and output.

Function *mkdir_output* will create a directory with exactly the same structure as your original directory, but replace all the PDF files with folders of the same names. As a result, output texts from OCR can be stored in the corresponding output folders.

Function *mkdir_input* will do the same job as *mkdir_output*, but in the meantime it calls functions *split* and *make_checkpoint*, which will be explained in details in the next sections. ::

    def mkdir_output(rootDir):
        for lists in os.listdir(rootDir):
            if not lists.startswith("."):
                path = os.path.join(rootDir, lists)
    
                if os.path.splitext(lists)[-1] == '.pdf':
                    path_file = os.path.splitext(path)[0]
                    if not os.path.exists(path_file + "_Folder"):
                        os.makedirs(path_file + "_Folder")
                        os.remove(path)
                elif os.path.isdir(path) and not path.endswith("_Folder"):
                    mkdir_output(path)
    
    
    def mkdir_input(rootDir, processor_count):
        for lists in os.listdir(rootDir):
            if not lists.startswith("."):
                path = os.path.join(rootDir, lists)
                if os.path.splitext(lists)[-1] == '.pdf':
                    file_path = os.path.splitext(path)[0]
    
                    if not os.path.exists(file_path + "_Folder"):
                        os.makedirs(file_path + "_Folder")
    
                        try:
                            split(file_path)
                            make_checkpoint(file_path, processor_count)
                        except:
                            print("Failed on", file_path + ".pdf")
                            print("Unexpected error:", sys.exc_info()[0])
    
                elif os.path.isdir(path) and not path.endswith("_Folder"):
                    mkdir_input(path, processor_count)



Split files
-----------
In the function *mkdir_input*, we call function *split* to split the input PDF file into separate images so that we can read each page as images one by one. The images will be stored in the  *your_directory_input/file_name_Folder* folder.  ::

    def split(file_path):
        print("File:", file_path + '.pdf')
        filename = str(os.path.basename(file_path))
    
        inputpdf = PyPDF2.PdfFileReader(open(file_path + '.pdf','rb'))
        if inputpdf.isEncrypted:
            inputpdf.decrypt('')
    
        for i in range(inputpdf.numPages):
            start = time.time()
    
            page0 = inputpdf.getPage(i)
            xObject = page0['/Resources']['/XObject'].getObject()
    
            for obj in xObject:
                if xObject[obj]['/Subtype'] == '/Image':
                    size = (xObject[obj]['/Width'], xObject[obj]['/Height'])
                    data = xObject[obj]._data
                    if xObject[obj]['/ColorSpace'] == '/DeviceRGB' or '/DeviceRGB' in xObject[obj]['/ColorSpace']:
                        mode = "RGB"
                    else:
                        mode = "P"
    
                    if '/Filter' in xObject[obj]:
                        if xObject[obj]['/Filter'] == '/FlateDecode':
                            img = Image.frombytes(mode, size, data)
                            img.save(file_path + "_Folder/" + filename + "_" + str(i) + ".png")
                        elif xObject[obj]['/Filter'] == '/DCTDecode':
                            img = open(file_path + "_Folder/" + filename + "_" + str(i) + ".jpg", "wb")
                            img.write(data)
                            img.close()
                        elif xObject[obj]['/Filter'] == '/JPXDecode':
                            img = open(file_path + "_Folder/" + filename + "_" + str(i) + ".jp2", "wb")
                            img.write(data)
                            img.close()
                        elif xObject[obj]['/Filter'] == '/CCITTFaxDecode':
                            img = open(file_path + "_Folder/" + filename + "_" + str(i) + ".tiff", "wb")
                            img.write(data)
                            img.close()
                    else:
                        img = Image.frombytes(mode, size, data)
                        img.save(file_path + "_Folder/" + filename + "_" + str(i) + ".png")
    
            end = time.time()
            print("Time for spliting PDF:", end - start, ",", end)


Master & Child checkpoint files
-------------------------------
Next, we create a master checkpoint file named checkpoint.csv in the  *your_directory_input/file_name_Folder* folder, which lists a number of tasks to be done. The initial state of each task is set as False, which represents that the job hasn’t been done yet.

Besides, we assign equal number of tasks to each processor according to the number of processors that we have entered in the previous command line. The list of tasks to be done in each processor are recorded in child checkpoint files named checkpoint_i_TODO.csv. *i* represents the ID of the processors. These files are also in the *your_directory_input/file_name_Folder* folder.  ::

    def make_checkpoint(file_path, processor_count):
        csv_file = []
        csv_state = []
    
        dir = file_path + "_Folder"
        if os.path.exists(dir + "/checkpoint.csv"):
            os.remove(dir + "/checkpoint.csv")
    
        for j in [n for n in os.listdir(dir) if (os.path.isfile(os.path.join(dir, n)) and not n.startswith("."))]:
            csv_file.append(dir + "/" + j)
            csv_state.append("F")
    
        dataframe = pd.DataFrame({'File Path': csv_file, 'State': csv_state})
    
        dataframe.to_csv(dir + "/checkpoint.csv", index=False, sep=',')
    
        with open(dir + '/checkpoint.csv', 'r') as f:
            reader = csv.reader(f)
            rows = [row for row in reader if row[1] == "F"]
        file_count = len(rows)
    
        jobs_count = file_count // processor_count
    
        remainder = file_count % processor_count
        cur = 0
        for i in range(remainder):
            with open(dir + "/checkpoint_" + str(i) + "_TODO.csv", "w") as c1:
                writer = csv.writer(c1)
                writer.writerow(["File Path"])
                for j in range(jobs_count + 1):
                    writer.writerow([rows[cur][0]])
                    cur += 1
    
        for i in range(remainder, processor_count):
            with open(dir + "/checkpoint_" + str(i) + "_TODO.csv", "w") as c2:
                writer = csv.writer(c2)
                writer.writerow(["File Path"])
                for j in range(jobs_count):
                    writer.writerow([rows[cur][0]])
                    cur += 1

pre_process.py
--------------
We run pre_process.py on Processor #0 to make similar input and output directories and execute the above functions.  ::

    import shutil
    import PyPDF2
    from PIL import Image
    import pandas as pd
    import os
    import sys
    import csv
    import time
    from optparse import OptionParser, Option, OptionValueError
    
    processor_count = int(sys.argv[1])
    directory = str(sys.argv[2])
    
    if proc_id == "0":
    
        if not os.path.exists(directory + '_input'):
            shutil.copytree(directory, directory + '_input')
        if not os.path.exists(directory + '_output'):
            shutil.copytree(directory, directory + '_output')
    
        mkdir_output(directory + '_output')
        mkdir_input(directory + '_input', processor_count)



