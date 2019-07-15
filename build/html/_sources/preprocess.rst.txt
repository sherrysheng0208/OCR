Preprocess
==========
Master checkpoint file
----------------------
In this part, we create a master checkpoint file named checkpoint.csv, which lists a number of jobs to be done. The state of each job is set as False, which represents that the job hasn't been done yet. ::

 import pandas as pd
 import os
 import sys
 import csv

 file_path = []
 state = []

 if os.path.exists("checkpoint.csv"):
     os.remove("checkpoint.csv")

 outputFile = os.listdir('output/pdf_2')

 for file in outputFile:
     if not file.endswith('.DS_Store'):
         print(file)
         os.remove('output/pdf_2/' + file)
 print(os.listdir('output/pdf_2'))

 directory = os.getcwd() + "/dataset"
 for i in [name for name in os.listdir(directory) if os.path.isdir(os.path.join(directory, name))]:
     newDir = directory + "/" + i
     for j in [n for n in os.listdir(newDir) if (os.path.isfile(os.path.join(newDir, n)) and n != ".DS_Store")]:
         file_path.append(newDir + "/" + j)
         state.append("F")

 dataframe = pd.DataFrame({'File Path': file_path, 'State': state})

 dataframe.to_csv("checkpoint.csv",index = False ,sep = ',')

Child checkpoint file
---------------------
Besides, we assign the same number of jobs to each processor according to the number of processors that we set in the previous slurm file. The list of jobs in each processor are recorded in child checkpoint files named checkpoint_i_TODO.csv. ::

 processor = int(sys.argv[1])

 with open('checkpoint.csv', 'r') as f:
     reader = csv.reader(f)
     rows = [row for row in reader if row[1] == "F"]
 fileNum = len(rows)

 numOfjobs = fileNum // processor
 remainder = fileNum % processor
 cur = 0
 for i in range(remainder):
     with open("checkpoint_" + str(i) + "_" + "TODO" + ".csv", "w") as c1:
         writer = csv.writer(c1)
         writer.writerow(["File Path"])
         for j in range(numOfjobs + 1):
             writer.writerow([rows[cur][0]])
             cur += 1

 for i in range(remainder, processor):
     with open("checkpoint_" + str(i) + "_" + "TODO" + ".csv", "w") as c2:
         writer = csv.writer(c2)
         writer.writerow(["File Path"])
         for j in range(numOfjobs):
             writer.writerow([rows[cur][0]])
             cur += 1

