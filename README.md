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

Make sure to be connected to the 'Secure' network or the VPN.
Get a `Kerberos` token which should last a few hours (unless you restart your machine).

Now you can finally login with `ssh sherlock`.

Note: if you want to log into the same machine because of temporary files (tmux) replace the HostName with a specific node (`sherlock-ln01.stanford.edu`).

## installing software

Notes on installing software can be found here:

http://sherlock.stanford.edu/mediawiki/index.php/Software

### the skinny on software

- `module available` to get a list of all the software that is installed.
- `module load SOFTWARE_NAME` to load the package `SOFTWARE_NAME`.
- `module spider SEARCH_TERM` to search for a specific package or load more information about a specific package.
- `module keyword KEYWORD` to do a more permissive search amongst packages for `KEYWORD`.

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

### updating your $PATH

If you are using any nonstandard Python software, make sure to include the `pip` (local) path into your main path.
Put this into your `~/.bashrc` on the server:

```
export PATH=~/.local/bin:$PATH
```

This won't update automatically.
Either logout/login or `source ~/.bashrc`

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

The `snakemake` binaries get loaded into `~/.local/bin`.
Before you run snakemake, make sure to load the module otherwise you will get very strange errors.
Consider putting it into your `~/.bashrc`.

## fetch (SCP)

The configuration below seems to be all that I need when I am connected via ssh:

- `Hostname`: sherlock
- `Username`: my_username
- `Connect using`: SFTP
- `Password`: empty

fetch and other SFTP clients have remote file edit capabilities which are pretty helpful sometimes.

## snakemake vs using SLURM directly

The queuing system used is SLURM.

...but I would say, don't toy around with that -- just use snakemake for everything!

benefits:
- keeping track of what jobs have completed.
- automatically requesting the appropriate amount of CPU.
- unless you are doing a 1 off script, less boilerplate.
- reproducible workflows that can be shared.

drawbacks:
- some certain types of jobs don't lend themselves well to the snakemake framework.
- some flexibility is lost in specifying resources.

## basic snakemake workflow

Let's forget about Sherlock for a moment... Let's just think about processing workflows.

Snakemake is a workflow system which unfortunately has the word `make` in it, often giving people a false impression it is as annoying as `GNU make`.

- In a directory you have a `Snakefile` which has a bunch of rules.
- Every time you run `snakemake` it checks whether or not the top level dependencies have been met. If they haven't been met, it's then checks the dependencies of those dependencies and builds a dependency graph. The really awesome thing is that it knows what is data parallel and what isn't. It will automatically parallelize the processes that can be parallel without any thought on your end.

### learning by example

Let's say I want to call a script a specific number of times.
I don't want to specify this in the Snakefile, I have some metadata file (`metadata/cool_metadata`) that tells me what arguments I want to use.

While snakemake is a domain specific language, everything that it doesn't interpret as part of the language gets interpreted as standard Python 3.
So if we want to execute things based on our metadata file, we can simply loaded using Python.

Let's look at `example/Snakefile`.

## submitting to a queue via snakemake

Once you have a Snakefile setup, it is simple to submit to a queue:

```
snakemake -p -j 999 --cluster "sbatch -p normal -n 5 -t 3:00:00"
```

This will make a request on the `normal` queue, with 5 tasks per node, with a maximum time of 3 hours.

## general recommendations

- always run snakemake with `-p` which will give you the commands it is going to run (not just the file output).
- always perform a dry run with `-n` / `--dryrun` (`snakemake -p --dryrun`).
- if the project is large, have different directories with different Snakefiles
- consider writing a `run.sh` script that simply has the snakemake command with some default resources.
- the cluster seems to behave strangely if you don't give it an absolute path.
  - I typically like to have a top level `config.py` file with a bunch of global paths/variables.

## checking status of jobs and logs

```
squeue -u $USER
```

snakemake will generate standard out and error logs at `slurm_Y` where `Y` is the job id.

## jobs that shouldn't be submitted to the queue but are part of your workflow

are called `localrules`. Specify them as `localrules` near the top of your workflow:

```
localrules: my_local_rule, my_other_local_rule

rule my_local_rule:
  ...

rule my_other_local_rule:
  ...
```

## job specific resource request

You can tie specific queue requests to a rule with a [cluster configuration](https://bitbucket.org/snakemake/snakemake/wiki/Documentation#markdown-header-cluster-configuration).

You can also [tie specific resources to specific rules](https://bitbucket.org/snakemake/snakemake/wiki/Documentation#markdown-header-resources) (e.g. number of gpus).

## where do lab specific files live?

```
/scratch/PI/pritch
```

## which queue should I submit to?

- `normal` for jobs that take more than a few hours/more than basic resources you might have on your machine
- `dev` for interactive jobs
- `owners`? when we get our machines

Other queues including large memory queues [exist as well](http://sherlock.stanford.edu/mediawiki/index.php/Current_policies#Sherlock_Queue_Structure).

## a note on tmux

A bit of strange behavior which I am trying to debug.
- Make sure to login to the same node
- One of the admin suggested `unset TMPDIR` before running `tmux` (I'm still having weird issues)

## submitting jobs

If you do want to use SLURM directly the documentation for submitting simple bash scripts is pretty good and can be found [here](http://sherlock.stanford.edu/mediawiki/index.php/SLURMSubmit).

Some additional points:
- you can specify which queue you want to run jobs on using the -p option to sbatch. The default is `normal`, there will be some name for our queue ('pritch'?), 'gpu' is probably self explanatory, and 'owners' allows you to submit jobs on other labs cluster allocation, with the caveat that your job will be cancelled (and later rescheduled) if that lab needs the compute. You can specify multiple queues! In which case SLURM will use the first available.
- Two useful ways of running sets of jobs are array jobs and using the --export option. Using e.g. --array=1-10 as an option to sbatch will submit 10 jobs with an environment variable SLURM_ARRAY_TASK_ID set to corresponding job index. Alternatively you can call sbatch multiple times using e.g. --export=MYINDEX=1 etc, which will create an environment variable MYINDEX set to 1 for that job.
- You can also modify the amount of memory with the argument `-m`.

## Interactive jobs

Running `sdev` gets you an interactive job. Annoyingly the default run time for these is only an hour, and the max is 2h [use -t=2:00:00 to get this]. These instances also don't get much RAM (and no GPU).

You can also run interactive jobs on other queues (see `sdev -h`)

## Storage

The storage options are very well documented here:
http://sherlock.stanford.edu/mediawiki/index.php/DataStorage

## Using the GPU

Sherlock is very well set up for using their/our GPU machines.

To have stuff setup for theano I have the following in my `.bash_profile`:

```
ml load cuDNN/v4
ml load theano
ml load openblas/0.2.15
alias sgpu="srun -p gpu,owners --qos gpu --gres gpu:1 --pty bash"
```

The last line creates an alias to request an iteractive GPU job. Note these don't tend to get scheduled until you leave the lab :/

# Sherlock 2

Login nodes on sherlock 2:

- sh-ln01.stanford.edu
- sh-ln02.stanford.edu

## tmux

tmux seems better behaved on sherlock for whatever reason.
It works by doing the following:

- add the following line to your `.bashrc`: `unset TMPDIR`.
- Logging into the same node consistently.

Here's an example of my `~/.ssh/config` that allows me to simply type `ssh sherlock2`:

```
Host sherlock2
GSSAPIDelegateCredentials yes
GSSAPIAuthentication yes
HostName sh-ln01.stanford.edu
User hjp
Port 22
```
