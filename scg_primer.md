# SCG Primer 
Contact: Nicole Gay nicolerg@stanford.edu

## Table of Contents
  - [Other resources](#other-resources)
  - [Login nodes](#login-nodes)
  - [Directories](#directories)
      - [Home directory](#home-directory-sunetid)
      - [Lab directory](#lab-directory)
      - [Scratch space](#scratch-space) 
  - [Compute partitions](#compute-partitions)
      - [FREE `interactive` partition](#free-interactive-partition)
      - [BILLED `batch` partition](#billed-batch-partition)
      - [`nih_s10` NIH Supercomputer](#nih_s10-nih-supercomputer)
  - [Software modules on SCG](#software-modules-on-scg)
  - [Installing software on SCG](#installing-software-on-scg)
  - [SCG OnDemand](#scg-ondemand)
      - [Interactive RStudio](#interactive-rstudio)
  - [SLURM basics](#slurm-basics)
  - [Miscellaneous tidbits](#miscellaneous-tidbits)

## Other resources 
  - [This Google Doc](https://docs.google.com/document/d/1kTEG6fDjbLhzV7e-ThgpYK3THnoibb9RU57ZLm8EtBs/edit) may also be helpful, but everything looks prettier with Markdown   
  - SCG docs: https://login.scg.stanford.edu/  
  - SCG Slack: https://susciclu.slack.com **If you use SCG, I highly recommend joining this Slack Workspace.** While it’s not quite as helpful now that John Hanks is gone (sob), you may still get responses from the SCG community if you post a question or error that a Google search didn’t help you answer. It’s also the place to post software installation requests (`#software-install-requests` channel). 
  
## Login nodes 
You start on a login node every time you log in to SCG. Login nodes are the same nodes as the interactive partition, but they still have limited resources (16GB of memory and a restricted number of processes) until you start a session in the interactive partition. Login nodes are meant for navigating directories, starting `screen`/`tmux` sessions, and other non-intenstive, minor processes. 

Because there are several login nodes and screen sessions are only available on the node on which they were initialized, I **highly recommend** adding an alias to your **LOCAL** `~/.bashrc` or `~/.bash_profile` to set a persistent login node (it doesn't matter which one). That way, you will always log into the same login node whenever you connect to SCG, and your `screen` sessions will always be where you expect them. For example:

```bash
echo 'alias scg="ssh SUNETID@login04.scg.stanford.edu"' >> ~/.bash_profile
```
The first time you add this line to your `~/bash*` file, you have to run `source ~/.bash_profile` for the alias to register. After that, the `scg` command will be recognized every time you start Terminal. Then just run `scg` to log in to SCG.  

## Directories 
### Home directory (`~/SUNETID`)
Home directory quota is fixed at 32GB. Just about the only thing that should be there is software that you install.  

### Lab directory 
  - Currently: `/labs/smontgom/`  
  - After April 3, 2020: `/oak/stanford/groups/smontgom`  
  - Most of your files should be in `${LAB_DIR}/SUNETID`. The first time you log into SCG, you will have to make your personal directory in `/labs/smontgom/` 
  - Shared lab data sets are in `${LAB_DIR}/shared` 

### Scratch space  
You can use `/tmp`, but be warned that, like everything else on SCG, nothing is backed up. The hardware for the last scratch space died.  

## Compute partitions 

### FREE `interactive` partition 
**This is where you should do the vast majority of your computation.** 16 cores, 128GB total are available for all running interactive jobs **PER PERSON**. You can split up those resources any way you would like. This is more than each person can politely use on `durga`.  

Launch a process in the `interactive` partition after logging in. There are two ways to do this: 

#### Interactive session like durga 
1. Start a `screen`/`tmux` session (you should do this for any process that you expect to take more than a minute in case your connection to SCG is interrupted)  
2. Launch a job in the `interactive` partition
    ```bash 
    sdev -c 1 -m 20G -t 24:00:00
    ```
    - `sdev` is a shortcut for `srun`, which is a SLURM command 
    - `-c`: number of cores
    - `-m`: memory (`M` or `G` suffix) 
    - `-t`: time (format `DD-HH:MM:SS`) 

Once the resources are allocated, you essentially get `ssh`-ed into a new bash session with the requested resources. Then you can start running scripts (almost) just like you would with `durga`. 

#### Submit a job to the `interactive` session with `sbatch` 
If you are running polished code or a standard pipeline that you don't need to worry about debugging, you may prefer to submit a job to the `interactive` job queue using `sbatch`. 

First, write an `sbatch` script, e.g. `test_sbatch.sh`:
```bash 
#!/bin/bash

# See `man sbatch` or https://slurm.schedmd.com/sbatch.html for descriptions
# of sbatch options.
#SBATCH --job-name=test
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --partition=interactive
#SBATCH --account=default
#SBATCH --time=12:00:00

# by default, log files are written to the pwd 

set -e 
module load miniconda/3
python some_python_script.py # this is the process I want to run with the sbatch script 
``` 
Then submit the job using `sbatch test_sbatch.sh`. **It is critical that the `--partition` flag is set to `interactive` if you don't want to get billed.** 

### BILLED `batch` partition 
**You must get clearance from Stephen before running any jobs on the `batch` partition, which should be reserved for parallelizations beyond 16 cores or processes requiring more than 128GB of memory.** For the most part, you can pretend this option doesn't exist. 

You submit jobs to the `batch` partition the same way, with a couple of added flags:  

#### Interactive session like durga ON THE BILLED `batch` PARTITION 
1. Start a `screen`/`tmux` session (you should do this for any process that you expect to take more than a minute in case your connection to SCG is interrupted)  
2. Launch a job in the `batch` partition
    ```bash 
    sdev -c 1 -m 20G -t 24:00:00 -a smontgom -p batch
    ```
    - `sdev` is a shortcut for `srun`, which is a SLURM command 
    - `-c`: number of cores
    - `-m`: memory (`M` or `G` suffix) - bump this up if you get a core dump or out-of-memory error
    - `-t`: time (format `DD-HH:MM:SS`)
    - `-a`: account (for us, `smontgom`)
    - `-p`: partition, either `interactive` (default, free) or `batch` (billed; requires `-a`)

#### Submit a job to the BILLED `batch` session with `sbatch` 
If you are running polished code or a standard pipeline that you don't need to worry about debugging, you may prefer to submit a job to the `batch` job queue using `sbatch`. 

First, write an `sbatch` script, e.g. `test_sbatch.sh`:
```bash 
#!/bin/bash

# See `man sbatch` or https://slurm.schedmd.com/sbatch.html for descriptions
# of sbatch options.
#SBATCH --job-name=test
#SBATCH --nodes=1
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --partition=batch
#SBATCH --account=smontgom
#SBATCH --time=12:00:00

# by default, log files are written to the pwd 

set -e 
module load miniconda/3
python some_python_script.py # this is the process I want to run with the sbatch script 
``` 
Then submit the job using `sbatch test_sbatch.sh`. 

### `nih_s10` NIH Supercomputer
Unless you're doing something really intensive like deep learning or modeling molecular dynamics, I don't know why you'd use this. You can read more about it [here](https://login.scg.stanford.edu/uv300/).

## Software modules on SCG 
`module load <module>` is your friend. If you can Google the bioinformatics tool, chances are SCG has already installed the software, and it’s loaded in a module. if you see `command not found`, try loading a module.  

Some `module` commands:
  - `module avail`: Get a (long) list of existing modules; use arrow keys to scroll; `q` to exit
  - `module keyword <keyword>`: Search for modules containing a keyword; use arrow keys to scroll; `q` to exit 
  - `module unload <module>`: Unload a module 
  - `module purge`: Unload all modules (i.e. revert back to login state)
  
*How can you find files associated with a module after you load it?*   
Almost all module add the path to the modules programs/scripts to the `PATH` variable, so this will show you that entry:
`echo $PATH | tr ':' '\n'`

A few of the most critical ones, for example:  
  - BEFORE trying to run `python`:
  ```bash
  module load miniconda/2 #python2
  module load miniconda/3 #python3
  ```
  - BEFORE trying to run `R` or `Rscript`:
  ```bash
  module load r/3.6
  module load r/3.5 
  ```
  - To load outdated modules, like older versions of R, run `module load legacy` first; then, for example, `module load r/3.4`
  
## Installing software on SCG 
Always [check if a module exists](#software-modules-on-scg) before installing software on SCG. If you're really sure you need to install it, there are a few ways to do it:  
- For `R` packages:
    - Load the `r` module corresponding to the version in which you want to install the package (e.g. `module load r/3.6`) 
    - Launch `R` and install packages as usual (e.g. with `install.packages()`). The first time you do this, answer `yes` to let it make a library in your home directory.  
    - Extra tidbit: Use the `.libPaths()` command in `R` to see which paths `R` is looking in to load libraries  
- For `python` modules:  
    - Load the `python` module corresponding to the version in which you want to install the package (e.g. `module load miniconda/3`)  
    - Use `pip install --user MODULE` to install a module locally. Note you will need to load the same module before trying to import this module in the future (e.g. `module load miniconda/3`)  
- For anything else: Install it as you would on durga, either in your home directory or in a `SOFTWARE` subfolder in your lab directory. 

For particularly tricky installations, or just for anything you think might be useful for anyone else on SCG, add a software installation request to the `#software-install-requests` channel in [SCG's Slack Workspace](https://susciclu.slack.com). I tend to install software myself and also add a request to the channel for anything that's not already installed on SCG.  
  
## SCG OnDemand
I <3 SCG OnDemand: https://ondemand.scg.stanford.edu/pun/sys/dashboard 
  - `Files` tab lets you do file I/O in your home or `/labs/smontgom` paths 
  - `Interactive Apps` lets you run RStudio, Jupyter Notebooks, and other tools interactively while using SCG file systems and compute resources 
  
### Interactive RStudio
By default, an `.rstudio` folder is created in your home directory the first time you start an RStudio session. For whatever reason, this can be REALLY slow (something about file I/O in the file system used for home directories). If you start noticing that basic commands in RStudio are being slow, do the following:
1. Kill your sessions
2. `mv ~/.rstudio ~/.rstudio-backup && ln -s /tmp ~/.rstudio`
3. Start a new session. Things should be faster now! (the file system that uses `/tmp` is faster)

## SLURM basics
SLURM is the job scheduler that SCG uses. See SCG documentation of SLURM basics [here](https://login.scg.stanford.edu/tutorials/job_scripts/). Here are a few commands to know:

### `squeue`
`squeue -u SUNETID` gives a list of the jobs currently running under your name (`interactive` or `batch` partition. you can use the `-p` flag to specify, e.g. `squeue -u nicolerg -p batch`)

### `sacct`
`sacct -j JOBID`, for example, tells you the amount of resources used for recent jobs.  

This is particularly important if you are planning to run a large number of jobs on the BILLED `batch` partition - run one job first, see how many resources it required, and limit the resources you request for your batch of jobs. This is ESPECIALLY important for number of CPUs requested since SCG charges per CPU hour. If you request 2 CPUs but your process only uses 1, you still pay for those 2 CPUs as long as your job runs. SCG charges based on actual CPU time, NOT total time requested (i.e. request as much time as you want, with the knowledge that longer jobs will sit in the queue longer, but be thoughtful with your CPU requests).  

eg: `sacct -u nicolerg -o JOBID,JobName,MaxRSS,nCPUs,CPUTime --starttime 03/20`

Use the `-o` flag to change the format. e.g.:
  - `MaxRSS`: Roughly the memory required (request somewhat more than this)
  - `nCPUs`: Number of CPUs requested
  - `CPUTime`: Time billed by SCG

Use the `--starttime` flag to change the time range in which to show job stats. By default, `sacct` will only show stats for jobs completed since 12:00 AM that day. Extend the time window with this flag (format: `MM/DD[/YY]-HH:MM[:SS]`).  

See other fields here: https://slurm.schedmd.com/sacct.html   

### `scancel`
Kill a job. Use `-j JOBID` for a single job or `-u SUNETID` for ALL of your jobs. 

### Other SLURM tips and tricks
  - `echo $SLURM_JOB_ID` will tell you if you’re inside a job (yes, you might forget when you get lost in the layers of screen sessions and interactive jobs)
  - If a running job looks “stuck”, i.e. it’s been actively running much longer than you would expect it to, run `scontrol requeue JOBID` to “unstick it”. This does NOT restart the job from the beginning
  - To attach to a node running with `srun` (to see what processes are running, for example), run `srun --jobid JOBID --pty bash -l`. This is like SSHing into the node that the job is running on. `exit` will end that SSH session but not your srun-initiated jobs on that node. (Alternatively, you can actually just `ssh` into the node displayed from `squeue`.) 

## Miscellaneous tidbits 
  - Because of NFS things, I recommend adding a `--latency-wait` flag to your calls to `snakemake` pipelines. This means the pipeline will wait up to the specified number of seconds for a file to appear before aborting with an error. 
  - See [this thread](https://susciclu.slack.com/archives/C8CNSTB88/p1550866979024200) in SCG Slack if you would like to keep track of resources for an interactive job. 
  - See [these instructions](https://login.scg.stanford.edu/tutorials/data_management/#samba) for how to mount SCG directories locally with Samba.  
