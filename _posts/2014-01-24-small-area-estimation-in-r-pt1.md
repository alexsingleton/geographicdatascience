---
author: Michail Pavlis
layout: post
avatar: michailpavlis
title: Small Area Estimation in R - Part 1
comments: true
categories:
- R
---


A substantial part of my work has to do with statistics and mostly in the area of geodemographics and [Small Area Estimation][1] (SAE). Basically what I am trying to do is to update the [Output Area Classification][2] (OAC) for England by producing intercensal estimates of the OAC variables.  Some of these estimates are already available, such as population estimates, but for the most part we have to create statistical models using secondary data sources such as school data, council tax bands, indices of deprivation and so on. <!-- more -->

## Background
There are two major distinctions in regards with the statistical methods used in SAE, the direct and indirect estimators. The former makes use of domain-specific data and design-based weights while the latter  mostly rely on the use of auxiliary data. Further distinctions of indirect estimators can be made by identifying whether they “borrow strength” from a different domain, time or a combination of the two. Indirect estimators are also called model-based estimators and use techniques such as mixed-effects models and the empirical best linear unbiased prediction (EBLUP), hierarchical Bayes and empirical Bayes. A wealth of information on SAE can be found in the book Small Area Estimation by Rao.

Mixed-effects models are applied to data which are nested (also referred to as hierarchical or multilevel). Geodemographic data are nested because there is a spatial hierarchy, for example output areas are within lower super output areas. This hierarchy can be included in the random part of the model while the fixed part of the model consists of the explanatory variables. More information on mixed-effects models can be found [here][3]. 

My own background is in the area of environmental sciences and I had the opportunity to use mixed-effects models, relying mostly on the excellent book by Zuur et al “Mixed effects Models and extentions in ecology using R”.  I have used in the past several packages in R to develop mixed effects models, including nlme, mgcv and lme4. There are several options for Bayesian analysis in R as well, but what I quickly found out was that running a Bayesian model at the national scale can take an enormous amount of time. This makes difficult to test and validate these models so in my case it was impossible to consider using Bayesian models.

## Example
The type of model that I used was dictated by the type of data that I was interested to predict. Most of the variables that were used to develop the OAC were proportional data and therefore a generalized mixed-effects model (GLMM) with a binomial distribution had to be applied for their prediction (having said that, it has been suggested that a linear mixed effects model sometimes can provide better results, see for example this [paper][4]). lme4 is a good package to start exploring GLMM models in R and I will provide an example of using lme4 later in this post. Aspects of the analysis that one should pay particular attention include:
  * Exploratory Data Analysis (identification of suitable predictors, outliers, non-linearity).
  * Model Selection for the fixed part and the random part of the regression.
  * Internal validation (based on the residuals).
  * Overdispersion (i.e. the variation in the data exceeds the expected variability based on the binomial distribution assumptions).

If you want to reproduce the analysis you will have to download the 2001 Census data on [Routine/Semiroutine Occupation][5]. So download the files for the following regions: East Midlands, East of England, London, North East, North West, South East, South West, West England and Yorkhire (9 files in total). Select csv as file type, download the zip file in a folder and then unzip the file ending in "\_OA.CSV". The following R script will select the appropriate columns, merge the data and save them in a new csv file.


```r
cols = c(12, 15, 28, 30)
setwd("path/to/census_data")  # Provide the path to Census 2001 csv files
csv_files = list.files(".", ".CSV")
temp <- NULL
for (i in csv_files) {
    pop <- read.csv(i, skip = 5, header = T)
    pop <- pop[, cols]
    names(pop) <- c("oa", "all_people", "semi_routine_occupation", "routine_occupation")
    temp <- rbind(temp, pop)
}
out <- data.frame(oa_01 = temp[, 1], all_people_01 = temp[, 2])
out$routine_count_01 <- rowSums(temp[, 3:4])
out$routine_perc_01 <- rowSums(temp[, 3:4])/temp[, 2]

write.csv(out, "occupation_2001.csv", row.names = F)

```

