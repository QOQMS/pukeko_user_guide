# Introduction

This guide provides a comprehensive overview of how to access and use Pukeko, Research Computing facilities to the [QOQMS group](http://qoqms.phys.strath.ac.uk/). 

The setup of Pukeko resembles [ARCHIE-WeSt](https://www.archie-west.ac.uk/) that is why this guide will often refer to the [ARCHIE-WeSt user guide](http://docs.hpc.strath.ac.uk/user-guide). Ideally, each unexperienced user should finish the ARCHIE-WeSt [training session](http://docs.hpc.strath.ac.uk/request/).

Pukeko has the following nodes: 

* `pukeko` is the interactive login and compute node
    * 20 cores
    * 128 GB RAM

* `pukeko-int` is the interactive compute node
    * 24 cores
    * 128 GB RAM

* `pukeko-fs` is the filesystem node with the job queueing system (scheduler) [Slurm](https://slurm.schedmd.com/). 
    * 16 cores
    * 32 GB RAM

*  `pukeko-[2-11]` are the compute nodes of the queueing system
    * `pukeko-[2,3,5,6]`
        * 20 cores per node
        * 64 GB RAM per node
    * `pukeko-[4,7,8]`
        * 20 cores per node
        * 128 GB RAM per node
    * `pukeko-[9-11]`
        * 24 cores per node
        * 128 GB RAM per node

> **Note:** Full-scale calculations are not permitted on `pukeko-fs`, it's used only for Slurm.

# Accessing Pukeko

Each user should already have a valid account set up by the University of Strathclyde with a user name `username` and password, if not ask Andrew. Linux/Mac/Windows users can log into Pukeko from any network using Secure Shell. The general procedure is similar to [accessing ARCHIE-WeSt](http://docs.hpc.strath.ac.uk/user-guide/access/).

To connect to `pukeko` use the following command:
```bash
ssh username@pukeko.phys.strath.ac.uk
```

To connect to `pukeko-int` or `pukeko-fs` you should then enter: `ssh pukeko-int` or `ssh pukeko-fs` from the logged in session on `pukeko`. 


> **Note:** On campus and in the university VPN users can directly connect to `pukeko-fs` using:
```bash
ssh username@pukeko-fs.phys.strath.ac.uk
```

> **Note:** In order to display programs graphically supply `-X` to the ssh command:
```bash
ssh -X username@pukeko.phys.strath.ac.uk
```

# Pukeko Storage & Data Transfer

## Storage Overview

All nodes of Pukeko have access to the shared memory:

* `/home` is the default home directory, 6.6 TB
* `/raid` is the secondary storage, 3.6 TB

and local temporary storages:

* `/stratch`, 500 GB per each node

> **Note:** The shared memory can be accessed from all nodes of `pukeko`, whereas the local storages are individual for each node. 

To check the file system disk space usage run:
```bash
df -h
```

To check the size of the current directory run:
```bash
du -sh
```

To check your local memory run:
```bash
du -h --max-depth=1 ~
```

> **Note:** Users should aim to share memory fairly and aim to use not more that 500 GB. Please let other users know if more memory is needed.

## Data Transfer

See [Storage and Data Transfer to/from ARCHIE-WeSt](http://docs.hpc.strath.ac.uk/user-guide/storage/) for general practices of data transfer on Linux/Mac/Windows as well as connection with other university data storage facilities.

Besides the mentioned above methods of data transfer users should also learn to use `rsync`, fast and smart way of transferring by synchronizing. It power comes while working with big data, especially when only essential information needs to be transfered. Comprehensive guides and examples can be found on the web, see for instance [here](https://www.linuxtechi.com/rsync-command-examples-linux/). 

The following example command transfers only *new* `mat` files from Pukeko to the current local directory:
```bash
rsync -avm --include='*.mat' --include='*/' --exclude='*' username@pukeko.phys.strath.ac.uk:source_directory/ ./
```
i.e. not only other files are not transfered but also previously transfered `mat` files are not transfered again.

> **Note:** Linux and Mac syntax of `rsync` is slightly different.

# Environment Modules

Pukeko uses environment modules to let users reproducibly use software of desired version. This system is identical to the one on ARCHIE-WeSt thus users should take a look at this part of the [guide](http://docs.hpc.strath.ac.uk/user-guide/modules/). 

> **Note:** The list of available software on Pukeko is shorter than on ARCHIE-WeSt, but it includes all the packages and libraries currently used in the group. If the user requires a module, which does not exist please contact Andrew to enable installation.

# Interactive Regime

Users can run full scale calculations using interactive nodes `pukeko` and `pukeko-int`. Below we provide a couple of instructive examples.

## Python Example

First, you should log in either of them and open/create a directory to practice. In the created directory create a Python file `example1.py`:

```python
print('Hello World!')
a = 1
b = 2
print(a, b, a + b)
```

Then load the appropriate module:
```bash
module purge
module load python/miniconda3
```

> **Note:** In order to locate Python run:
```bash
which python
```

> **Note:** In order to check the version of Python run:
```bash
python --version
```

The next step is to specify the number of threads to use in parallel regions of your code. This simple example does not use parallelization, but it is a good practice to always add the following lines while working with Python, C/C++, and Fortran. In this case parallelization is not needed, so define all threads to 1: 
```bash
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1
```

> **Note:** Depending on the parallelization method you need to set either or both thread parameters greater than 1. Alternatively these parameters can be defined inside of the Python script, please refer to the Internet for a manual.

The simplest way to run this job is by typing:
```bash
python example1.py
```

In case of success the following should appear on the screen:
```
Hello World!
1 2 3
```

> **Note:** The job will be terminated if the connection with Pukeko is lost.

In case of long calcualtions the job can be sent to the background and the terminal can be disconnected from Pukeko and closed. For this you can use `nohup` by running:
```bash
nohup python -u example1.py </dev/null >nohup.out 2>nohup.err &
```
then the standard output will be saved in `nohup.out` and the file with error messages `nohup.err` should be empty. Here `</dev/null` simply defined standard input of `nohup` to be empty and `-u` stops buffering of standard input and outputs, i.e. `nohup.out` and `nohup.err` will be updated live.

> **Note:**  For further reading about keeping remote SSH sessions and processes running after disconnection using `nohup` and `disown` refer to [this](https://support.ehelp.edu.au/support/solutions/articles/6000089713-tips-for-running-jobs-on-your-vm) and [that](https://www.tecmint.com/keep-remote-ssh-sessions-running-after-disconnection/).



## Matlab Example

The example with Matlab is very similar, the main difference is that Matlab uses it's own internal way of parallelizing calculations, thus then number of computation threads is defined differently.

We start by creating a Matlab file `example1.m`:

```matlab
disp('Hello World!')
a = 1;
b = 2;
disp([a, b, a + b])
```

Then load the appropriate module:
```bash
module purge
module load matlab/R2018b
```

> **Note:** For Matlab jobs we do not need to define `OMP_NUM_THREADS` and `MKL_NUM_THREADS`.

Then simply run:
```bash
matlab -nodisplay -nodesktop -nojvm -singleCompThread -r "example1; exit"
```
Here the job has all visualization and parallelization turned off, also note `exit` in the end of the command.

For sending the same job to the background run:
```bash
nohup matlab -nodisplay -nodesktop -nojvm -singleCompThread -r "example1; exit" </dev/null >nohup.out 2>nohup.err &
```

For parallelization `-singleCompThread` should be removed. In case if the number of threads is not defined the it is set to the number of physical cores of the node:
```bash
nohup matlab -nodisplay -nodesktop -nojvm -r "example1; exit" </dev/null >nohup.out 2>nohup.err &
```

> **Note:** Make sure there are no other jobs running on the node in this case as then might compete for resources.

In order to define the number of threads use `maxNumCompThreads`:
```bash
nohup matlab -nodisplay -nodesktop -nojvm -r "maxNumCompThreads(N); example1; exit" </dev/null >nohup.out 2>nohup.err &
```
where `N` is the number of threads not greater than the number of cores.

> **Note:** Refer to the [Matlab manual](https://uk.mathworks.com/help/matlab/ref/maxnumcompthreads.html) for running parallel jobs.

## Monitoring Tasks

Jobs sent to the background using `nohup` should be monitored for two reasons:
1. The total number of compute threads should not be greater than the number of cores on the node.
2. The memory should be fairly shared between users and must not exceed the total amount.

The commands `top` and `free` show active processes and memory usage on the current machine. For more reading refer to [here](https://www.linux.com/learn/5-commands-checking-memory-usage-linux).

> **Note:** It is a good practice to monitor your jobs once started and at some time later.

# Queuing System Slurm

Jobs on the compute nodes `pukeko-[2-11]` must be submitted via the job queueing system [Slurm](https://slurm.schedmd.com/) while logged in `ruru`. 

> **Note:** Slurm settings are defined in `/ect/slurm/`

While the general guidance on [job submissions](http://docs.hpc.strath.ac.uk/user-guide/jobs/) with variety of [examples](http://docs.hpc.strath.ac.uk/user-guide/example_scripts/) for ARCHIE-WeSt can be used as it is, Pukeko has a few differences:

* There is only one partition (queue) called `qoqms_intel`
* There is no upper time bound on job time
* Hence there is no accounting for used time and no need to define `--account`
* Each user is restricted to using 140 cores at the same time
* The priority multi-factor is set slightly different and can be found in `/etc/slurm/slurm.conf`
* Pukeko hasn't been tested for interactive job on Slurm, so ignore that part

Below we provide working examples for Pukeko with further comments.

## Example Job Scripts

Log in `ruru` and choose the directory with `example1.py` and `example1.m`.

##### Python Job Script

Create a batch file `job_python.batch`:

```bash
#!/bin/bash
#SBATCH --export=ALL
#SBATCH --partition=qoqms_intel
#SBATCH --ntasks=1
#SBATCH --time=00:01:00
#SBATCH --mem=100M
#SBATCH --job-name=test
#SBATCH --output slurm-%j.out
#SBATCH --error  slurm-%j.err

/home/QOQMS/slurm_scripts/job_prologue.sh

module purge
module load python/miniconda3

export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1

python example1.py

/home/QOQMS/slurm_scripts/job_epilogue.sh
```

To submit the job run:
```bash
sbatch job_python.batch
```

##### Matlab Job Script

Create a batch file `job_matlab.batch`:

```bash
#!/bin/bash
#SBATCH --export=ALL
#SBATCH --partition=qoqms_intel
#SBATCH --ntasks=1
#SBATCH --time=00:01:00
#SBATCH --mem=1000M
#SBATCH --job-name=test
#SBATCH --output slurm-%j.out
#SBATCH --error  slurm-%j.err

/home/QOQMS/slurm_scripts/job_prologue.sh

module purge
module load matlab/R2018b

matlab -nodisplay -nodesktop -nojvm -singleCompThread -r "example1; exit"

/home/QOQMS/slurm_scripts/job_epilogue.sh
```

To submit the job run:
```bash
sbatch job_matlab.batch
```

After each job is done check the standard output file and make sure the error file is empty.

> **Note:** For further modifications of batch files please see examples in the [ARCHIE-WeSt documentation](http://docs.hpc.strath.ac.uk/user-guide/example_scripts/). 

## Creation of Multiple Job Scripts

In case if the same simulation has to run with a variety of parameters the process of job submission can be automated. For that we should slightly modify the previous example. 

Let's create 2 new files, `input_pars.py`:

```python
a = 1
b = 2
```

and `example2.py`:

```python
import input_pars

print('Hello World!')
print(input_pars.a, input_pars.b, input_pars.a + input_pars.b)

```

By executing:

```bash
module purge
module load python/miniconda3
export OMP_NUM_THREADS=1
export MKL_NUM_THREADS=1
python example2.py
```

we obtain the same result as for `example1.py`.

> **Note:** The lines with `module` and `export` has to be executed only once for each SSH connection with Pukeko, and we added it here for reminding of its necessity as a good habit.

Now we want to run the same script for different parameters `a` and `b`, for that create a file `make_jobs.py`, which will create an individual directory and batch file for each job:

```bash
import os

# loop over the range of parameters
for a in range(3):
  for b in range(3):
    # create a unique name for each job
    job_name = 'a' + str(a) + 'b' + str(b)

    # create a directory for each job and copy the main script there
    os.system('mkdir -p ' + job_name)
    os.system('cp example2.py ' + job_name)

    # create an input file for each job
    with open(os.path.join(job_name, 'input_pars.py'), "w") as f:
      f.write('a = ' + str(a) + '\n')
      f.write('b = ' + str(b) + '\n')

    # create a batch-file for each job    
    with open(job_name + '.batch', "w") as f:
      f.write('#!/bin/bash\n')
      f.write('#SBATCH --export ALL\n')
      f.write('#SBATCH --partition qoqms_intel\n')
      f.write('#SBATCH --ntasks 1\n')
      f.write('#SBATCH --mem 100M\n')
      f.write('#SBATCH --time 00:01:00\n')
      f.write('#SBATCH --job-name ' + job_name + '\n')
      f.write('#SBATCH --output   ' + job_name + '_%j.out\n')
      f.write('#SBATCH --error    ' + job_name + '_%j.err\n')
      f.write('\n')
      f.write('cd ' + job_name + '\n')
      f.write('\n')
      f.write('module purge\n')
      f.write('module load python/miniconda3\n')
      f.write('\n')
      f.write('export OMP_NUM_THREADS=1\n')
      f.write('export MKL_NUM_THREADS=1\n')
      f.write('\n')
      f.write('/home/QOQMS/slurm_scripts/job_prologue.sh\n')
      f.write('\n')
      f.write('python example2.py\n')
      f.write('\n')
      f.write('/home/QOQMS/slurm_scripts/job_epilogue.sh\n')
```

Run this script:
```bash
python make_jobs.py
```
and check the created directories. Each of the should have a different input file `input_pars.py` as well as the main script `example2.py`.

In order to submit *all* these jobs to the queue system together run:
```bash
for i in *.batch; do sbatch $i; done
```

That is it, all your jobs were submitted and will run as soon as there are available resources.

## Monitoring Jobs

All jobs run on `pukeko-[2-11]` systems should be submitted via the job scheduling system Slurm that also have a monitoring facilities. Please refer to the [ARCHIE-WeSt documentation](http://docs.hpc.strath.ac.uk/user-guide/monitoring/) for a summary.

> **Note:** Also Pukeko has predefined commands `sinfoextra` and `squeueextra` in `/etc/profile.d/slurm-aliases.sh` on `pukeko-fs`. More useful aliases can be added by demand.

> **Note:** Users can add local aliases in `~/.bashrc`. For instance the following alias
```bash
alias squeueextrame='squeueextra -u username'
```
will show the formatted list of jobs only for the user name `username`.

Since sinfoextra is removed, you can put 
```
alias="sinfo -N -o '%12N %.6D %15P %.11T %.4c %15C %.6m %.10e'"
```
in your `.bash_profile`
## Canceling Jobs

If a job or a range of jobs need to be cancelled, because e.g. the jobs will take longer than the specified time in 
```#SBATCH --time 00:01:00```, you can use the following command:
```
scancel  JOB_ID 
```
which will delete the job identified by JOB_ID to prevent these jobs from running until the specified time.

Other useful options for canceling jobs:

By running:
>
``` 
scancel JOB_ID1 JOB_ID2 ... JOB_IDN   
```   
all jobs with job_id's JOB_ID1, JOB_ID1, etc. are deleted.

>
``` 
scancel -u USER_NAME 
```   
all jobs from a given user are deleted. 

>
``` 
scancel -u USER_NAME -t PD/R
```   
all pending (PD) or running (R) jobs from a given user are deleted, simply chose the appropriate flag.

>
``` 
echo {JOB_ID1..JOB_ID2} | xargs scancel 
```  
all jobs with job_id's between JOB_ID1 and JOB_ID2 (both included) are deleted.

For additional information on how to use ```scancel``` refer to this [documentation](https://slurm.schedmd.com/scancel.html). 

# Scalability

The performance of parallelized codes can vary widely depending on the chosen [methods](https://en.wikipedia.org/wiki/Parallel_computing). Please refer to the [ARCHIE-WeSt Scalability page](http://docs.hpc.strath.ac.uk/user-guide/scalability/) for a general quidance and to the [performance characterisation and benchmarking](https://www.epcc.ed.ac.uk/research/computing/performance-characterisation-and-benchmarking) by EPCC for further reading on benchmarking.

In simple words you need a good reason to run your jobs with parallel environment and set `OMP_NUM_THREADS` and `MKL_NUM_THREADS` to larger than 1 or remove `-singleCompThread` from Matlab scripts. In most of the cases it is more beneficial to run several non-parallelized jobs at the same time rather than parallelize them and run consecutively using the same number of cores but significantly longer time.

# Practical Hints & Limitations

Please review the [ARCHIE-WeSt suggestions](http://docs.hpc.strath.ac.uk/user-guide/hints_limits/).

Some of the following points repeat previously said but they are important to memorize:

* Always estimate how much time and memory each job will take. If any of the limits is reached Slurm will terminate the job, i.e. it is better to slightly overestimate the resources. On the other hand you should not overestimate the resources too much because it will lower the priority of the job and it will start later.

* Keep track of your storage and remove unnecessary data.

* Do not connect to `pukeko-[2-11]` for running jobs interactively, use only the queue system on `pukeko-fs` for those nodes.

* Keep the log of all current projects in order to know what you are doing and why. Jobs can run for a long time and one can forget about their true purpose without any notes. Eventually it will decrease the amount of wasted CPU-hours of calculation. 

* One way of storing simulation results is in databases with meta informations regarding each job. Pukeko has SQLite for this purpose, but this is outside of this manual.

* Every imaginable tedious operation with files can be automated by simple bash script. Get familiar with the most general commands bash commands for HPC [here](http://hpc.mediawiki.hull.ac.uk/Training/Linux_-_command_line) and [here](https://portal.tacc.utexas.edu/c/document_library/get_file?uuid=a0c33bf1-b2a4-48b4-a23d-fc8a79c887ec&groupId=13601).

* [SSH login without password](http://www.linuxproblem.org/art_9.html)

* Add alias of this kind to your local machine for
```bash
export pukeko="username@pukeko.phys.strath.ac.uk"
export pukekofs="username@pukeko-fs.phys.strath.ac.uk"
```
replacing `username` with your user name.

* Even though `/home` is backed up, it is advisable to backup the source codes via other services, either provided by the university or externally.

* Since nodes `pukeko-[2-11]` vary with respect to the number of cores and memory it is advisable to submit jobs that require a lot of memory to the nodes with most memory per node. In order to exclude nodes with less memory add the following to the batch file:
```bash
#SBATCH --exclude=pukeko-[2,3,5,6]
```

* Also `--exclude` allows you to restrict your jobs to specific nodes, i.e. to decrease the total number of running jobs for the sake of sharing with other users.

# Administration

In this section we provide useful commands for administration of the cluster for users with sudo rights:

* If a node `pukeko-[X]` is drained for some reason run:
```bash
scontrol update nodename=pukeko-[X] state=idle
```

# Feedback

Please email all typos and suggestions to [anton.buyskikh@strath.ac.uk](anton.buyskikh@strath.ac.uk) or [tomohiro.hashizume@strath.ac.uk](tomohiro.hashizume@strath.ac.uk),  your feedback would be much appreciated.

Pukeko User Guide maintained by [Anton Buyskikh](https://github.com/anton-buyskikh), [Tomohiro Hashizume](https://github.com/zoome0215), [Tom Bintener](https://github.com/tomb14) and [Jorge Yago Malo](https://github.com/jyagom).
