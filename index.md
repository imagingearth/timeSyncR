timeSyncR
========================================================

## Introduction

Welcome to the timeSyncR page! timeSyncR is a package loosely based on the timeSync method for interpreting satellite image time series for calibration and validation of change methods and products. This version has been developed with and for RStudio. 

At the moment only Landsat data are supported. In theory, however, any raster data should work, provided the write metadata are there (more on that later).



## tsChipsRGB demo

One of the main functions of the timeSyncR packages is the `tsChipsRGB()` function. This function takes several time series RasterBricks and displays a given extent for each date as an RGB composite using the `plotRGB()` function of the raster package. In this demo, we will look at a small area in Madre de Dios, Southeastern Peru. We will use `tsChipsRGB()` to check if, where and when deforestation has taken place.

First, we have to install and load the package:


```r
library(devtools)
install_github('bendv/timeSyncR')
library(timeSyncR)
```

`timeSyncR` comes with an external .tar.gz file containing three RasterBricks (in .grd format) from the three visible bands of the Landsat sensors. To load these files, we first need to find the .tar.gz, untar it and store it in a designated folder. Set your working directory and make a 'test' directory in it. Then, identify the .tar.gz file and copy it to your test directory.


```r
setwd('path/to/working/directory/')
dir.create('test')
fl <- system.file('extdata', 'MDD.tar.gz', package = 'timeSyncR')
newfl <- 'test/MDD.tar.gz'
file.copy(fl, newfl)
```

Now it's time to untar the copy of the tar file you made.


```r
untar(newfl, exdir = 'test')
```

If you look in the 'test' folder, you should see three .grd files and their associated .gri files: blue, green and red. Load these into your workspace and check out these rasters


```r
r <- brick('test/MDD_red.grd')
g <- brick('test/MDD_green.grd')
b <- brick('test/MDD_blue.grd')

nlayers(r)
plot(g, 1:9)

# layer names correspond to Landsat ID's
names(b)
# see ?getSceneinfo to get more info out of these
s <- getSceneinfo(names(b))
print(s)

# each brick has a z-dimension containing date info
getZ(g)
# also found in the date column of the getSceneinfo() data.frame
s$date
all(getZ(g) == s$date)
```

