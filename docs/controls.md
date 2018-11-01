---
layout: docs
title: STILT Controls
---

# Controls

The following parameters are found in `r/run_stilt.r` and are used to configure STILT. These settings are used to adjust model parameters, execute parallelized simulations, and calculate produce upstream influence footprints.

### System configuration

Arg       | Description
----------|----------------------------------------------------------------------
`project` | Project name
`stilt_wd`| Working directory for the given project
`lib.loc` | Path to R package installations, passed to `library()`

### Parallel simulation settings

Arg       | Description
----------|----------------------------------------------------------------------
`n_nodes` | If using SLURM for job submission, number of nodes to utilize
`n_cores` | Number of cores per node to parallelize simulations by receptor locations and times
`slurm`   | Logical indicating the use of rSLURM to submit job(s). When using SLURM, a `_rslurm` directory is created at the top level of the STILT project. This directory contains the SLURM submission scripts and node-specific log files.
`slurm_options` | Named list of options passed to `sbatch` using `rslurm::slurm_apply()`, which typically includes `time`, `account`, and `partition` values

### Simulation timing

Arg               | Description
------------------|--------------------------------------------------------------
`t_start/t_end`   | Receptor time(s) to initialize simulations, formatted as `'yyyy-mm-dd HH:MM:SS'` UTC
`run_times`       | Hourly simulations spanning `t_start` through `t_end` of length *n*

### Receptor locations

Arg               | Description
------------------|--------------------------------------------------------------
`lati`            | Receptor latitude(s), in degrees. Can be a single value or a vector of length *n*
`long`            | Receptor longitude(s), in degrees. Can be a single value or a vector of length *n*
`zagl`            | Receptor height(s), in meters above ground level. Can be a single value or a vector of length *n*

> Simulation timing and receptor locations are defined in this way for convenience and then expanded to contain the unique receptors in a *x*, *y*, *z*, *t* table. For more complex lists of receptors, this table can be created manually instead of using the simulation timing and receptor location definitions. To specify the receptors manually, a data frame named `receptors` can be created with column names `run_time` (POSIXct), `long` (double), `lati` (double), and `zagl` (double).

<div class="terminal">
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-osx-button"></div>
  <div class="terminal-command">
    str(receptors)
  </div>
  <div class="terminal-return">
    'data.frame':	100 obs. of  4 variables:<br>
      $ run_time: POSIXct, format: "2015-07-02 11:00:00" "2015-07-02 11:00:00" ...<br>
      $ long    : num  -112 -112 -112 -112 -112 ...<br>
      $ lati    : num  40.8 40.8 40.8 40.8 40.8 ...<br>
      $ zagl    : num  5 5 5 5 5 5 5 5 5 5 ...
  </div>
</div>

### Meteorological data input

Arg               | Description
------------------|--------------------------------------------------------------
`met_directory`   | Full directory path in which ARL compatible meteorological data files can be found
`met_file_format` | String detailing file naming convention for meteorological data files using a mixture of datetime and regex syntax. The formatting string accepts grep compatible regular expressions ('.\*.arl'), `strftime` compatible datetime strings ('%Y%m%d%H') or any combination of the two ('d.\*\_%Y%m%d_filename.arl'). Datetime syntax is expanded to all unique combinations required for the receptor and simulation duration and the intersection between the requested files and files available in `met_directory` is determined with grep, allowing partial matching and compatible regular expressions to be used to identify the relevant data. Matching does not require the full format to be specified - e.g. '\*.arl', '%Y', '%Y%m%d', '%Y%m%d_d0.*.arl' would all match with a file named '20180130_d01.arl' for a relevant receptor. For timestamped meteorological data files, supplying the structure of the timestamp is sufficient and recommended to avoid unexpected behavior with regular expressions. To require a minimum number of files to be present for a given simulation to proceed, see `n_met_min`.
`n_met_min`       | Minimum number of meteorological data files with which to proceed with simulation. Useful for capturing missing periods. For a -24 hour simulation using the 6 hour HRRR met data files, `n_met_min` should be set to 5, since `find_met_files()` ensures that data before and after the simulation is included. Defaults to 1.

