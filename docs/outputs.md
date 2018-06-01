---
layout: docs
title: STILT Outputs
---

# Outputs

The model outputs can be found nested within the `out/` directory (see [Structure]({{"/docs/structure.html"|relative_url}})). STILT outputs two files for later use. Particle trajectories are saved to a `_traj.rds` file and gridded footprints saved to a `_foot.nc` file.

### Particle Trajectories

Particle trajectories and simulation parameter information are packaged and saved in a compressed `.rds` file ([serialized single R object](https://stat.ethz.ch/R-manual/R-devel/library/base/html/readRDS.html)) with the naming convention with the naming convention `YYYYMMDDHH_LONG_LATI_ZAGL_traj.rds`. This allows regridding of the footprints without recalculating particle trajectories.

This object is can be loaded with `readRDS()` and is structured as

```r
out <- readRDS('YYYYMMDDHH_LONG_LATI_ZAGL_foot.rds')
str(out)
# List of 4
#  $ runtime : POSIXct[1:1], format: "2015-06-16"
#  $ file    : chr "/uufs/chpc.utah.edu/common/home/u0791983/stilt-sims/test/out/2015061600_-111.847672_40.766189_10/2015061600_-111.847672_40.7661"| __truncated__
#  $ receptor:Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	1 obs. of  4 variables:
#   ..$ run_time: POSIXct[1:1], format: "2015-06-16"
#   ..$ lati    : num 40.8
#   ..$ long    : num -112
#   ..$ zagl    : num 10
#  $ particle:Classes ‘tbl_df’, ‘tbl’ and 'data.frame':	128441 obs. of  26 variables:
#   ..$ time: num [1:128441] -2 -2 -2 -2 -2 -2 -2 -2 -2 -2 ...
#   ..$ indx: num [1:128441] 1 2 3 4 5 6 7 8 9 10 ...
#   ..$ long: num [1:128441] -112 -112 -112 -112 -112 ...
#   ..$ lati: num [1:128441] 40.8 40.8 40.8 40.8 40.8 ...
#   ..$ zagl: num [1:128441] 61.3 100.6 87.3 98.3 88.6 ...
#   ..$ sigw: num [1:128441] 1.15 1.15 1.15 1.15 1.15 ...
#   ..$ tlgr: num [1:128441] 8.05 8.05 8.05 8.05 8.05 ...
#   ..$ zsfc: num [1:128441] 1532 1532 1532 1532 1532 ...
#   ..$ icdx: num [1:128441] 2 2 2 2 2 2 2 2 2 2 ...
#   ..$ temp: num [1:128441] 301 301 301 301 301 ...
#   ..$ samt: num [1:128441] 2 2 2 2 2 ...
#   ..$ foot: num [1:128441] 0.0219 0.0219 0.0219 0.0219 0.0219 ...
#   ..$ shtf: num [1:128441] 0 0 0 0 0 0 0 0 0 0 ...
#   ..$ tcld: num [1:128441] 87.9 87.9 87.9 87.9 87.9 ...
#   ..$ dmas: num [1:128441] 1.104 1.104 1.104 0.991 1.119 ...
#   ..$ dens: num [1:128441] 0.968 0.965 0.966 0.965 0.966 ...
#   ..$ rhfr: num [1:128441] 0.277 0.282 0.28 0.282 0.28 ...
#   ..$ sphu: num [1:128441] 0.00222 0.00222 0.00222 0.00222 0.00222 ...
#   ..$ solw: num [1:128441] -99 -99 -99 -99 -99 -99 -99 -99 -99 -99 ...
#   ..$ lcld: num [1:128441] -99 -99 -99 -99 -99 -99 -99 -99 -99 -99 ...
#   ..$ zloc: num [1:128441] -999 -999 -999 -999 -999 -999 -999 -999 -999 -999 ...
#   ..$ dswf: num [1:128441] 25.2 25.2 25.2 25.2 25.2 25.2 25.2 25.2 25.2 25.2 ...
#   ..$ wout: num [1:128441] 0.722 0.722 0.722 0.722 0.722 ...
#   ..$ mlht: num [1:128441] 329 329 329 329 329 ...
#   ..$ rain: num [1:128441] 4.02e-07 4.02e-07 4.02e-07 4.02e-07 4.02e-07 ...
#   ..$ crai: num [1:128441] -0.9 -0.9 -0.9 -0.9 -0.9 -0.9 -0.9 -0.9 -0.9 -0.9 ...
```

Particle trajectory data is stored in a data frame with columns corresponding with varsiwant and can be accessed with `out$particle`.

### Gridded Footprints

Footprints are packaged and saved in a compressed .nc file conforming to the [Climate and Forecast (CF) metadata convention](http://cfconventions.org) with the naming convention `YYYYMMDDHH_LONG_LATI_ZAGL_foot.nc`. This object contains information about the model domain, the grid resolution, and footprint values. This object is typically a three dimensional array with dimensions ordered (*x*, *y*, *z*). However, the object will only have dimensions (*x*, *y*) for time integrated footprints.


```bash
ncdump -h 2015061822_-111.980323_40.782561_5_foot.nc

  netcdf 2015061822_-111.980323_40.782561_5_foot.nc {
  dimensions:
  	lon = 550 ;
  	lat = 500 ;
  	time = 24 ;
  variables:
  	double lon(lon) ;
  		lon:units = "degrees_east" ;
  		lon:standard_name = "longitude" ;
  		lon:long_name = "longitude at cell center" ;
  	double lat(lat) ;
  		lat:units = "degrees_north" ;
  		lat:standard_name = "latitude" ;
  		lat:long_name = "latitude at cell center" ;
  	double time(time) ;
  		time:units = "seconds since 1970-01-01 00:00:00Z" ;
  		time:standard_name = "time" ;
  		time:long_name = "utc time" ;
  		time:calendar = "standard" ;
  	float foot(time, lat, lon) ;
  		foot:units = "ppm (umol-1 m2 s)" ;
  		foot:_FillValue = -1.f ;
  		foot:standard_name = "footprint" ;
  		foot:long_name = "stilt surface influence footprint" ;

  // global attributes:
  		:crs = "+proj=longlat" ;
  		:crs_format = "PROJ.4" ;
  		:documentation = "github.com/uataq/stilt" ;
  		:title = "STILT Footprint" ;
  		:time_created = "2018-05-14 19:49:09" ;
  }
```

For those familiar with raster operations, the default output adheres to the [CF-1.4 metadata conventions](http://cfconventions.org/) which is inherently compatible compatible with the R `raster` package. For more information about raster manipulation, the [Raster R package](https://geoscripting-wur.github.io/IntroToRaster/) is a good place to start.


#### Interfacing with Raster R package
If `time_integrate = TRUE`, the footprint .nc files can be loaded directly with `raster()`. If `time_integrate = FALSE`, the data can be loaded with `brick()`, which is a three dimensional version of a standard raster. The POSIX time (UTC seconds since 1970-01-01) is stored in the Z dimension and can be easily accessed with `getZ()`.

Using the [Raster R package](https://geoscripting-wur.github.io/IntroToRaster/), the data can be loaded with `brick()` and is structured as

```r
library(raster)
out <- brick('2015070211_-111.835_40.763_7_foot.nc')
out
# class       : RasterBrick
# dimensions  : 280, 390, 109200, 5  (nrow, ncol, ncell, nlayers)
# resolution  : 0.002, 0.002  (x, y)
# extent      : -112.3, -111.52, 40.39, 40.95  (xmin, xmax, ymin, ymax)
# ...
time <- as.POSIXct(getZ(out), tz = 'UTC', origin = '1970-01-01')
str(time)
 # POSIXct[1:5], format: "2015-07-02 05:00:00" "2015-07-02 06:00:00" ...
```

#### Manual loading with ncdf4 R package

Alternatively, the data can be loaded using standard netCDF methods.

```r
library(ncdf4)
nc <- nc_open('2015070211_-111.835_40.763_7_foot.nc')
nc
# File 2015070211_-111.835_40.763_7_foot.nc (NC_FORMAT_CLASSIC):
#      1 variables (excluding dimension variables):
#         float Footprint[longitude_center,latitude_center,time]   
#             units: ppm (umol-1 m2 s)
#             _FillValue: -1
#      3 dimensions:
#         longitude_center  Size:390
#             units: degrees_east
#             long_name: longitude_center
#             position: cell_center
#         latitude_center  Size:280
#             units: degrees_north
#             long_name: latitude_center
#             position: cell_center
#         time  Size:5
#             units: seconds since 1970-01-01
#             long_name: time
#             timezone: UTC
#     7 global attributes:
#         crs: +proj=longlat +ellpsWGS84
#         crs_format: PROJ.4
#         Conventions: CF-1.4
#         Title: STILT Footprint Output
#         Compatibility: raster::raster() and raster::brick()
#         Documentation: benfasoli.github.io/stilt
#         Author: Ben Fasoli
```

To get your feet wet, try one of the tutorials such as [simulating carbon dioxide for the William Browning Building at the University of Utah](https://github.com/uataq/stilt-tutorials/tree/master/01-wbb).
