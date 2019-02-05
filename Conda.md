Dependency management with Anaconda
========================================================
author: Kevin Thornton
date:  Week 5
autosize: true

The problem
========================================================

* Keeping version numbers constant for project A...
* ...while project B to use different versions of stuff...
* ...while letting a third project, C, use the latest version of anything.

In general, this is a **very difficult** problem.

The solution(s)
======================================================

Broadly-speaking, there are two classes of solutions:

1. Package managers
2. Containers

Package managers
===========================================

1. Source code
2. apt/yum/etc. on Linux
3. [brew](http://brew.sh)
4. [conda](http://conda.io), usually via [anaconda](https://www.anaconda.com/)

[conda](http://conda.io) is the only one that satisfies our requirements by allowing "environments" or "envs".

(Also, [homebrew-science](https://github.com/Homebrew/homebrew-science) is deprecated due to poor quality control.)

Conda
============================

* Conda is a *package manager*
* Aanconda is a distribution of conda.
* Conda **cannot** be used to replicate environments between OS X and Linux.  That is not the intended use.

Installation
==================================

* Get a distribution from Anaconda [here](https://repo.continuum.io/)
* The "miniconda3" version is fine. **Get the 64-bit version**
* (Avoid "miniconda2"--that is Python 2.)
* Follow the instructions

Installing on HPC
====================================

* DO NOT install into your user's home directory.
* Doing so can easily fill your space quota.
* Also, your $HOME is not backed up and you aren't supposed to keep **anything** important there.
* Install it as a *module* instead:


```sh
# This is the installation for my lab
module load krthornt/anaconda
```

Module files
==================================

* The go in ```/data/modulefiles/user_contributed_software/$USER```.
* They have the following file name format:


```eval
/data/modulefiles/user_contributed_software/$USER/program/version
```

* For example:

```sh
#"3" is a file name
/data/modulefiles/user_contributed_software/krthornt/anaconda/3
```

Example
====================================================

```sh
#%Module1.0

module-whatis "Thornton lab's anaconda3 installation"

exec /bin/logger -p local6.notice -t module-hpc $env(USER) "krthornt/anaconda/3"

set ROOT /data/apps/user_contributed_software/krthornt/anaconda3

prepend-path    PATH               $ROOT/bin

## for dev/lib installs
prepend-path -d " "   LDFLAGS           "-L${ROOT}/lib"
prepend-path -d " "   CPPFLAGS          "-I$ROOT/include"
prepend-path          INCLUDE           "$ROOT/include"
```

Where the software goes
===================================================================

* `/data/apps/user_contributed_software/$USER/program`

This location is pointed to via the `set ROOT` command in the module file.

Installing conda to a custom location (like on HPC)
============================================================

When installing `conda`, you'll get to a prompt like this:

```
Miniconda2 will now be installed into this location:
/home/molpopgen/miniconda2

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below
```

Simply enter the place to install:


```sh
/data/apps/user_contributed_software/$USER/anaconda3
```

Conda channels
================================================

* Different "channels" provide different software
* You'll probably want to know about [BioConda](https://bioconda.github.io/)

Use:


```sh
# Order matters!
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda install bwa
```

To list your channels:

```
conda config --get channels
```

Cite your software
==================================================

If you use BioConda, cite it!  It is maintained by volunteers (like me), who get *no credit* for doing this unless you cite the paper:

```
Grüning, Björn, Ryan Dale, Andreas Sjödin, Brad A. Chapman, Jillian Rowe, Christopher H. Tomkins-Tinch, Renan Valieris, the Bioconda Team, and Johannes Köster. 2018. “Bioconda: Sustainable and Comprehensive Software Distribution for the Life Sciences”. Nature Methods, 2018 doi:10.1038/s41592-018-0046-7.
```

Same goes for R, tidyverse, pandas, numpy, etc.--cite what you use!

Environments
==================================================

* This is where the magic happens!
* Environment have names, and they can contain different versions of software.
* Let's look at the [docs](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#creating-an-environment-with-commands) for "envs".
* They are like Python "virtualenvs", but they handle C/C++/Java/whatnot packages, too, *as well as* stuff installed via `pip`.

Environments: getting started
=============================================
```
conda create --name env_name
source activate env_name
# do work
source deactivate
```

**NOTE** channels are persistent across environments!

Environment: tips and tricks
=======================================================
Base it on specific packages/versions:
```
conda create -n py36_rstudio python==3.6 rstudio
```
Clone an environment:
```
conda create --name myclone --clone myenv
```
List your envs:
```
conda info --envs
```

Environments: dumping and restoring
=============================================================
Dump to a `YAML` file:
```
conda env export -n bioconda > bioconda.yml
```
Create from a `YAML` file:
```
conda env create -n env_name -f file.yml
```

Use "explicit method":
``` 
conda list --explicit -n bioconda > bioconda_explicit.txt
```
Create from that file:
```
conda create -n env_name --file bioconda_explict.txt
```

Containers (a.k.a the future)
==============================================

* We are talking about Linux only here!
* A very different method for development/distribution/packaging of software.
* Start with a light OS image.  For example, Ubuntu Linux 18.10.
* Install your software + all its dependencies onto that image.
* Provide a client/server/cloud model for making your images available
* More info [here](https://opensource.com/resources/what-are-linux-containers)
* [Docker](http://www.docker.com) is the lower-level infrastructure
* [Singularity](https://singularity.lbl.gov/) is a more convenient approach, intended for use on HPC.
* All bioconda packages are turned into Docker containers. (I have not played with this yet.)


