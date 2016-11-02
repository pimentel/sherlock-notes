# sherlock notes

## connecting

Add the following to your `~/.ssh/config`:

```
Host sherlock sherlock.stanford.edu sherlock* sherlock*.stanford.edu
GSSAPIDelegateCredentials yes
GSSAPIAuthentication yes
HostName sherlock
User hjp
Port 22
```

(replacing the `User`).

Make sure to be connected to the 'Secure' network or be connected to the VPN.
Once you are on the secure network, get a `Kerberos` token which should last a few hours (unless you restart your machine).

Now you can finally login with `ssh sherlock`.

## installing software

Notes on installing software can be found here:

http://sherlock.stanford.edu/mediawiki/index.php/Software

### the skinny on software

- `module available` to get a list of all the software that is installed
- `module load SOFTWARE_NAME` to load the package `SOFTWARE_NAME`
- `module spider SEARCH_TERM` to search for a specific package or load more information about a specific package
- `module keyword KEYWORD` to do a more permissive search amongst packages for `KEYWORD`

Example:

```
$ module spider python

------------------------------------------------------------------------------------------------------------
  python:
------------------------------------------------------------------------------------------------------------
     Versions:
        python/2.7.5
        python/3.3.2
$ module spider python/3.3.2

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  python: python/3.3.2
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

    This module can be loaded directly: module load python/3.3.2

    Help:
        This module provides support for the
        Python 3.3.2 via Redhat Software Collections.
$ module keyword python

---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
The following modules match your search criteria: "python"
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

  gsutil: gsutil/120.0
    gsutil is a Python application that lets you access Cloud Storage from the command line.

  neon: neon/1.4.0
    neon is Nervana's Python-based deep learning library. It provides ease of use while delivering the highest performance

  py-cython: py-cython/0.24

  py-numpy: py-numpy/1.10.4

  py-psycopg2: py-psycopg2/2.6.1
    PostgreSQL adapter for Python

  py-scikit-image: py-scikit-image/0.11.3

  py-scipy: py-scipy/0.17.0

  python: python/2.7.5, python/3.3.2
```

### installing snakemake

snakemake is simply a python 3 package.
You can install it locally after getting sad about the default python version and that there is no python 3 already in your path:

```
$ python --version
Python 2.6.6
$ python3
-bash: python3: command not found
```

```
$ module spider python

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  python:
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
     Versions:
        python/2.7.5
        python/3.3.2

--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
  For detailed information about a specific "python" module (including how to load the modules) use the module's full name.
  For example:

     $ module spider python/3.3.2
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
```

Load the python module:

```
$ module load python/3.3.2
````

```
$ pip3 install --user snakemake
[...]
```

## snakemake vs using SLURM directly 

The queuing system used is SLURM.

...but I would say, don't toy around with that -- just use snakemake for everything!

benefits:
- keeping track of what jobs have completed
- automatically requesting the appropriate amount of CPU
- unless you are doing a 1 off script, less boilerplate

drawbacks:
- some certain types of jobs don't lend themselves well to the snakemake framework

## submitting jobs

If you do want to use SLURM directly the documentation for submitting simple bash scripts is pretty good and can be found [here](http://sherlock.stanford.edu/mediawiki/index.php/SLURMSubmit).

Some additional points: 
- you can specify which queue you want to run jobs on using the -p option to sbatch. The default is `normal`, there will be some name for our queue ('pritch'?), 'gpu' is probably self explanatory, and 'owners' allows you to submit jobs on other labs cluster allocation, with the caveat that your job will be cancelled (and later rescheduled) if that lab needs the compute. You can specify multiple queues! In which case SLURM will use the first available. 
- Two useful ways of running sets of jobs are array jobs and using the --export option. Using e.g. --array=1-10 as an option to sbatch will submit 10 jobs with an environment variable SLURM_ARRAY_TASK_ID set to corresponding job index. Alternatively you can call sbatch multiple times using e.g. --export=MYINDEX=1 etc, which will create an environment variable MYINDEX set to 1 for that job. 

## Interactive jobs

Running `sdev` gets you an interactive job. Annoyingly the default run time for these is only an hour, and the max is 2h [use -t=2:00:00 to get this]. These instances also don't get much RAM (and no GPU). 

## Storage 

The storage options are very well documented here:
http://sherlock.stanford.edu/mediawiki/index.php/DataStorage

## Using the GPU 

Sherlock is very well set up for using their/our GPU machines. 

To have stuff setup for theano I have the following in my .bash_profile: 
```
ml load cuDNN/v4
ml load theano
ml load openblas/0.2.15
alias sgpu="srun -p gpu,owners --qos gpu --gres gpu:1 --pty bash"
```

The last line creates an alias to request an iteractive GPU job. Note these don't tend to get scheduled until you leave the lab :/
