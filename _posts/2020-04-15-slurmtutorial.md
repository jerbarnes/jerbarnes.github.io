---
layout: archive
title: "A crash-course in using SLURM"
date: 2020-04-15
---

**Slurm** is a workload manager for distributed systems.


The most important commands
---

```
squeue -u USERNAME
squeue -p PARTITION
sbatch SCRIPT
scancel SLURM_ID
```

squeue -u USERNAME will tell you what jobs are running for the user. You can keep track of your jobs this way, although it only tells you which ones are running. You can look at what is happening (to a degree) by checking the log files which appear in your home directory (or somewhere else if you specify it). If you're running a python script and want to monitor what's happening, it's better to use [logging](https://docs.python.org/2/howto/logging-cookbook.html) rather than print statements, as the print statements only appear in the log file after the whole script is finished running.

squeue -p PARTITION will show you all the jobs being run on a specific partition. In the case of Saga, we usually stick to normal and accel. Accel has GPUs, so if you're running a compute heavy job, it's probably better.

sbatch SCRIPT will run a slurm job that you have set up. This also give you the SLURM_ID for the job. If you need to cancel the job for some reason, run scancel SLURM_ID


Importing / Setting up necessary libraries and dependencies
---

Saga already has most of the dependencies you need installed (Pytorch, Sklearn, etc). You can check available modules [here](http://wiki.nlpl.eu/index.php/Infrastructure/software/catalogue).

To upload a module, either in an interactive session (when you first log on), do:

```
module load MODULENAME
```


Example slurm job
---

```
#!/bin/bash
#
#SBATCH --job-name=bert_finetune --account=nn9447k  #change the job-name
#SBATCH --output=bert_finetune-%j.out               #name of log file
#
#SBATCH --partition=normal                          #normal or accel
#SBATCH --time=48:00:00                             #how long to run
#
#
# Max memory usage:
#SBATCH --mem-per-cpu=100G
#
# Number of threads per task:
#SBATCH --cpus-per-task=1

## Set up job environtment:
# source /cluster/bin/jobsetup
module restore system                              #clear any inherited modules
set -o errexit                                     #exit on errors

# List imported modules
module list

# Load models
module load Python/3.6.6-fosscuda-2018b

## In addition to the models already available, sometimes I need extra stuff.
## I set up virtual environments and use pip --user install X to get what I
## need.

# load virtual environment
source /cluster/home/jeremycb/doc_level_transfer_hierarchical/my_virtualenv/bin/activate


# Run the python script

python task_tuning.py --data_dir="./data" --bert_model="bert-base-cased" --output_dir="task_tuned/ontonotes-reuters-50000" --trained_model_dir="lm_output/ontonotes-reuters" --do_train --do_eval --save_all_epochs --num_training_examples=50000

```

