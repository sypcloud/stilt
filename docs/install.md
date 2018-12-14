---
layout: docs
title: STILT Installation
---

# Fair Use Policy

STILT is freely available and we encourage others to use it. Kindly keep us informed of how you are using the model and of any publication plans. Please acknowledge the source as a citation. STILT is continuously updated and improved by the development consortium, and in some cases \(as when new elements are used for the first time\) we may suggest that one or more of the developers be offered participation as authors. If your work directly competes with our analysis, we may ask that we have the opportunity to submit a manuscript before you submit one that uses unpublished features. The software is updated from time to time, and it is your responsibility to ensure that your publication is consistent with the most recent version.

# Installation

STILT has been compiled to run on UNIX platforms \(Mac, Linux\). Required software includes

* R \(v &gt;= 3.25\), [https://www.r-project.org/](https://www.r-project.org/)
  * `dplyr` package, for speed and data manipulation
  * `parallel` package, for parallel computation on a single node
  * `raster` package, for raster-based spatial analysis
  * `rslurm` package, for parallel computation across multiple nodes
  * `uataq` package, for data manipulation
* One of the following Fortran compilers (listed by preference/simulation speed)
  * GNU Fortran Compiler \(gfortran\)
  * Portland Group Compiler \(pgf90\)
  * Intel Fortran Compiler \(ifort\)
  * G95 Fortran Compiler \(g95\)
* Git, [https://git-scm.com/](https://git-scm.com/)

## Install methods

Three options exist to initialize a new STILT project.

### R \(preferred\)

This method uses R to initialize a new project. `stilt_init()` in the Utah Atmospheric Trace gas & Air Quality \(UATAQ\) R package, which includes tools for trace gas data manipulation and analysis, is a wrapper around several system commands that do much of the heavy lifting. The `uataq` R package is available on [Github](https://github.com/benfasoli/uataq/) and can be installed in R using `devtools`.

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">R</div>
  <div class="terminal-command">
    if (!require('devtools')) install.packages('devtools')
  </div>
  <div class="terminal-command">
    devtools::install_github('benfasoli/uataq')
  </div>
</div>


A STILT project is initialized in plain R code with

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">R</div>
  <div class="terminal-command">
    uataq::stilt_init('myproject')
  </div>
  <div class="terminal-return">
    Cloning into 'stilt'...<br>
    remote: Counting objects: 1712, done.<br>
    remote: Compressing objects: 100% (275/275), done.<br>
    remote: Total 1712 (delta 71), reused 267 (delta 39), pack-reused 1384<br>
    Receiving objects: 100% (1712/1712), 32.85 MiB | 8.57 MiB/s, done.<br>
    Resolving deltas: 100% (652/652), done.<br>
    Checking connectivity... done.<br>
    <br>
    STILT hymodelc installation options:<br>
    1 - Compile hymodelc from source<br>
    2 - hymodelc-centos-7.4<br>
    3 - hymodelc-ubuntu-16.04<br>
    4 - hymodelc-ubuntu-14.04<br>
    5 - hymodelc-macos-10.13<br>
    <br>
    Install option (number from above):
  </div>
</div>

[Binaries are available](https://github.com/uataq/stilt/releases) for several systems or you can choose to compile the `hymodelc` binary from the source. Compiling from source code requires [user registration](https://mail.bgc-jena.mpg.de/mailman/listinfo/stilt_user) to receive login credentials to the SVN repository.

> Since the name of the Github Repository is "stilt", a name other than "stilt" should be chosen for projects. Otherwise, conflicts will arise when attempting to clone the master repository for initializing future projects in the same parent directory.

### Manual

While the R method is preferable since it streamlines the process of initializing new projects, the same can be accomplished manually. To reproduce the results above,

```bash
# Clone GitHub repo for R wrapper
git clone https://github.com/benfasoli/stilt
# Set the name of the project
mv stilt myproject

# Checkout merged_stilt_hysplit from SVN
svn --username {USERNAME} checkout \
    https://projects.bgc-jena.mpg.de/STILT/svn/trunk/merged_stilt_hysplit/ \
    fortran/
# Compile hymodelc using the provided makefile
(cd fortran && make)
# Move the hymodelc binary to exe/
mv fortran/hymodelc exe/hymodelc
# Ensure hymodelc is executable
chmod +x exe/hymodelc

# Compile permute DLL for footprint kernel aggregation
R CMD SHLIB r/src/permute.f90
```

Finally, edit settings in `r/run_stilt.r`, being sure to specify the project name and the working directory.


### Docker \(advanced\)

Building a STILT docker image enables STILT to be run dependency free. This method does not use `run_stilt.r` and `stilt_apply()` related parallelization. Instead, runs are intended to be single-shot by supplying `run_stilt.r` parameters as command arguments when running a container. Parallelization should be accomplished using container oriented tools ([Kubernetes](https://kubernetes.io/), [Docker swarm](https://docs.docker.com/engine/swarm/)).

```bash
# Clone GitHub repo for R wrapper
git clone https://github.com/benfasoli/stilt --depth=1 --single-branch
# Build docker image
cd stilt
docker build -t stilt .

# Fetch example met data for testing
git clone https://github.com/uataq/stilt-tutorials /tmp/stilt-tutorials

# Create host input/output paths
METDIR=/tmp/stilt-tutorials/01-wbb/met
OUTDIR=/tmp/stilt-out
mkdir -p $METDIR $OUTDIR
docker run \
  --rm \
  --mount type=bind,source=$METDIR,destination=/app/met,readonly \
  --mount type=bind,source=$OUTDIR,destination=/app/out/by-id \
  stilt \
  r_run_time=2015-12-10T00:00:00Z \
  r_lati=40.5 \
  r_long=-112.0 \
  r_zagl=5 \
  met_loc=/app/met \
  met_file_format=%Y%m%d.%H \
  xmn=-112.3 \
  xmx=-111.52 \
  xres=0.01 \
  ymn=40.39 \
  ymx=40.95 \
  yres=0.01
```