> NOAA publishes High Resolution Rapid Refresh (HRRR) mesoscale model data in the ARL packed format required for STILT at [ftp://arlftp.arlhq.noaa.gov/pub/archives/hrrr/](ftp://arlftp.arlhq.noaa.gov/pub/archives/hrrr/). This is often the easiest place to start but is only available after June 15, 2015. The coupling of the popular Weather Research and Forecasting (WRF) model with STILT is well documented by [Nehrkorn, 2010](https://link.springer.com/article/10.1007%2Fs00703-010-0068-x).

### Model control

Arg             | Description
----------------|----------------------------------------------------------------
`n_hours` | Number of hours to run each simulation; negative indicates backward in time
`numpar` | number of particles to be run; defaults to 200
`rm_dat`  | Logical indicating whether to delete `PARTICLE.DAT` after each simulation. Default to TRUE to reduce disk space since all of the trajectory information is also stored in `STILT_OUTPUT.rds` alongside the calculated upstream influence footprint
`run_foot` | Logical indicating whether to produce footprints. If FALSE, `run_trajec` must be TRUE. This can be useful when calculating trajectories separate from footprints
`run_trajec` | Logical indicating whether to produce new trajectories with `hymodelc`. If FALSE, will try to load the previous trajectory outputs. This is often useful for regridding purposes
`timeout` | number of seconds to allow hymodelc to complete before sending SIGTERM and moving to the next simulation; defaults to 3600 (1 hour)
`varsiwant` | character vector of 4-letter hymodelc variables. Options include a vector containing a minimum of `'time', 'indx', 'lati', 'long', 'zagl', 'foot'` and optionally containing elements from the following options:

- `crai` convective rainfall rate [m/min]
- `dens` air density [kg/m3]
- `dmas` particle weight changes due to mass violation in wind fields [initial value = 1.0]
- `dswf` downward shortwave radiation [W/m2]
- `foot` footprint, or sensitivity of mixing ratio to surface fluxes [ppm/(μmole/m2/s)]
- `icdx` cloud index when using RAMS (Grell scheme) [1=updraft,2=environment,3=downdraft]
- `indx` unique particle identifier
- `lati` latitude position of particle [degrees]
- `lcld` low cloud cover [%]
- `lhtf` latent heat flux [W/m2]
- `long` longitude position of particle [degrees]
- `mlht` mixed-layer height [m]
- `rain` total rainfall rate [m/min]
- `rhfr` relative humidity fraction [0~1.0]
- `samt` amount of time particle spends below VEGHT (see section on SETUP.CFG) [min]
- `shtf` sensible heat flux [W/m2]
- `sigw` standard deviation of vertical velocity; measure of strength of vertical turbulence [m/s]
- `solw` soil moisture
- `sphu` specific humidity [g/g]
- `tcld` total cloud cover [%]
- `temp` air temperature at lowest model layer [K]
- `time` time since start of simulation; negative if going backward in time [min] indx particle index
- `tlgr` Lagrangian decorrelation timescale [s]
- `wout` vertical mean wind [m/s]
- `zagl` vertical position of particle [m above ground level]
- `zloc` limit of convection heights [m]
- `zsfc` terrain height [m above sea level]

### Footprint gridding

Arg             | Description
----------------|----------------------------------------------------------------
`xmn` | grid start longitude, in degrees from -180 to 180
`xmx` | grid end longitude, in degrees from -180 to 180
`ymn` | grid start latitude, in degrees from -180 to 180
`ymx` | grid end latitude, in degrees from -180 to 180
`xres` | resolution for longitude grid, in projection units (degrees for lat/lon, meters for other)
`yres` | resolution for latitude grid, in projection units (degrees for lat/lon, meters for other)
`hnf_plume` | logical indicating whether to apply a vertical gaussian plume model to rescale the effective dilution depth for particles in the hyper near-field. This acts to scale up the influence of hyper-local fluxes on the receptor. If enabled, requires `varsiwant` to include a minimum of `dens, tlgr, sigw, foot, mlht, samt`. Defaults to TRUE
`projection` | [proj4](https://proj4.org/usage/quickstart.html) string defining the map projection of the footprint netCDF output
`smooth_factor` | factor by which to linearly scale footprint smoothing; 0 to disable all smoothing, defaults to 1
`time_integrate` | logical indicating whether to integrate footprint over time or retain discrete hourly time steps in footprint output

### Transport and dispersion

Arg             | Description
----------------|----------------------------------------------------------------
`conage` | particle/puff conversions at conage (hours); defaults to 48
`cpack` | binary concentration packing. 0 - all grid points written to file. 1 - only nonzero points written to file. 2 - special non-regular grid. Defaults to 1
`delt` | integration timestep [min]; if set to 0.0, then timestep is dynamically determined
`dxf` | horizontal x grid adjustment factor for ensemble; defaults to 1
`dyf` | horizontal y grid adjustment factor for ensemble; defaults to 1
`dzf` | vertical factor for ensemble; defaults to 0.01 ~ 250m
`emisshrs` | duration of time to release particles over at the start of simulation, in hours; defaults to 0.01
`frhmax` | maximum value for horizontal rounding parameter; defaults to 3
`frhs` | horizontal puff rounding fraction for merge; defaults to 1
`frhs` | mass rounding fraction for enhanced merging; defaults to 0.1
`frmr` | the fraction of the mass that is permitted to be removed at krnd intervals. For certain simulations, such as when a pollutant has a high ambient background relative, a small removal rate will significantly reduce the number of puffs on the grid at no loss in accuracy; defaults to 0
`frts` | temporal puff rounding fraction for merge; defaults to 0.1
`frvs` | vertical puff rounding fraction for merge; defaults to 0.1
`hscale` | horizontal Lagrangian time scale (sec); defaults to 10800
`ichem` | special chemistry or conversion modules. 1 - concentration grid treated as source-receptor matrix format. 2 - convert pollutant from species #1 to species #2. 3 - enable pm10 dust storm emission module. 4 - configure concentration grid similar to meteorology grid. 5 - treat 3D particle deposition using probability function. 7 - enable water surface transport of particle deposition. Defaults to 0
`iconvect` | flag for convection. If set to 1, then runs excessive convection as described in Gerbig et al., 2003. For specialized RAMS output, the particles will be vertically redistributed according to the output convective mass fluxes; defaults to 0
`isot` | flag used to set the isotropic turbulence option; defaults to 0 to compute horizontal turbulence from wind field deformation. Setting to 1 results in the horizontal turbulence to be the same in both the u and v directions
`kbls` | boundary layer stability derived from 1 - heat and momentum fluxes or 2 - wind and temperature profiles. Defaults to 1
`kblt` | boundary layer turbulence parameterization. 1 - Beljaars/Holtslag and Betchov/Yaglom. 2 - Kanthar/Clayson. 3 - TKE field from input meteorology data file. 4 - velocity variances from input meteorology. Defaults to 1
`kdef` | horizontal turbulence computation. 0 - in proportion to vertical turbulence. 1 - computed from velocity deformation. Defaults to 1
`khmax` | max age a particle is allowed to attain; defaults to 9999
`kmix0` | minimum mixing depth (abs(kmix0) is used as the minimum mixing depth), negative values are used to force mixing heights coincident with model levels; defaults to 250
`kmixd` | PBL height computation: compute from bulk Ri profile (but see zicontroltf); defaults to 3
`kmsl` | starting heights default to AGL = 0 or MSL = 1; defaults to 0
`kpuff` | horizontal puff dispersion. 0 - linear. 1 - square root. Defaults to 0
`krnd` | at this interval in hrs, enhanced puff merging occurs; defaults to 6
`kspl` | standard splitting interval (hours); defaults to 1
`kzmix` | vertical mixing adjustments. 0 - none, vertical diffusivity in PBL varies with height. 1 - vertical diffusivity in PBL single average value. 2 - scale PBL values by tvmix. 3 - scale free-troposphere values by tvmix. Defaults to 1
`maxdim` | maximum number of pollutants to carry on one mass particle; defaults to 1
`maxpar` | maximum number of particles carried in simulation; defaults to 10,000
`mgmin` | determines the size of the sub domain, set >1000 when working with high-res WRF-ARW met fields
`ncycl` | pardump output cycle time; defaults to 0
`ndump` | flag to dump all particle/puff points at the end of a simulation to a file called PARDUMP. This can be read at the start of a new simulation to continue the previous calculation. Valid settings include 0 (no i/o), 1 (read/write), 2 (read only), 3 (write only); defaults to 0
`ninit` | particle initialization. 0 - none. 1 - once. 2 - add. 3 - replace. Defaults to 1
`nturb` | no turbulence flag; defaults to 0, which includes turbulence rather than simulating mean trajectories
`outdt` | interval [min] to output data to PARTICLE.DAT; defaults to 0.0, which outputs at every timestep
`outfrac` | the fraction of the particles that are allowed to leave the model domain (given by met data); defaults to 0.9. If exceeded, the model stops
`p10f` | dust threshold velocity sensitivity factor; defaults to 1
`qcycle` | optional cycling of emissions (hours); defaults to 0
`random` | flag that tells the random number generator whether to have a different random sequence for each model run (0 - false, 1 - true); defaults to 1
`splitf` | horizontal flow splitting. 0 - disable. 1 - automatic size adjustment. >1 - constant value. Defaults to 1
`tkerd` | day (unstable) turbulent kinetic energy ratio; defaults to 0.18
`tkern` | night (stable) turbulent kinetic energy ratio; defaults to 0.18
`tlfrac` | the fraction of the lagrangian timescale TL to set as timestep in dispersion subroutine. The smaller this fraction is, the more finely the turbulence is resolved; defaults to 0.1
`tratio` | maximum fraction of gridcell to be travelled by a particle in a single integration timestep. This determines the timestep if DELT is set to be dynamic
`tvmix` | vertical mixing scale factor; defaults to 1
`veght` | height below which a particle's time spent is tallied; defaults to 0.5, which specifies half of the PBL. Setting <=1.0 specifies a fraction of the boundary layer height, and setting >1.0 specifies a height above ground in meters
`vscale` | vertical Lagrangian time scale (sec); defaults to 200
`w_option` | vertical motion calculation method. 0: use vertical velocity from data, 1: isob, 2: isen, 3: dens, 4: sigma; defaults to 0
`zicontroltf` | flag that specifies whether to scale the PBL heights in STILT uniformly in the entire model domain; defaults to 0. If set to 1, then STILT looks for a file called "ZICONTROL" that specifies the scaling for the PBL height. The first line indicates the number of hours that the PBL height will be changed, and each subsequent line indicates the scaling factor for that hour
`ziscale` | manually scale the mixed-layer height, with each element specifying a scaling factor for each simulation hour (ziscale can be of length that is smaller than abs(nhrs). A vector can be passed as a list (e.g. `ziscale <- list(rep(0.8, 24))` scales the mixed layer height to 80% for the first 24 hours of all simulations) or a list of vectors specific to each simulation (e.g. `ziscale <- rep(list(rep(0.8, 24)), nrow(receptors))`)
`z_top` | top of model domain, in meters above ground level; defaults to 25000.0


### Transport error calculations

Arg             | Description
----------------|----------------------------------------------------------------
`siguverr` | standard deviation of horizontal wind errors [m/s]
`tluverr` | standard deviation of horizontal wind error timescale [min]
`zcoruverr` | vertical correlation lengthscale [m]
`horcoruverr` | horizontal correlation lengthscale [km]
`sigzierr` | standard deviation of mixed layer height errors [%]
`tlzierr` | standard deviation of mixed layer height timescale [min]
`horcorzierr` | horizontal correlation lengthscale of mixed layer height errors [km]
