---
layout: tutorial
title: STILT WBB Tutorial
---

# WBB Carbon Dioxide

Here, we'll simulate a day of carbon dioxide concentrations for the [UATAQ Lab](https://air.utah.edu), housed in the William Browning Building (WBB) on the University of Utah campus. This tutorial assumes a base level of knowledge for [navigating UNIX based filesystems from the command line](https://www.digitalocean.com/community/tutorials/basic-linux-navigation-and-file-management), that you have read through [installing STILT]({{"/docs/install.html"|relative_url}}), and know where to find documentation for the different [model controls]({{"/docs/controls.html"|relative_url}}).

### Dependencies

Let's start by checking to be sure the necessary dependencies are installed.

> The University of Utah CHPC clusters use [modules](https://www.chpc.utah.edu/documentation/software/modules.php) to manage environment paths. To load dependencies within one of the CHPC clusters, you need to load the `netcdf-c` and `R` modules. You can run `module load netcdf-c R` in your current bash session or add it to the bottom of your shell configuration file (`.bashrc`) to load the dependencies at every login.

We need R version 3.2.5 or higher, which you can find in the output from

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    Rscript --version
  </div>
  <div class="terminal-return">
    R scripting front-end version 3.4.1 (2017-06-30)
  </div>
</div>

> If R is not found or the version is too old, you'll need to update R on your system before continuing.

STILT requires several R packages as helpers for file input/output, data manipulation, and parallel execution. We can install the dependencies in R with

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">R</div>
  <div class="terminal-command">
    install.packages(c('devtools', 'dplyr', 'ncdf4', 'parallel', 'raster', 'rslurm'))
  </div>
  <div class="terminal-return">
    ...
  </div>
</div>

To compile the `hymodelc` executable, the makefile will default to searching for gfortran. We can check if gfortran is installed with

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    which gfortran
  </div>
  <div class="terminal-return">
    /usr/bin/gfortran
  </div>
</div>

> If `which` returns a path, then `gfortran` is installed. If `which` returns something along the lines of `/usr/bin/which: no gfortran in...`, you need to make some changes for the compilation to be successful. To use gfortran, you'll need to add the executable to your shell's PATH. To use a different compiler, you'll need to manually clone the [STILT Github Repository](https://github.com/uataq/stilt), specify the compiler in `fortran/Makefile` using the `FC` variable, then run the `./setup` executable. See the [documentation for installation]({{"/docs/install.html"|relative_url}}) for details.

Last, we need to check if we have netCDF installed. Footprints are saved in compressed netCDF files, which reduces their file size and stores results with associated metadata so that the output is self documenting. We can check if netCDF is installed with

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    nc-config --all
  </div>
  <div class="terminal-return">
    This netCDF 4.4.1 has been built with the following features:<br>
    ...
  </div>
</div>

> If `nc-config --all` returns build information about the netCDF configuration, then `netCDF` is installed. Otherwise, you will need to install netCDF on your system before continuing or (advanced) reconfigure the footprint file output extension in `simulation_step()` and `calc_footprint()` to return footprints using a different file format.


### Project setup

Now that we have the dependencies we need, let's start a new STILT project using the [uataq R package](https://github.com/benfasoli/uataq). We can install this package from Github within R using the `devtools` package as

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">R</div>
  <div class="terminal-command">
    devtools::install_github('benfasoli/uataq')
  </div>
</div>

Then we can initialize our STILT project in our current directory within R `uataq::stilt_init()`. This function will prompt you through compiling the required binaries.

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">R</div>
  <div class="terminal-command">
    uataq::stilt_init('wbb_tutorial')
  </div>
</div>

To ensure everything compiled correctly, check to be sure you can find `hymodelc` in `exe/`

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    cd wbb_tutorial
  </div>
  <div class="terminal-command">
    ls exe
  </div>
  <div class="terminal-return">
    ASCDATA.CFG hymodelc LANDUSE.ASC ROUGLEN.ASC
  </div>
</div>

Success! We've now set up our STILT project.


### Input data

The minimum we need to simulate the carbon dioxide concentration at WBB is (1) meteorological data to transport the STILT particles and (2) a near-field emissions inventory. You can download example data for this tutorial in the base directory of your STILT project using

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    git clone https://github.com/uataq/stilt-tutorials
  </div>
  <div class="terminal-command">
    ls stilt-tutorials/01-wbb
  </div>
  <div class="terminal-return">
    emissions.rds met/ tutorial.r
  </div>
</div>

which contains

1. `emissions.rds` - 0.002 deg hourly emissions inventory
1. `met/` - meteorological data files
1. `tutorial.r` - a simple script to combine footprints with the emissions inventory and plot a timeseries of the concentrations


### Configuration

Now, we need to configure STILT for our example. Begin by opening `r/run_stilt.r` in a text editor. Unless otherwise described below, leave the simulation settings as their defaults.

Set the simulation timing and receptor location to

```r
# Simulation timing, yyyy-mm-dd HH:MM:SS
t_start <- '2015-12-10 00:00:00'
t_end <- '2015-12-10 23:00:00'
run_times <- seq(from = as.POSIXct(t_start, tz='UTC'),
                 to = as.POSIXct(t_end, tz='UTC'),
                 by = 'hour')

# Receptor locations
lati <- 40.766189
long <- -111.847672
zagl <- 25
```

Next, we need to tell STILT where to find the meteorological data files for the sample. Set the `met_directory` to

```r
# Meteorological data input
met_directory <- file.path(stilt_wd, 'stilt-tutorials', '01-wbb', 'met')
met_file_format <- '%Y%m%d.%Hz.hrrra'
```

Last, let's adjust the footprint grid settings so that it uses the same domain as our emissions inventory. Set the footprint grid settings to

```r
# Footprint grid settings
xmn <- -112.30
xmx <- -111.52
ymn <- 40.390
ymx <- 40.95
xres <- 0.002
yres <- xres
```

That's it! We're all set to run the model. From the base directory of our STILT project, run `Rscript r/run_stilt.r` and wait a few minutes for the simulations to complete.

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    Rscript r/run_stilt.r
  </div>
  <div class="terminal-return">
    Parallelization disabled...<br>
    Running simulation ID: 2015121000_-111.847672_40.766189_25<br>
    Running simulation ID: 2015121001_-111.847672_40.766189_25<br>
    ...
  </div>
</div>


### Applying emissions

Now that we have 24 footprints for each hour of our simulation, the next step is to convolve the footprints with our emissions inventory. An example of how to do this can be found in `stilt-tutorials/01-wbb/tutorial.r`, which makes some overly-basic assumptions to calculate the carbon dioxide concentration at the receptor.

To convolve the footprints with emissions estimates,

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-lang">bash</div>
  <div class="terminal-command">
    cd stilt-tutorials/01-wbb
  </div>
  <div class="terminal-command">
    Rscript tutorial.r
  </div>
</div>

which will output `timeseries.png` to the current directory showing the modeled concentrations

<div class="text-center">
  <img style="max-width:400px; margin: auto;" src="{{"/img/wbb-tutorial-timeseries.png"|relative_url}}">
</div>

as well as maps for the average footprint (`average_footprint.png`) and average contribution of fluxes over space (`average_contribution.png`).
<div class="row">
  <div class="col-sm-6">
    <img src="{{"/img/wbb-tutorial-average-footprint.jpg"|relative_url}}">
  </div>
  <div class="col-sm-6">
    <img src="{{"/img/wbb-tutorial-average-contribution.jpg"|relative_url}}">
  </div>
</div>

> Note that the strange boundary of the contributed emissions is the extent of Salt Lake County, which is the area for which the emissions data exists and is non-NaN.
