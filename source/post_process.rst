Post-process
============
Last but not least, we will read the checkpoint_i_DONE.csv of each processor and write the tasks that have been done back into the master checkpoint file. The state of those tasks will be changed from False to True. ::

    import csv
    import sys
    
    dict1 = {}
    dict2 = {}
    processor = int(sys.argv[1])
    fileName = str(sys.argv[2])
    
    for i in range(processor):
        with open(fileName + "/checkpoint_" + str(i) + "_" + "DONE" + ".csv", "r", encoding="utf-8") as f:
            reader = csv.reader(f)
            names = [row for row in reader]
            del names[0]
    
            dict1.update({names[i][0]: "T" for i in range(len(names))})
    
    with open(fileName + "/checkpoint.csv", "r", encoding="utf-8") as f:
        reader = csv.reader(f)
        names = [row for row in reader]
        del names[0]
    
        dict2 = {names[i][0]: names[i][1] for i in range(len(names))}
        # print(dict2)
    
    for j in dict1.keys() & dict2.keys():
        dict2[j] = "T"
    
    with open(fileName + '/checkpoint.csv', 'w') as f:
        writer = csv.writer(f)
        writer.writerow(["File Path","state"])
        writer.writerows(dict2.items())
