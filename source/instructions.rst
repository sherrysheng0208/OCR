Instructions to run the program
===============================
Input files
-----------
First, you need to use scp to transfer input files to HTC from your local machine. 

Make directory
--------------
Then, you will find out that there are two folders called dataset and output in the current HTC directory. Create one folder in each of these two directories and also in your current directory. Make sure the names of the three folders are the same. They should be the name of your PDF file so that it will be easier for you to find them. We will use *fileName* to represent the name of your PDF file in the following instructions.

The folder in the dataset directory is to store the split images from pre-process, the folder in the output directory is to store the output texts from OCR and the one in the current directory is to store master and child checkpoint files and also benchmark.

Run in one command line
-----------------------
After making directories, you only need to run one command line to specify the name of your PDF file and how many processors you want to use. ::

 sbatch —array=array_index myjob.slurm #processors fileName

For example, assume your PDF file is document.pdf. You want it to run on 10 processors and the index of these 10 processors is 0-9. You should use sbatch as follows: ::

 sbatch —array=0-9 myjob.slurm 10 document

If you submit the job successfully, it will show: ::

 Submitted batch job job_ID.

myjob.slurm
-----------
myjob.slurm mentioned in the above command line is a slurm script used to run on HTC. It installs the packages we need in the python programs and run the python programs in sequence. ${1} represents the first input in the above command line, which is the number of processors. ${2} represents the second input in the above command line, which is the name of your PDF file. $SLURM_ARRAY_TASK_ID is the ID of processors. These arguments passed by command line will play an important role in each python program. ::

 #!/bin/bash
 #SBATCH --job-name=ocr
 #SBATCH --partition=commons
 #SBATCH --ntasks=1
 #SBATCH --time=00:30:00
 #SBATCH --export=ALL
 #SBATCH --mail-user=sz58@rice.edu
 #SBATCH --mail-type=ALL
 #SBATCH --array=1-${1}

 module load GCC/7.3.0 OpenMPI/3.1.2
 module load Python/3.6.6
 pip3 install --user pandas
 pip3 install --user pytesseract
 pip3 install --user PyPDF2

 sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"

 echo "export PATH=/home/yourID/.linuxbrew/bin/:$PATH" >> ~/.bash_profile && source ~/.bash_profile

 brew install tesseract

 python pre_process.py ${1} ${2}
 python ocr.py ${2} -p $SLURM_ARRAY_TASK_ID
 python post_process.py ${1} ${2}

Output
------
The time for completing the jobs depends on the size of your PDF files and the number of processors you use. Please check the master checkpoint file in the *fileName* folder to see whether there is any unfinished tasks after it completes. You can use scp to transfer the output texts in the folder output/*fileName* back to your local machine.
