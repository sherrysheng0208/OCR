Run OCR on HTC
==============
The number of job array correspond to the number of processors that we set in the previous slurm file. For example, we set the number of processors as 30 in the previous slurm file, then we should set array as 0-29 or 1-30 to ensure there are 30 processors in total. ::

 #!/bin/bash
 #SBATCH --job-name=ocr
 #SBATCH --partition=commons
 #SBATCH --ntasks=1
 #SBATCH --time=00:30:00
 #SBATCH --export=ALL
 #SBATCH --mail-user= ##Enter your email
 #SBATCH --mail-type=ALL
 #SBATCH --array=0-29  ##The number of the processors

 module load GCC/7.3.0 OpenMPI/3.1.2
 module load Python/3.6.6
 pip3 install --user pytesseract

 sh -c "$(curl -fsSL https://raw.githubusercontent.com/Linuxbrew/install/master/install.sh)"

 echo "export PATH=/home/hy31/.linuxbrew/bin/:$PATH" >> ~/.bash_profile && source ~/.bash_profile

 brew install tesseract

 python ocr.py -p $SLURM_ARRAY_TASK_ID
