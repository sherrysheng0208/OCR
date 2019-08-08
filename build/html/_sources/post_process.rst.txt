Post-process
============
Get Processor ID
----------------
Same as before, each processor first get its own ID number so that only one specific processor will run the post-process. ::

    from optparse import OptionParser, Option, OptionValueError

    def list_callback(option, opt, value, parser):
            setattr(parser.values, option.dest, value.split(','))

    parser = OptionParser()
    parser.add_option("-p", action="store", type="str", dest="proc_id")
    (options, args) = parser.parse_args()
    proc_id = str(options.proc_id)


Update checkpoint file
----------------------
We will read the checkpoint_i_DONE.csv of each processor and write the tasks that have been done back into the master checkpoint file. The state of those tasks will be changed from False to True. ::

    def update_checkpoint(file_path, processor_count):
        dict1 = {}
        dict2 = {}
    
        for i in range(processor_count):
            with open(file_path + "_Folder/checkpoint_" + str(i) + "_DONE.csv", "r", encoding="utf-8") as f:
                reader = csv.reader(f)
                names = [row for row in reader]
                del names[0]
    
                dict1.update({names[i][0]: "T" for i in range(len(names))})
    
        with open(file_path + "_Folder/checkpoint.csv", "r", encoding="utf-8") as f:
            reader = csv.reader(f)
            names = [row for row in reader]
            del names[0]
            dict2 = {names[i][0]: names[i][1] for i in range(len(names))}
    
        for j in dict1.keys() & dict2.keys():
            dict2[j] = "T"
    
        with open(file_path + '_Folder/checkpoint.csv', 'w') as f:
            writer = csv.writer(f)
            writer.writerow(["File Path", "state"])
            writer.writerows(dict2.items())


post_process.py
---------------
We run post_process.py on Processor #0 to update all the checkpoint files.  ::

    import csv
    import sys
    import os
    
    processor_count = int(sys.argv[1])
    directory = str(sys.argv[2])

    def post_process(rootDir, processor_count):
        for lists in os.listdir(rootDir):
            if not lists.startswith("."):
                path = os.path.join(rootDir, lists)
                if os.path.splitext(lists)[-1] == '.pdf':
                    file_path = os.path.splitext(path)[0]
                    if os.path.exists(file_path + "_Folder"):
                        update_checkpoint(file_path, processor_count)
                        os.remove(path)
                if os.path.isdir(path) and not path.endswith("_Folder"):
                    post_process(path, processor_count)
    if proc_id == "0":
        post_process(directory + '_input', processor_count)




