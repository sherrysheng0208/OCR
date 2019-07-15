Run pre-process on HTC
======================
To run on high-performance computers, we need to run slurm file. Assume we want it to run on 30 processors. ::

 #!/bin/bash
 #SBATCH --job-name=pre_process
 #SBATCH --partition=commons
 #SBATCH --ntasks=1
 #SBATCH --time=00:30:00
 #SBATCH --export=ALL
 #SBATCH --mail-user=  ##Enter your email
 #SBATCH --mail-type=ALL

 module load GCC/7.3.0 OpenMPI/3.1.2
 module load Python/3.6.6
 pip3 install --user pandas

 python imageCleaning.py
 python pre_process.py 30 ##The last number is the number of the processors
