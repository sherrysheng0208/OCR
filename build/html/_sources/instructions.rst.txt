Instructions on running the program
===================================
Input files
-----------
First, you need to use scp to transfer input files from your local machine to clusters. 

Run in one command line
-----------------------
Then, you only need to run one command line to specify the name of your directory and how many processors you want to use. It is possible to use Job Arrays with an exotic numbering scheme, but in this program, you must start your Job Arrays with index=0. ::

 sbatch --array=0-array_index myjob.slurm number_of_processors “your_directory”

For example, assume your directory is “My PDF Files”. You want it to run on 10 processors and the index of these 10 processors is 0-9. You should use sbatch as follows: ::

 sbatch --array=0-9 myjob.slurm 10 “My PDF Files”

If you submit the job successfully, it will show: ::

 Submitted batch job job_ID

myjob.slurm
-----------
myjob.slurm mentioned in the above command line is a slurm script used to run on HTC. It installs the packages we need in the python programs and run the python programs in sequence. ${1} represents the first input in the above command line, which is the number of processors. ${2} represents the second input in the above command line, which is the name of your PDF file. $SLURM_ARRAY_TASK_ID is the ID of processors. These arguments passed by command line will play an important role in each python program. ::

 #!/bin/bash
 #SBATCH --job-name=ocr
 #SBATCH --partition=commons
 #SBATCH --ntasks=1
 #SBATCH --time=00:30:00
 #SBATCH --export=ALL
 #SBATCH --mail-user=your_email
 #SBATCH --mail-type=ALL

 module load GCC/7.3.0 OpenMPI/3.1.2
 module load Python/3.6.6
 pip3 install --user pandas
 pip3 install --user pytesseract
 pip3 install --user PyPDF2

 sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"

 echo "export PATH=/home/yourID/.linuxbrew/bin/:$PATH" >> ~/.bash_profile && source ~/.bash_profile

 brew install tesseract

 python pre_process.py ${1} "${2}" -p $SLURM_ARRAY_TASK_ID
 python ocr.py "${2}" -p $SLURM_ARRAY_TASK_ID
 python post_process.py ${1} "${2}" -p $SLURM_ARRAY_TASK_ID

Output
------
The time for completing the jobs depends on the size of your PDF files and the number of processors you use. Please check master checkpoint files in the folders named after each PDF file in the input directory, which is *your_directory_input/file_name_Folder*, to see whether there is any unfinished tasks after it completes. You can use scp to transfer the output texts in the *your_directory_output* directory back to your local machine.