Consequently download the [lookup table][6] between OA 2001 and LSOA and unzip the csv file. You will also need the Indices of Deprivation 2004 (Education), so go ahead and download it [here][7]. Again select csv as data type and from the zipped file export the file ending to "_LSOA.CSV". We will also need the council tax bands data for 2001, which you can find [here][8]. Like the 2001 census data, tax bands are available for each region so save them in a new folder and then merge them to a single file using the following script.


```r
setwd("path/to/tax_bands_data")  # Provide the path to tax bands data
csv_files <- list.files(".", "*.CSV")
temp <- NULL
cols <- c(12, 15, 16, 18, 20, 22, 24, 26, 28, 30)
for (i in csv_files) {
    read_csv <- read.csv(i, skip = 5, header = T)[, cols]
    names(read_csv) <- c("oa_01", "all_bands", "band_a", "band_b", "band_c", 
        "band_d", "band_e", "band_f", "band_g", "band_h")
    temp <- rbind(temp, read_csv)
}

write.csv(temp, "tax_bands_2001.csv", row.names = F)
```

Finally we can put all the data together and start with the analysis.


```r
library(lme4)
source(file = "http://www.highstat.com/Book2/HighstatLibV6.R")
```



```r
census_01 <- read.csv("path/to/occupation_2001.csv")[, 1:4]

lookup <- read.csv("path/to/OA01_LSOA01_MSOA01_EW_LU.csv")[, 1:2]

imd_education <- read.csv("path/to/IMD_Education_2004.csv", skip = 5)[, c(9,14)]
names(imd_education) <- c("LSOA01CD", "rank_education")

bands_01 <- read.csv("path/to/tax_bands_2001.csv")
bands_01$a_perc <- ifelse(bands_01$all_bands == 0, 0, bands_01$band_a/bands_01$all_bands)

census_01 <- merge(census_01, lookup, by.x = "oa_01", by.y = "OA01CD")
all_data <- merge(census_01, bands_01[, c("oa_01", "a_perc")], by.x = "oa_01", 
    by.y = "oa_01")
all_data <- merge(all_data, imd_education[, c("LSOA01CD", "rank_education")], 
    by.x = "LSOA01CD", by.y = "LSOA01CD")

all_data$tr_a_perc <- asin(sqrt(all_data$a_perc/(100 + 1))) + asin(sqrt((all_data$a_perc + 1)/(100 + 1)))

str(all_data)
# 'data.frame': 165665 obs. of 9 variables: 
# $ LSOA01CD : Factor w/ 34378 levels 'E01000001','E01000002',..: 1 1 1 1 1 1 1 1 2 2 ...  
# $ oa_01 : Factor w/ 165665 levels '00AAFA0001','00AAFA0002',..: 1 2 3 4 5 6 7 8 17
# $ all_people_01 : int 181 92 180 246 206 197 164 148 199 189 ...
# $ routine_count_01: int 6 0 3 3 0 7 9 3 0 6 ...  
# $ routine_perc_01 : num 0.0331 0 0.0167 0.0122 0 ...  
# $ a_perc : num 0 0 0 0 0 ...  
# $ rank_education : int 28215 28215 28215 28215 28215 28215 28215 28215 28452
# $ tr_a_perc : num 0.0997 0.0997 0.0997 0.0997 0.0997 ...
```


The response variable is routine\_perc\_01, while the explanatory variables are a\_perc (Tax Band A) and rank\_education. I should note that I prefer to use the Education Rank instead of Education Score as the former is ordinal data and we will have less problems with the regression analysis. I have also transformed the variable Band A using the Freeman - Tukey double arcsine transformation which is suitable for percentage data. You can see the effect of the transformation in the following QQ-plot, the main body of the data distribution is closer to normal.

![QQplot](/public/images/qqplots.jpeg)

You can create the QQ-plot as follows:


