---
author: Michail Pavlis
layout: post
avatar: michailpavlis
title: Small Area Estimation in R - Part 2
comments: true
categories:
- R
---

To continue from the previous post, I will show how to make predictions of people in Routine/Semiroutine occupations for the period 2002 - 2010 using the GLMM model which was presented in the previous post. The predictions will be made using both the fixed effects and the random effects of the model (at the LSOA level), which will result in what is known as the empirical best linear unbiased prediction (EBLUP). <!-- more -->

To carry out the analysis we will need to update the data for the fixed variables, i.e. rank education and (transformed) Band A percent. The data for Tax Bands for the period 2002 - 2010 can be found [here][1], as explained previously, apart from the year we need to select 9 regions (East Midlands, East of England, London, North East, North West, South East, South West, West England and Yorkhire), the type of file (csv) and extract the file ending in "\_OA.CSV". Before extracting the files you should create a base folder and within that folder create 9 more folders named after each year (2002 - 2010). Different OA codes are used from 2010 so we will need a [lookup table][2] between the 2001 and 2010 OA codes. Updates for the indicator of education deprivation are created every 3 years, so for 2007 can be found [here][3] and for 2010 [here][4]. For the years that the deprivation data are not available we will keep them constant.

We will first need to merge the tax bands data for each year. This can be accomplished with the following function, simply provide the path to the base folder.

```r
mergeTaxData <- function(path_to_base_folder){
  for (year in 2002:2010){ 
    setwd(paste(path_to_base_folder, year, sep = "/"))
    csv_files <- list.files(".", "*.CSV")
    temp <- NULL
    if (year < 2008){
      cols <- c(12, 15, 16, 18, 20, 22, 24, 26, 28, 30)
      } else {
      cols <- c(14, 17, 18, 20, 22, 24, 26, 28, 30, 32)
      }
    for (i in csv_files){
      read_csv <- read.csv(i, skip = 5, header = T)[,cols]
      temp <- rbind(temp, read_csv)
    }
    if (year != 2010){
        names(temp) <- c("oa_01", "all_bands", "band_a", "band_b", "band_c", "band_d", "band_e", "band_f", "band_g", "band_h")
        } else {
          names(temp) <- c("oa_11", "all_bands", "band_a", "band_b", "band_c", "band_d", "band_e", "band_f", "band_g", "band_h")
        }
    write.csv(temp, paste("tax_bands_",year,".csv",sep=""), row.names = F)
  }
}
```
So if the path to the base folder was C:\tax_bands then the function could be used as follows:
```{r exampleFun, eval = FALSE}
mergeTaxData("C:/tax_bands")
```
We could also check if it is possible to improve our model. For example if spatial autocorrelation was found in the residuals we could take advantage of that and krige the residuals to add the predicted values back to our predictions (from the GLMM model), a methodology which is known as rgeression kriging. We can evaluate if there is autocorrelation in the residuals using the variogram, but first we will need the coordinates of the centroids of the OAs. To do that, download the OA boundaries from the site of [Edina][5] (you will need to login to access the data). The following script will extract the coordinates and draw the variogram for the dependent variable and the residuals obtained from the GLMM model. If the curve of the variogram is increasing with distance then there is autocorrelation in the data.

```r
library(rgdal)
library(gstat)

oa_boundaries <- readOGR("/home/mick/GIS_Data", "England_oa_2001_clipped_area")
oa_coords <- coordinates(oa_boundaries) # Extract the coordinates of the centroids of the OA polygons
oa_coords <- data.frame(oa_01 = oa_boundaries@data[,2], easting = oa_coords[,1], northing = oa_coords[,2])
all_data2 <- merge(all_data, oa_coords, by.x = "oa_01", by.y = "oa_01")
coordinates(all_data2) <- ~ easting + northing # Convert to SpatialPointsDataFrame
vgm1 <- variogram(routine_perc_01 ~ 1, all_data2)
vgm2 <- variogram(E3 ~ 1, all_data2)
par(mfrow = c(1,2))
plot(vgm1)
plot(vgm2)
```
![vgm1](/public/images/vgm1.jpeg)
![vgm2](/public/images/vgm2.jpeg)

As can be seen from the variograms, autocorrelation was present in the dependent variable but it was accounted for after regression was applied. So the next step of the analysis, is to make predictions for the period 2002 - 2010, which can be accomplished using the following script.

