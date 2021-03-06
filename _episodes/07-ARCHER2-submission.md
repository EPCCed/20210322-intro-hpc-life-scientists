---
title: "PRACTICAL: Batch Systems and ARCHER2 Slurm Scheduler"
teaching: 15
exercises: 15
questions:
- "How do I write job submission scripts?"
- "How do I control jobs?"
- "How do I find out what resources are available?"
objectives:
- "Understand the use of the basic Slurm commands."
- "Know what components make up and ARCHER2 scheduler."
- "Know where to look for further help on the scheduler."
keypoints:
- "ARCHER2 uses the Slurm scheduler."
- "`srun` is used to launch parallel executables in batch job submission scripts."
- "There are a number of different partitions (queues) available."
---

ARCHER2 uses the Slurm job submission system, or *scheduler*, to manage resources and how they are made
available to users. The main commands you will use with Slurm on ARCHER2 are:

* `sinfo`: Query the current state of nodes
* `sbatch`: Submit non-interactive (batch) jobs to the scheduler
* `squeue`: List jobs in the queue
* `scancel`: Cancel a job
* `salloc`: Submit interactive jobs to the scheduler
* `srun`: Used within a batch job script or interactive job session to start a parallel program

Full documentation on Slurm on ARCHER2 can be found in [the *Running Jobs on ARCHER2* section of the User
and Best Practice Guide](https://docs.archer2.ac.uk/user-guide/scheduler.html).

## Finding out what resources are available: `sinfo`

The `sinfo` command shows the current state of the compute nodes known to the scheduler:

```
auser@login01-nmn:~> sinfo
```
{: .language-bash}
```
PARTITION AVAIL  TIMELIMIT  NODES  STATE NODELIST
standard     up 1-00:00:00     60  down* nid[001006,001033,001045-001047,001061,001068,001074,001109,001125,001138,001149,001163,001171,001227-001228,001241,001255,001262,001273,001287,001326,001336,001347,001366,001369,001395,001435,001462,001478,001490,001505,001539,001546,001552,001581,001614,001642,001644-001645,001647,001652,001664,001669,001709,001719,001723,001729,001747,001751,001757,001810,001817,001839,001903,001919,001932,001950,001955,002014]
standard     up 1-00:00:00     11  drain nid[001016,001069,001092,001468,001520-001521,001812,001833-001835,001838]
standard     up 1-00:00:00      5   resv nid[001001-001004,001021]
standard     up 1-00:00:00    565  alloc nid[001000,001005,001007-001015,001018-001020,001022-001032,001034-001044,001048-001060,001062-001067,001070-001073,001075-001091,001093-001108,001110-001124,001126-001137,001139-001148,001150-001155,001158-001162,001164-001170,001172-001226,001229-001240,001242-001254,001256-001261,001263-001272,001274-001286,001288-001317,001319-001325,001327-001335,001337-001346,001348-001365,001367-001368,001370-001394,001396-001434,001436-001461,001463-001467,001469-001477,001491-001504,001547-001551,001553-001580,001582-001613,001615-001641,001648-001651,001653-001663,001665-001668,001951-001954]
standard     up 1-00:00:00    380   idle nid[001017,001156-001157,001479-001489,001506-001519,001522-001538,001540-001545,001643,001646,001670-001688,001690-001708,001710-001718,001720-001722,001724-001728,001730-001746,001748-001750,001752-001756,001758-001809,001811,001813-001816,001818-001824,001826-001832,001836-001837,001840-001902,001904-001918,001920-001931,001933-001949,001956-002013,002015-002023]
```
{: .output}

There is a row for each node state and partition combination. The default output shows the following columns:

* `PARTITION` - The system partition
* `AVAIL` - The status of the partition - `up` in normal operation
* `TIMELIMIT` - Maximum runtime as `days-hours:minutes:seconds`: on ARCHER2, these are set using *QoS*
  (Quality of Service) rather than on partitions
* `NODES` - The number of nodes in the partition/state combination
* `STATE` - The state of the listed nodes (more information below)
* `NODELIST` - A list of the nodes in the partition/state combination

The nodes can be in many different states, the most common you will see are:

* `idle` - Nodes that are not currently allocated to jobs
* `alloc` - Nodes currently allocated to jobs
* `draining` - Nodes draining and will not run further jobs until released by the systems team
* `down` - Node unavailable
* `fail` - Node is in fail state and not available for jobs
* `reserved` - Node is in an advanced reservation and is not generally available
* `maint` - Node is in a maintenance reservation and is not generally available

If you prefer to see the state of individual nodes, you can use the `sinfo -N -l` command.

> ## Lots to look at!
> Warning! The `sinfo -N -l` command will produce a lot of output as there are over 1000 individual
> nodes on the current ARCHER2 system!
{: .callout}

```
auser@login01-nmn:~> sinfo -N -l
```
{: .language-bash}
```
Fri Jul 10 09:45:54 2020
NODELIST   NODES PARTITION       STATE CPUS    S:C:T MEMORY TMP_DISK WEIGHT AVAIL_FE REASON
nid001001      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001002      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001003      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001004      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001005      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001006      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001007      1    standard        idle  256   2:64:2 244046        0      1   (null) none
nid001008      1    standard        idle  256   2:64:2 244046        0      1   (null) none

...lots of output trimmed...

```
{: .output}

> ## Explore a compute node
> Let us look at the resources available on the compute nodes where your jobs 
> will actually run. Try running this command to see the name, CPUs and memory 
> available on the worker nodes (the instructors will give you the ID of the 
> compute node to use):
> ```
> [auser@login01-nmn:~> sinfo -n nid001005 -o "%n %c %m"
> ```
> {: .language-bash}
> This should display the resources available for a standard node.
>
> It is also possible to search nodes by state. Can you find all the free nodes in the system?
> > ## Solution
> > `sinfo` lets you specify the state of a node to search for, so to get all the free nodes in the system you can use:
> > ```
> > sinfo -N -l --state=idle
> > ```
> > More information on what `sinfo` can display can be found in the `sinfo` manual page, i.e. `man sinfo`
>> 
> {: .solution}
{: .challenge}

## Using batch job submission scripts

### Header section: `#SBATCH`

As with most other scheduler systems, job submission scripts in Slurm consist of a header section with the
shell specification and options to the submission command (`sbatch` in this case) followed by the body of
the script that actually runs the commands you want. In the header section, options to `sbatch` should
be prepended with `#SBATCH`.

Here is a simple example script that runs the `xthi` program, which shows
process and thread placement, Here we consider only MPI and assume there
in no OpenMP involved.

The intention is to run using two nodes (nodes will always be allocated on
and exclusive basis) and use 128 MPI tasks per node (i.e., one per
physical core).


```
#!/bin/bash

#SBATCH --partition=standard
#SBATCH --qos=standard
#SBATCH --time=00:02:00

#SBATCH --nodes=2
#SBATCH --ntasks-per-node=128

#SBATCH --hint=nomultithread
#SBATCH --distribution=block:block

# Load the default programming environment, and the xthi utility
module load epcc-job-env
module load xthi/1.0

# srun to launch the executable
srun xthi
```
{: .language-bash}

The options shown here are:

* `--partition=standard` - Submit to the standard set of nodes.
* `--qos=standard` - Submit with the standard quality of service settings.
* `--time=00:02:00` - Set wall clock time limit (here 2 minutes).
* `--hint=nomultithread` - Base placement on 128 physical cores per node.
* `--distribution=block:cyclic` - Set inter-node:intra-node MPI distribition.
* `--nodes=2` - Request two nodes for this job
* `--ntasks-per-node=128` - Use 128 parallel processes per node (MPI ranks)

Other options include (there are many)
* `--job-name=my_mpi_job` - Set the name for the job that will be displayed in Slurm output
* `--account=t01` - Charge the job to the `t01` budget

We will discuss the `srun` command further below.

### Submitting jobs using `sbatch`

You use the `sbatch` command to submit job submission scripts to the scheduler.
For example, if the above script was saved in a file called `test_job.slurm`,
you would submit it with:

```
auser@login01-nmn:~> sbatch test_job.slurm
```
{: .language-bash}
```
Submitted batch job 23996
```
{: .output}

Slurm reports back with the job ID for the job you have submitted

> ## What are the default for `sbatch` options?
> If you do not specify job options, what are the defaults for Slurm on ARCHER2? Submit jobs to find out
> what the defaults are for:
>
> 1. The number of nodes used by the job?
> 2. The number of tasks per node?
> 3. The wall time limit? (Hint: you may need the command: `sacct` or `sinfo`)
> 4. What other options can be ommited without error?
>
> > ## Solution
> >
> > (1) If `--nodes` is omitted, the default is 1 node.
> >
> > (2) If `--ntasks-per-node` is omitted, the default is 1 task per node.
> >
> > (3) Check `man sacct` and look for the time limit field.
> >
> > If we had a job with jobid 12345, then we could query the time limit
> > for that particular job with, e.g.,
> >
> > ```
> > auser@login01-nmn:~> sacct -o "TimeLimit" -j 12345
> > ```
> > {: .language-bash}
> > ```
> >  Timelimit
> > ----------
> >   01:00:00
> > ```
> > {: .output}
> >
> > (4) A `--partition` must be specified, and a `--qos` must be specifed. An
> >     error will be generated at the point of submission if either is
> >     omitted.
> {: .solution}
{: .challenge}

### Checking progress of your job with `squeue`

You use the `squeue` command to show the current state of the queues on ARCHER2. Without any options, it
will show all jobs in the queue:

```
auser@login01-nmn:~> squeue
```
{: .language-bash}
```
             JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
```
{: .output}

### Cancelling jobs with `scancel`

You can use the `scancel` command to cancel jobs that are queued or running. When used on running jobs
it stops them immediately.

### Running parallel applications using `srun`

Once past the header section your script consists of standard shell commands required to run your
job. These can be simple or complex depending on how you run your jobs but even the simplest job
script usually contains commands to:

* Load the required software modules
* Set appropriate environment variables (you should always set `OMP_NUM_THREADS`, even if you are
  not using OpenMP you should set this to `1`)

After this you will usually launch your parallel program using the
`srun` command. At its simplest, `srun` does not require any arguments
(it will use values supplied to `sbatch` to work out how many parallel
processes to launch). In the example above, our `srun` command simply
looks like:

```
srun xthi
```
{: .language-bash}

> ## Fewer tasks than cores
> You may want to run fewer MPI tasks per node than there are cores per node
> on ARCHER2 to access more memory, or more memory bandwidth, per task.
> This requires the option ``--cpus-per-task`` to specify how many "cpus"
> (in this context, cores) are allocated to each MPI task.
> Can you determine the `sbatch` options you would use to run `xthi`:
>
> 1. On 4 nodes with 64 tasks per node?
> 2. On 2 nodes with 2 tasks per node, 1 task per socket?
> 3. On 4 nodes with 8 tasks per node, ensuring an even distribution across
>    the 8 NUMA regions on the node?
>
> Once you have your answers run them in job scripts and check that the
> placement on
> nodes and cores output by `xthi` is what you expect.
>
> > ## Solution
> > 1. `--nodes=4 --ntasks-per-node=64`
> > 2. `--nodes=2 --ntasks-per-node=2 --cpus-per-task=64`
> > 3. `--nodes=4 --ntasks-per-node=8`
> {: .solution}
{: .challenge}




## STDOUT/STDERR from jobs

STDOUT and STDERR from jobs are, by default, written to a file called `slurm-<jobid>.out` in the
working directory for the job (unless the job script changes this, this will be the directory
where you submitted the job). So for a job with ID `12345` STDOUT and STDERR would be in
`slurm-12345.out`.

If you run into issues with your jobs, the Service Desk will often ask you to send your job
submission script and the contents of this file to help debug the issue.

If you need to change the location of STDOUT and STDERR you can use the `--output=<filename>`
and the `--error=<filename>` options to `sbatch` to split the streams and output to the named
locations.

## Other useful information

In this section we briefly introduce other scheduler topics that may be useful to users. We
provide links to more information on these areas for people who may want to explore these
areas more.

### Interactive jobs: `salloc`

Similar to the batch jobs covered above, users can also run interactive jobs using the Slurm
command `salloc`. `salloc` takes the same arguments as `sbatch` but, obviously, these are
specified on the command line rather than in a job submission script.

Once the job requested with `salloc` starts, you will be returned to the command line
and can now start parallel jobs on the compute nodes interactively with the `srun` command
in the same way as you would within a job submission script.

For example, to execute `xthi` across all cores on two nodes (1 MPI task per core and no
OpenMP threading) within an interactive job you would issue the following commands:

```
auser@login01-nmn:~> salloc --nodes=2 --ntasks-per-node=128 --hint=nomultithread --time=00:10:00 --partition=standard --qos=standard
salloc: Granted job allocation 24236
auser@login01-nmn:~> module load xthi/1.0
auser@login01-nmn:~> srun xthi
```
{: .language-bash}
```
Node    0, hostname nid001107, mpi 128, omp   1, executable xthi
Node    1, hostname nid001108, mpi 128, omp   1, executable xthi
Node    0, rank    0, thread   0, (affinity =    0)
...
```
{: .output}

Once you have finished your interactive commands, you exit the interactive job with `exit`:

```
auser@login01-nmn:~> exit
exit
salloc: Relinquishing job allocation 24236
auser@login01-nmn:~>
```
{: .language-bash}

<!-- Need to add information on the solid state storage and Slurm once it is in place

### Using the ARCHER2 solid state storage

-->


{% include links.md %}