```r
par(mfrow = c(1, 2))
for (i in c("a_perc", "tr_a_perc")) {
    qqnorm(all_data[[i]], main = i)
    qqline(all_data[[i]])
}
```


### Exploratory Data Analysis
The next step is to perform EDA and we can start with looking at the correlations among the variables.

```r
cor(all_data[,c(5,6:8)])
#                 routine_perc_01     a_perc rank_education  tr_a_perc
# routine_perc_01       1.0000000  0.5894138     -0.7334502  0.5974160
# a_perc                0.5894138  1.0000000     -0.6021874  0.9795349
# rank_education       -0.7334502 -0.6021874      1.0000000 -0.5994592
# tr_a_perc             0.5974160  0.9795349     -0.5994592  1.0000000
```

Education rank seems to be a good predictor and Band A is also related to the dependent variable.
We can check for outliers:

```r
vars <- c("a_perc", "tr_a_perc", "rank_education")
Mydotplot(all_data[, c(vars, "routine_perc_01")])
```


![dotplot](/public/images/dotplot.jpeg)

And visualize the relationship between the variables.

```r
Myxyplot(all_data, vars, "routine_perc_01")
```


![xyplot](/public/images/xyplot.jpeg)

The functions Mydotplot and Myxyplot are available from the HighstatLibV6.R script that we imported earlier. Outliers do not appear to be a significant problem for our data and the relationships seem to be linear.

### Regression Analysis
Before we start with regression analysis we need to create the response variable which should consist of two columns, the number of people working in a routine occupation and the number of people who don't. We will also use the LSOA code in the random part of the regression. We will first use the untransformed percentage of Band A. The code in R is as follows:


```r
y <- cbind(all_data$routine_count_01, all_data$all_people_01 - all_data$routine_count_01)

glm1 <- glmer(y ~ a_perc + rank_education + (1 | LSOA01CD), data = all_data, 
    family = binomial)

summary(glm1)
# Generalized linear mixed model fit by maximum likelihood ['glmerMod']
# Family: binomial ( logit )
# Formula: y ~ a_perc + rank_education + (1 | LSOA01CD) 
#   Data: all_data 

#       AIC       BIC    logLik  deviance 
# 1428001.9 1428041.9 -713996.9 1427993.9 

# Random effects:
#  Groups   Name        Variance Std.Dev.
#  LSOA01CD (Intercept) 0.08882  0.298   
# Number of obs: 165665, groups: LSOA01CD, 32482

# Fixed effects:
#                  Estimate Std. Error z value Pr(>|z|)    
# (Intercept)    -9.541e-01  3.725e-03  -256.1   <2e-16 ***
# a_perc          5.367e-01  2.492e-03   215.3   <2e-16 ***
# rank_education -3.607e-05  1.912e-07  -188.6   <2e-16 ***
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

# Correlation of Fixed Effects:
#             (Intr) a_perc
# a_perc      -0.404       
# rank_eductn -0.873  0.287
```


All the parameters are significant, the next step is to perform internal validation of the model.


```r
# E1 <- residuals(glm1, type="pearson")
# summary(E1)
#       Min.    1st Qu.     Median       Mean    3rd Qu.       Max. 
# -14.200000  -1.044000  -0.062490  -0.007915   0.957100  18.520000 
# p1 <- length(fixef(glm1)) + 1
# overdispersion1 <- sum(E1 ^ 2) / (nrow(all_data) - p1)  # 2.615604, should be around 1
# F1 <- fitted(glm1)
# summary(F1)
#    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
# 0.01805 0.14610 0.20170 0.20960 0.26850 0.50770 
summary(all_data$routine_perc_01)
#    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#  0.0000  0.1366  0.2021  0.2094  0.2764  0.5666 
cor(all_data$routine_perc_01, F1)  # 0.8765911

par(mfrow = c(2, 1))
plot(F1, E1, xlab = "Fitted values", ylab = "Pearson residuals")
abline(h = 0, lty = 2)
plot(F1, all_data$routine_perc_01, xlab = "Fitted values", ylab = "Observed data")
abline(coef = c(0, 1), lty = 2)

all_data$E1 <- E1
vars <- c("a_perc", "rank_education")
Myxyplot(all_data, vars, "E1")
```


