---
author: Nick Bearman
layout: post
avatar: nickbearman
title: Reprojecting Crime data from Police.uk from Latitude & Longitude (WGS84) to Eastings & Northings (British National Grid)
comments: true
categories:
- R
---

I have recently taken over a second year undergraduate module, 'Applied GIS and Modelling'. The first practical element of the course (spread over 8 hours of lab sessions over the first 2 weeks) introduces the students to GIS and using R to show, create and handle spatial data. One of the exercises uses crime data downloaded from the Police.uk website.

While going through the material, I discovered that the crime data available through the Police.uk website had changed format. <!-- more --> The data file used last year previously contained coordinates in Eastings and Northings (the [British National Grid](http://www.ordnancesurvey.co.uk/resources/maps-and-geographic-resources/the-national-grid.html) projection system) now had coordinates specified in Latitude and Longitude ([WGS84](http://spatialreference.org/ref/epsg/4326/)). The students go on to plot the crime data over some LSOA boundaries (which come projected in BNG) so it would be useful to have the crime data in the same projection. This is in the first practical, so while the students do need to learn about projections at some point, I would prefer not to have to throw them in at the deep end to begin with!

However, the data had changed, so I needed to find a simple way of reprojecting them. Fortunately with a few tweaks to the R code, I managed to incorporate this into the practical in a way that wasn't too complex. The full practical is available at <http://rpubs.com/nickbearman/gettingstartedwithr>, and the bit relevant to this post begins about half-way down, at 'Working with Open Government Data'. The steps to download the crime data from Police.uk are straight forward, and are covered in the practical. The students use data from Merseyside Police, as this is their local police area. This is all done using the MapTools library in R. 



```r
	#Load the library
		require(maptools) 
	#Read in the data
		crimes <- read.csv("2013-11-merseyside-street.csv") 
```

The next step reads the CSV data (with latitude and longitude fields) into a SpatialPointsDataFrame:

```r
	#Show the head of the data
		head(crimes)
	#Create the coordinates
		coords <- cbind(Longitude = as.numeric(as.character(crimes$Longitude)), Latitude = as.numeric(as.character(crimes$Latitude))) 
	#Create SpatialPointsDataFrame, specifing coordinates, data and projection
		crime.pts <- SpatialPointsDataFrame(coords, crimes[, -(5:6)], proj4string = CRS("+init=epsg:4326"))
```

From the practical:

>"This creates a SpatialPointsDataFrame object. This first line prepares the coordinates into a form that the SpatialPointsDataFrame can use. The SpatialPointsDataFrame function on the second line takes three arguments - the first is coordinates, created in the line above. The second argument is the data frame minus columns 5 and 6 - this is what `-(5:6)` indicates. The third argument is the projection. These columns provide all the non-geographical data from the data frame. The resulting object crime.pts is a spatial points geographical shape object, whose points are each recorded crime in the data set you download."

Creating a SpatialPointsDataFrame from the CSV is straight forward, but remember to specify the projection - `proj4string = CRS("+init=epsg:4326")` in our case (WGS84). The `[,-(5:6)]` means that the data element of the SpatialPointsDataFrame contains everything in crimes (the CSV file read in) apart from columns 5 & 6. This is a new way of doing it to me, but quite effective!

Once the data is stored as a SpatialPointsDataFrame, the actual reprojection is easy - you essentially just say 'reproject this data frame to this projection', with `+init=epsg:27700` being BNG:

```r
	#Reproject data to BNG (27700)	
		crime.pts <- spTransform(crime.pts, CRS("+init=epsg:27700"))
```

As with many things in R, if it's setup correctly, then a reprojection is quite easy to do (just one line in this example!). However, it is setting up the data correctly which takes time. Previously both the shape file of the LSOAs and the crime data were read in without any projection - fine if they are both the same, but to reproject it, both need their projections specifying. 

Hopefully this R code will be useful to you - feel free to use it in any of your work. 