Pre-process
===========
Split files
-----------
The first thing we need to do in the pre-process is to split the input PDF file into separated images so that we can read each page one by one. The images will be stored in the dataset/*fileName* folder. ::

    import pandas as pd
    import os
    import sys
    import csv
    import PyPDF2
    from PIL import Image
    
    processor = int(sys.argv[1])
    fileName = str(sys.argv[2])
    
    if __name__ == '__main__':
        inputpdf = PyPDF2.PdfFileReader(open(fileName + '.pdf','rb'))
        for i in range(inputpdf.numPages):
            page0 = inputpdf.getPage(i)
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
                            img.save('dataset/' + fileName + "/" + obj[1:] + ".png")
                        elif xObject[obj]['/Filter'] == '/DCTDecode':
                            img = open('dataset/' + fileName + "/" + obj[1:] + ".jpg", "wb")
                            img.write(data)
                            img.close()
                        elif xObject[obj]['/Filter'] == '/JPXDecode':
                            img = open('dataset/' + fileName + "/" + obj[1:] + ".jp2", "wb")
                            img.write(data)
                            img.close()
                        elif xObject[obj]['/Filter'] == '/CCITTFaxDecode':
                            img = open('dataset/' + fileName + "/" + obj[1:] + ".jpg", "wb")
                            img.write(data)
                            img.close()
                    else:
                        img = Image.frombytes(mode, size, data)
                        img.save('dataset/' + fileName + "/" + obj[1:] + ".png")
    else:
        print("No image found.")


Master checkpoint file
----------------------
Next, we create a master checkpoint file named checkpoint.csv in the *fileName* folder, which lists a number of tasks to be done. The initial state of each task is set as False, which represents that the job hasn’t been done yet. ::

    file_path = []
    state = []
    
    if os.path.exists(fileName + "/checkpoint.csv"):
        os.remove(fileName + "/checkpoint.csv")
    
    outputFile = os.listdir('output/' + fileName)
    
    for file in outputFile:
        if not file.endswith('.DS_Store'):
            os.remove('output/'+ fileName +'/' + file)
    
    directory = os.getcwd() + "/dataset" + "/" + fileName
    print(directory)
    for j in [n for n in os.listdir(directory) if (os.path.isfile(os.path.join(directory, n)) and n != ".DS_Store")]:
        file_path.append(directory + "/" + j)
        state.append("F")
    
    dataframe = pd.DataFrame({'File Path': file_path, 'State': state})
    
    dataframe.to_csv(fileName + "/checkpoint.csv",index = False ,sep = ',')


Child checkpoint_TODO file
--------------------------
Besides, we assign equal number of tasks to each processor according to the number of processors that we have entered in the previous command line. The list of tasks to be done in each processor are recorded in child checkpoint files named checkpoint_i_TODO.csv. i represents the ID of the processors. These files are also in the *fileName* folder.  ::

    with open(fileName + '/checkpoint.csv', 'r') as f:
        reader = csv.reader(f)
        rows = [row for row in reader if row[1] == "F"]
    fileNum = len(rows)
    
    numOfjobs = fileNum // processor
    
    remainder = fileNum % processor
    cur = 0
    for i in range(remainder):
        with open(fileName + "/checkpoint_" + str(i) + "_" + "TODO" + ".csv", "w") as c1:
            writer = csv.writer(c1)
            writer.writerow(["File Path"])
            for j in range(numOfjobs + 1):
                writer.writerow([rows[cur][0]])
                cur += 1
    
    for i in range(remainder, processor):
        with open(fileName + "/checkpoint_" + str(i) + "_" + "TODO" + ".csv", "w") as c2:
            writer = csv.writer(c2)
            writer.writerow(["File Path"])
            for j in range(numOfjobs):
                writer.writerow([rows[cur][0]])
                cur += 1