```r
out <- data.frame(oa_01 = all_data$oa_01, routine_perc_01 = all_data$routine_perc_01)

lookup2 <- read.csv("path/to/lookup_01_11.csv")[,2:3]
names(lookup2) <- c("oa_01", "oa_11")

for (year in 2002:2010){
  
  if (year <= 2004){
    imd_education <- read.csv("path/to/IMD_Education_2004.csv", skip = 5)[,c(9,14)]
    names(imd_education) <- c("LSOA01CD", "rank_education_x")
  } else if (year > 2004 & year < 2008){
    imd_education <- read.csv("path/to//IMD_Education_2007.csv", skip = 5)[,c(9,14)]
    names(imd_education) <- c("LSOA01CD", "rank_education_x")
    } else {
      imd_education <- read.csv("path/to/IMD_Education_2010.csv", skip = 5)[,c(11,14)]
      names(imd_education) <- c("LSOA01CD", "rank_education_x")
    }
  
  bands <- read.csv(paste("/home/mick/dwelling/", year, "/tax_bands_", year,".csv", sep = ""))
  bands$a_perc_x <- ifelse(bands$bands_all  != 0, bands$band_a / bands$bands_all, 0)
  
  if (year == 2010){
    bands <- merge(bands, lookup2, by.x = "oa_11", by.y = "oa_11")
  }
  
  temp <- merge(all_data[,c("oa_01", "LSOA01CD", "a_perc", "rank_education")], 
                bands[,c("oa_01", "a_perc_x")], all.x = T, by.x = "oa_01", by.y = "oa_01")
  temp <- merge(temp, imd_education, all.x = T, by.x = "LSOA01CD", by.y = "LSOA01CD")
  
  # Replace any NAs
  temp$a_perc_x <- as.numeric(ifelse(is.na(temp$a_perc_x), temp$a_perc, temp$a_perc_x))
  temp$rank_education_x <- as.numeric(ifelse(is.na(temp$rank_education_x), temp$rank_education, temp$rank_education_x))
  
  temp$tr_a_perc_x <- asin(sqrt(temp$a_perc_x/(100 + 1))) + asin(sqrt((temp$a_perc_x + 1)/(100 + 1))) 
  
  X <- with(temp, data.frame(tr_a_perc =  tr_a_perc_x, rank_education = rank_education_x, LSOA01CD = LSOA01CD))
  
  pred_name <- paste("pred_routine_", year, sep = "")
  temp[[pred_name]] <- predict(glm3, X, REform =~ (1|LSOA01CD))
  temp[[pred_name]] <- exp(temp[[pred_name]]) / (1 + exp(temp[[pred_name]]))
  
  out <- merge(out, temp[,c("oa_01", pred_name)], all.x = T, by.x = "oa_01", by.y = "oa_01")
  out[[pred_name]] <- as.numeric(ifelse(is.na(out[[pred_name]]), out$routine_perc_01, out[[pred_name]]))
}

write.csv(out, "pred_routine.csv", row.names = F)
```
We can have a look at the correlations between the Census 2001 data and the predicted values.

```r
cor(out[,2:11])
#                   routine_perc_01 pred_routine_2002 pred_routine_2003 pred_routine_2004 pred_routine_2005 pred_routine_2006 pred_routine_2007 pred_routine_2008 pred_routine_2009 pred_routine_2010
# routine_perc_01         1.0000000         0.8711056         0.8710991         0.8711549         0.8655008         0.8654215         0.8654712         0.8616898         0.8524694         0.8614698
# pred_routine_2002       0.8711056         1.0000000         0.9998875         0.9994646         0.9882453         0.9882038         0.9882186         0.9811680         0.9682729         0.9806596
# pred_routine_2003       0.8710991         0.9998875         1.0000000         0.9995565         0.9883512         0.9883136         0.9883332         0.9812892         0.9683978         0.9807874
# pred_routine_2004       0.8711549         0.9994646         0.9995565         1.0000000         0.9887782         0.9887463         0.9887642         0.9817424         0.9687370         0.9811330
# pred_routine_2005       0.8655008         0.9882453         0.9883512         0.9887782         1.0000000         0.9997909         0.9994789         0.9885431         0.9757190         0.9880140
# pred_routine_2006       0.8654215         0.9882038         0.9883136         0.9887463         0.9997909         1.0000000         0.9997002         0.9887677         0.9759328         0.9881429
# pred_routine_2007       0.8654712         0.9882186         0.9883332         0.9887642         0.9994789         0.9997002         1.0000000         0.9890717         0.9762212         0.9884217
# pred_routine_2008       0.8616898         0.9811680         0.9812892         0.9817424         0.9885431         0.9887677         0.9890717         1.0000000         0.9871076         0.9993604
# pred_routine_2009       0.8524694         0.9682729         0.9683978         0.9687370         0.9757190         0.9759328         0.9762212         0.9871076         1.0000000         0.9868587
# pred_routine_2010       0.8614698         0.9806596         0.9807874         0.9811330         0.9880140         0.9881429         0.9884217         0.9993604         0.9868587         1.0000000
```

As can be seen from the correlation analysis, the correlation between the Census data and the predicted values is quite good and the correlation decreases over time as should be expected.

# Conclusion
Using a GLMM it was possible to make predictions for the percentage of people in routine semi-routine occupation, internal validation based on the residuals was used to evaluate the outcome of the analysis. Even though there is a good correlation between the Census data and the predictions, there is a stronger correlation between the predictions themselves which might indicate that we don't predict some of the changes that occur over time. A more elegant solution to the problem of overdispersion would be to apply Baysian statistics and to use a different distribution from binomial (e.g. beta-binomial).  



[1]: http://www.neighbourhood.statistics.gov.uk/dissemination/datasetList.do?JSAllowed=true&Function=&%24ph=60&CurrentPageId=60&step=1&CurrentTreeIndex=-2&searchString=tax+bands&datasetFamilyId=938&Next.x=13&Next.y=7
[2]: https://geoportal.statistics.gov.uk/Docs/Lookups/Output_areas_(2001)_to_output_areas_(2011)_to_local_authority_districts_(2011)_E+W_lookup.zip
[3]: http://www.neighbourhood.statistics.gov.uk/dissemination/instanceSelection.do?JSAllowed=true&Function=&%24ph=60_61&CurrentPageId=61&step=2&datasetFamilyId=2026&instanceSelection=123850&Next.x=11&Next.y=10
[4]: http://www.neighbourhood.statistics.gov.uk/dissemination/instanceSelection.do?JSAllowed=true&Function=&%24ph=60_61&CurrentPageId=61&step=2&datasetFamilyId=2394&instanceSelection=129711&Next.x=20&Next.y=13
[5]: http://ukbsrv-at.edina.ac.uk/html/easy_download/easy_download.html?data=England_oa_2001
