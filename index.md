timeSyncR
========================================================

## Introduction

Welcome to the <a href="http://github.com/bendv/timeSyncR">timeSyncR</a> page! timeSyncR is a package loosely based on the TimeSync method for interpreting satellite image time series for calibration and validation of change methods and products. This version has been developed with and for RStudio. 

At the moment only Landsat data are supported. In theory, however, any raster data should work, provided the right metadata are there (more on that later).




## tsChipsRGB demo

One of the main functions of the timeSyncR packages is the `tsChipsRGB()` function. This function takes several time series RasterBricks and displays a given extent for each date as an RGB composite using the `plotRGB()` function of the raster package. In this demo, we will look at a small area in Madre de Dios, Southeastern Peru. We will use `tsChipsRGB()` to check if, where and when deforestation has taken place.

First, we have to install and load the package:


```r
library(devtools)
install_github("bendv/timeSyncR")
library(timeSyncR)
```


`timeSyncR` comes with an external .tar.gz file containing three RasterBricks (in .grd format) from the three visible bands of the Landsat sensors. To load these files, we first need to find the .tar.gz, untar it and store it in a designated folder. Set your working directory and make a 'test' directory in it. Then, identify the .tar.gz file and copy it to your test directory.


```r
setwd("path/to/working/directory/")
dir.create("test")
fl <- system.file("extdata", "MDD.tar.gz", package = "timeSyncR")
newfl <- "test/MDD.tar.gz"
file.copy(fl, newfl)
```


Now it's time to untar the copy of the tar file you made.


```r
untar(newfl, exdir = "test")
```


If you look in the 'test' folder, you should see three .grd files and their associated .gri files: blue, green and red. Load these into your workspace and check out these rasters


```r
r <- brick("test/MDD_red.grd")
g <- brick("test/MDD_green.grd")
b <- brick("test/MDD_blue.grd")

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


With these RasterBricks in our workspace, we are ready to look at some image chips using `tsChipsRGB`. It is always a good idea to check out the help file (`?tsChipsRGB`) before diving in. In this example, we will take an (x,y) coordinate and focus in on that point.


```r
xy <- c(472000, -1293620)
# save original (default) plotting parameters to workspace
op <- par()

tsChipsRGB(xr = r, xg = g, xb = b, loc = xy, start = "2000-01-01")
# reset plotting parameters (plotRGB() changes parameters internally)
par(op)
```


If you need to focus on a single pixel, you can use `pixelToPolygon()` to convert the coordinates to a SpatialPolygons object representing the outline of that pixel. The SpatialPolygons object can be supplied to `loc` instead of the xy vector in that case.


```r
pix <- pixelToPolygon(x = r, cell = xy)
plot(pix)
tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, start = "2000-01-01")
```


To display a plot of the time series of each of the bands, set `plot` to `TRUE`. This will show all three bands (corresponding to the red, green and blue channels) as stacked time series plots.


```r
tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, start = "2000-01-01", plot = TRUE)
par(op)
```


For more informative plot labels, set `plotlabs`:


```r
tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, start = "2000-01-01", plot = TRUE, 
    plotlabs = c("band 3", "band 2", "band 1"))
par(op)
```


The `percNA` argument controls the image chips that are 'allowed' in the series. A value of 100 allows all images (ie. max percNA = 100%), whereas a value of 0 omits all image chips with any NA's (from cloud cover or SLC-off gaps). So, to see only images with no NA's starting in 2005:


```r
tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, start = "2005-01-01", percNA = 0, 
    plot = TRUE)
par(op)
```


The number of NAs is computed on the fly, depending on the extent of the chips. If a different zoom is specified, this might affect the image dates that are ultimately included. The default value for `buff` is 17, which indicates the number of pixels in all directions from the centre pixel to be included in the chips (ie. `buff=17` results in a 35x35 pixel window). Zooming out by setting `buff` to a higher value will naturally result in more images being excluded (since there is a higher chance of getting a cloud or SLC-off gap in the chip extent).


```r
tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, buff = 50, start = "2005-01-01", 
    percNA = 0, plot = TRUE)
par(op)
```


The resulting chips can be exported as a `list` of raster bricks by setting `exportChips` to `TRUE`:


```r
b <- tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, buff = 50, start = "2005-01-01", 
    percNA = 0, exportChips = TRUE)
# e.g. plot the first time step
plotRGB(b$R[[1]], b$G[[1]], b$B[[1]])
par(op)
```


The time series for the centre pixel can also be exported as a `list` of ordered vectors (see the `zoo` package) by setting `exportZoo` to `TRUE`:


```r
z <- tsChipsRGB(xr = r, xg = g, xb = b, loc = pix, start = "2005-01-01", percNA = 0, 
    exportZoo = TRUE)
par(op)
par(mfrow = c(3, 1))
plot(z$R, ylab = "red", xlab = "time")
plot(z$G, ylab = "green", xlab = "time")
plot(z$B, ylab = "blue", xlab = "time")
par(op)
```



<a href="http://github.com/bendv/timeSyncR">Back to timeSyncR github page</a>