![glm1_plot1](/public/images/glm1_plot1.jpeg)
![glm1_plot2](/public/images/glm1_plot2.jpeg)

Based on the internal validation, the model is overdispersed and we don't predict extremely high and low values (nevertheless we do predict within the physical range). Plotting the Pearson residuals against predicted values there is greater variance in the middle of the graph compared to the variance on the right of the graph. The correlation of the predicted values against the actual values seems to be good. The plot of Pearson residuals against Band A shows that we have greater variance at the ends of the plot, therefore a transformation of the variable might be useful. Rank education data against the residuals does not show any serious patterns. 
So the next step is to repeat the regression analysis using the Band A transformed data.


```r
glm2 <- glmer(y ~ tr_a_perc + rank_education + (1 | LSOA01CD), data = all_data, 
    family = binomial)

summary(glm2)
# Generalized linear mixed model fit by maximum likelihood ['glmerMod']
#  Family: binomial ( logit )
# Formula: y ~ tr_a_perc + rank_education + (1 | LSOA01CD) 
#    Data: all_data 

#       AIC       BIC    logLik  deviance 
# 1429994.9 1430035.0 -714993.5 1429986.9 

# Random effects:
#  Groups   Name        Variance Std.Dev.
#  LSOA01CD (Intercept) 0.08436  0.2904  
# Number of obs: 165665, groups: LSOA01CD, 32482

# Fixed effects:
#                  Estimate Std. Error z value Pr(>|z|)    
# (Intercept)    -1.342e+00  4.744e-03  -282.9   <2e-16 ***
# tr_a_perc       3.622e+00  1.722e-02   210.3   <2e-16 ***
# rank_education -3.636e-05  1.870e-07  -194.5   <2e-16 ***
---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

# Correlation of Fixed Effects:
#             (Intr) tr__pr
# tr_a_perc   -0.713       
# rank_eductn -0.787  0.291
```

```r
E2 <- residuals(glm2, type = "pearson")
summary(E2)
#      Min.    1st Qu.     Median       Mean    3rd Qu.       Max. 
# -14.200000  -1.050000  -0.063270  -0.007156   0.963700  18.810000 
p2 <- length(fixef(glm2)) + 1
overdispersion2 <- sum(E2 ^ 2) / (nrow(all_data) - p2)  # 2.638352
F2 <- fitted(glm2)
summary(F2)
#    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
# 0.01724 0.14590 0.20200 0.20960 0.26950 0.49210 
summary(all_data$routine_perc_01)
#    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
#  0.0000  0.1366  0.2021  0.2094  0.2764  0.5666 
cor(all_data$routine_perc_01, F2)  # 0.8755668

par(mfrow = c(2, 1))
plot(F2, E2, xlab = "Fitted values", ylab = "Pearson residuals")
abline(h = 0, lty = 2)
plot(F2, all_data$routine_perc_01, xlab = "Fitted values", ylab = "Observed data")
abline(coef = c(0, 1), lty = 2)

all_data$E2 <- E2
vars <- c("tr_a_perc", "rank_education")
Myxyplot(all_data, vars, "E2")
```


![glm2_plot1](/public/images/glm2_plot1.jpeg)
![glm2_plot2](/public/images/glm2_plot2.jpeg)

With regard to the second model, the use of the transformed variable appears to have slightly increased the AIC, so one could argue against transforming BAND A. Nevertheless, the plot of residuals against the transformed variable shows that heterogeneity was decreased so I decided to use the transformed variable. The final step of the analysis is to deal with overdispersion and to do this a random intercept for each entry was included in the model.


