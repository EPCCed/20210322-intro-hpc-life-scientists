---
title: "PRACTICAL: Benchmarking Molecular Dynamics Performance Using GROMACS 1"
teaching: 10
exercises: 20
questions:
- "How does a small, 80k-atom system performance scale as more cores are used?"
- "What about a larger, 12M-atom system?"
objectives:
- "Gain a basic understanding of key aspects of running molecular dynamics
simulations in parallel."
keypoints:
- "Larger systems scale better to large core-/node-counts than smaller systems."
---

## Aims

In this exercise, you will be running molecular dynamics simulations using
[GROMACS](https://manual.gromacs.org/). You will be benchmarking how
efficiently a small system will run as you increase the number of processors.

## Instructions

You  will be running a benchmark that has been used to explore the
performance/price behaviour  of  GROMACS on various  generations of CPUs and
GPUs. A number of publications have used this benchmark to report on, and
compare, the performance of GROMACS on different systems (*e.g* see:
[https://doi.org/10.1002/jcc.24030](https://doi.org/10.1002/jcc.24030)
).

The  benchmark system in question is “benchMEM”, which is available from the
list of Standard MD benchmarks at https://www.mpibpc.mpg.de/grubmueller/bench.
This benchmark simulates a membrane channel protein embedded in a lipid
bilayer surrounded by water and ions. With its size of ~80 000 atoms, it
serves as a prototypical example for a large class of setups used to study all
kinds of membrane-embedded proteins.For some more information see
[here](https://www.mpibpc.mpg.de/16460085/bench.pdf).

To get a copy of "benchMEM", run the following from your work directory:

```
wget https://www.mpinat.mpg.de/benchMEM
unzip benchMEM.zip
```
{: .language-bash}

Once the file is unzipped, you will need to create a Slurm submission script.
Below is a copy of a script that will run on a single node, using a single
processor. You can either copy the script from here, or from ARCHER2 in
directory `/work/ta017/shared/GMX_sub.slurm`.

```
#!/bin/bash

#SBATCH --job-name=GMX_test
#SBATCH --account=ta017
#SBATCH --partition=standard
#SBATCH --qos=short
#SBATCH --reservation=shortqos
#SBATCH --time=0:5:0

#SBATCH --nodes=1
#SBATCH --tasks-per-node=1
#SBATCH --cpus-per-task=1

#SBATCH --distribution=block:block
#SBATCH --hint=nomultithread

module restore /etc/cray-pe.d/PrgEnv-gnu
module load gromacs

export OMP_NUM_THREADS=$SLURM_CPUS_PER_TASK

srun gmx_mpi mdrun -ntomp $SLURM_CPUS_PER_TASK -s benchMEM.tpr
```
{: .language-bash}

Run this script to make sure that it works -- how quickly does the job
complete? You can see the walltime and performance of your code by running
`tail md.log`. This simulation is meant to perform 10,000 steps, each of which
should simulate 2 ps of "real-life" time. The "Walltime" tells you how quickly
the job ran on ARCHER2, and the "Performance data will let you know how many
nanoseconds you can run in a day, and how many hours it will take to run a
nanosecond of simulation.

How do the "Walltime" and "Performance" data change as you increase the number
of cores being used? You can vary this by changing
`#SBATCH --tasks-per-node=1` to a higher number. Try filling out the table
 below:

 |Number of cores| Walltime | Performance (ns/day) | Performance (hours/ns) |
 |---------------|----------|----------------------|------------------------|
 |   1  | | | |
 |   2  | | | |
 |   4  | | | |
 |   8  | | | |
 |  16  | | | |
 |  32  | | | |
 |  64  | | | |
 | 128  | | | |
 | 256* | | | |
 | 512* | | | |

 ---
 **NOTE**

 Jobs run on more than one node will need to be run with constant
 `#SBATCH --tasks-per-node=128` but varying `#SBATCH --nodes=1`

 ---

If you finish early, you can try generating this same table for the GROMACS
benchPEP benchmark. You can get the benchmark by running:

```
wget https://www.mpibpc.mpg.de/15615646/benchPEP.zip
unzip benchPEP.zip
```

This is a much larger system (12M atoms compared with 80k atoms for the
benchPEP system). How does the performance scaling of this system compare with
that of the smaller system?

{% include links.md %}
