Post-process
============

Last but not least, we should write the jobs that have been done back into the master checkpoint file and change their state from False to True. ::

 dict2 = {}
 with open("checkpoint.csv", "r", encoding="utf-8") as f:
     reader = csv.reader(f)
     names = [row for row in reader]
     del names[0]

     dict2 = {names[i][0]: names[i][1] for i in range(len(names))}

 for j in dict1.keys() & dict2.keys():
     dict2[j] = "T"

 with open('checkpoint.csv', 'w') as f:
     writer = csv.writer(f)
     writer.writerow(["File Path","state"])
     writer.writerows(dict2.items())