```r
all_data$Eps <- 1:nrow(all_data)

glm3 <- glmer(y ~ tr_a_perc + rank_education + (1 | LSOA01CD) + (1 | Eps), data = all_data, 
    family = binomial)
summary(glm3)
# Generalized linear mixed model fit by maximum likelihood ['glmerMod']
#  Family: binomial ( logit )
# Formula: y ~ tr_a_perc + rank_education + (1 | LSOA01CD) + (1 | Eps) 
#    Data: all_data 

#       AIC       BIC    logLik  deviance 
# 1286541.7 1286591.8 -643265.9 1286531.7 

# Random effects:
#  Groups   Name        Variance Std.Dev.
#  Eps      (Intercept) 0.07189  0.2681  
#  LSOA01CD (Intercept) 0.06475  0.2545  
# Number of obs: 165665, groups: Eps, 165665; LSOA01CD, 32482

# Fixed effects:
#                  Estimate Std. Error z value Pr(>|z|)    
# (Intercept)    -1.263e+00  6.474e-03  -195.1   <2e-16 ***
# tr_a_perc       3.156e+00  2.849e-02   110.8   <2e-16 ***
# rank_education -3.806e-05  1.961e-07  -194.1   <2e-16 ***
# ---
# Signif. codes:  0 ‘***’ 0.001 ‘**’ 0.01 ‘*’ 0.05 ‘.’ 0.1 ‘ ’ 1

# Correlation of Fixed Effects:
#             (Intr) tr__pr
# tr_a_perc   -0.867       
# rank_eductn -0.783  0.462

E3 <- residuals(glm3, type = "pearson")
summary(E3)
#      Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
# -4.024000 -0.329100  0.001431 -0.039470  0.295900  2.959000 
p3 <- length(fixef(glm3)) + 1
overdispersion3 <- sum(E3 ^ 2) / (nrow(all_data) - p3)  # 0.2939502

all_data$E3 <- E3
```

With that overdispersion for the last model is no longer a problem. The next part of the analysis is to predict new values of the response variable for the period 2002 - 2010, which will be presented in the next post.

[1]: http://en.wikipedia.org/wiki/Small_area_estimation
[2]: http://areaclassification.org.uk/getting-started/getting-started-what-is-the-output-area-classification/
[3]: http://zoonek2.free.fr/UNIX/48_R/14.html
[4]: http://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=3&ved=0CDQQFjAC&url=http%3A%2F%2Fwww.researchgate.net%2Fpublication%2F256972075_Use_of_Spatial_Information_in_Small_Area_Models_for_Unemployment_Rate_Estimation_at_Sub-Provincial_Areas_in_Italy%2Ffile%2F504635241a2a1c8159.pdf&ei=fsvnUqWSENSB7Qab-oHYCw&usg=AFQjCNECEufo24VHT_f3AYyJi9I1JfvLoA&bvm=bv.59930103,d.ZGU
[5]: http://www.neighbourhood.statistics.gov.uk/dissemination/instanceSelection.do?JSAllowed=true&Function=&%24ph=60_61&CurrentPageId=61&step=2&datasetFamilyId=41&instanceSelection=121&Next.x=16&Next.y=5
[6]: https://geoportal.statistics.gov.uk/Docs/Lookups/Output_areas_(2001)_to_lower_layer_super_output_areas_(2001)_to_middle_layer_super_output_areas_(2001)_E+W_lookup.zip
[7]: http://www.neighbourhood.statistics.gov.uk/dissemination/instanceSelection.do?JSAllowed=true&Function=&%24ph=60_61&CurrentPageId=61&step=2&datasetFamilyId=2088&instanceSelection=124670&Next.x=13&Next.y=18
[8]: http://www.neighbourhood.statistics.gov.uk/dissemination/datasetList.do?JSAllowed=true&Function=&%24ph=60&CurrentPageId=60&step=1&CurrentTreeIndex=-2&searchString=tax+bands&datasetFamilyId=938&Next.x=13&Next.y=7