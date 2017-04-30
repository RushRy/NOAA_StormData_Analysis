# NOAA Storm Database Severe Weather Events Analysis
Ryan Rush  
April 29, 2017  

# Synopsis
The goal of this analysis is to explore the NOAA Storm Database to answer questions relating to severe weather events.

# Setup
This analysis requires the dplyr and ggplot2 libraries:

```r
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(ggplot2))
```

For purposes of clarity, we'll print information on the current environment.

```r
sessionInfo()
```

```
## R version 3.3.2 (2016-10-31)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 14393)
## 
## locale:
## [1] LC_COLLATE=English_United States.1252 
## [2] LC_CTYPE=English_United States.1252   
## [3] LC_MONETARY=English_United States.1252
## [4] LC_NUMERIC=C                          
## [5] LC_TIME=English_United States.1252    
## 
## attached base packages:
## [1] stats     graphics  grDevices utils     datasets  methods   base     
## 
## other attached packages:
## [1] ggplot2_2.2.1 dplyr_0.5.0  
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.9      digest_0.6.11    rprojroot_1.2    assertthat_0.1  
##  [5] plyr_1.8.4       grid_3.3.2       R6_2.2.0         gtable_0.2.0    
##  [9] DBI_0.5-1        backports_1.0.5  magrittr_1.5     scales_0.4.1    
## [13] evaluate_0.10    stringi_1.1.2    lazyeval_0.2.0   rmarkdown_1.4   
## [17] tools_3.3.2      stringr_1.1.0    munsell_0.4.3    yaml_2.1.14     
## [21] colorspace_1.3-2 htmltools_0.3.5  knitr_1.15.1     tibble_1.2
```

# Data Processing
The data for this analysis comes from the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage.

## Obtaining the data
We obtained the database from the <a href=https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2>Reproducible Research Course site</a>, downloaded as a bz2 compressed file.

```r
# Check if data directory exists; create it if not
dataDirectory <- "./data"
if (!file.exists(dataDirectory)) dir.create(dataDirectory)

# download source file
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
dataFile <- paste(dataDirectory, "storm_data.bz2", sep='/')
download.file(fileUrl, destfile = dataFile)
```

## Reading in the data
We start by reading in the database.  We'll read it in straight from the compressed file.

```r
stormDataRaw <- read.table(dataFile,sep=",", header=TRUE)
```

Now that the data is read in, we'll take a glimpse at the dataset and look at the first few variables (there are 37) of the first few rows (there are 902,297 total) and  in the dataset.

```r
glimpse(stormDataRaw)
```

```
## Observations: 902,297
## Variables: 37
## $ STATE__    <dbl> 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, ...
## $ BGN_DATE   <fctr> 4/18/1950 0:00:00, 4/18/1950 0:00:00, 2/20/1951 0:...
## $ BGN_TIME   <fctr> 0130, 0145, 1600, 0900, 1500, 2000, 0100, 0900, 20...
## $ TIME_ZONE  <fctr> CST, CST, CST, CST, CST, CST, CST, CST, CST, CST, ...
## $ COUNTY     <dbl> 97, 3, 57, 89, 43, 77, 9, 123, 125, 57, 43, 9, 73, ...
## $ COUNTYNAME <fctr> MOBILE, BALDWIN, FAYETTE, MADISON, CULLMAN, LAUDER...
## $ STATE      <fctr> AL, AL, AL, AL, AL, AL, AL, AL, AL, AL, AL, AL, AL...
## $ EVTYPE     <fctr> TORNADO, TORNADO, TORNADO, TORNADO, TORNADO, TORNA...
## $ BGN_RANGE  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ BGN_AZI    <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ BGN_LOCATI <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ END_DATE   <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ END_TIME   <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ COUNTY_END <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ COUNTYENDN <lgl> NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA, NA,...
## $ END_RANGE  <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ END_AZI    <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ END_LOCATI <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ LENGTH     <dbl> 14.0, 2.0, 0.1, 0.0, 0.0, 1.5, 1.5, 0.0, 3.3, 2.3, ...
## $ WIDTH      <dbl> 100, 150, 123, 100, 150, 177, 33, 33, 100, 100, 400...
## $ F          <int> 3, 2, 2, 2, 2, 2, 2, 1, 3, 3, 1, 1, 3, 3, 3, 4, 1, ...
## $ MAG        <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ FATALITIES <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 4, 0, ...
## $ INJURIES   <dbl> 15, 0, 2, 2, 2, 6, 1, 0, 14, 0, 3, 3, 26, 12, 6, 50...
## $ PROPDMG    <dbl> 25.0, 2.5, 25.0, 2.5, 2.5, 2.5, 2.5, 2.5, 25.0, 25....
## $ PROPDMGEXP <fctr> K, K, K, K, K, K, K, K, K, K, M, M, K, K, K, K, K,...
## $ CROPDMG    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ CROPDMGEXP <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ WFO        <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ STATEOFFIC <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ ZONENAMES  <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ LATITUDE   <dbl> 3040, 3042, 3340, 3458, 3412, 3450, 3405, 3255, 333...
## $ LONGITUDE  <dbl> 8812, 8755, 8742, 8626, 8642, 8748, 8631, 8558, 874...
## $ LATITUDE_E <dbl> 3051, 0, 0, 0, 0, 0, 0, 0, 3336, 3337, 3402, 3404, ...
## $ LONGITUDE_ <dbl> 8806, 0, 0, 0, 0, 0, 0, 0, 8738, 8737, 8644, 8640, ...
## $ REMARKS    <fctr> , , , , , , , , , , , , , , , , , , , , , , , , 
## $ REFNUM     <dbl> 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, ...
```


```r
head(stormDataRaw)
```

```
##   STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
##    EVTYPE BGN_RANGE BGN_AZI BGN_LOCATI END_DATE END_TIME COUNTY_END
## 1 TORNADO         0                                               0
## 2 TORNADO         0                                               0
## 3 TORNADO         0                                               0
## 4 TORNADO         0                                               0
## 5 TORNADO         0                                               0
## 6 TORNADO         0                                               0
##   COUNTYENDN END_RANGE END_AZI END_LOCATI LENGTH WIDTH F MAG FATALITIES
## 1         NA         0                      14.0   100 3   0          0
## 2         NA         0                       2.0   150 2   0          0
## 3         NA         0                       0.1   123 2   0          0
## 4         NA         0                       0.0   100 2   0          0
## 5         NA         0                       0.0   150 2   0          0
## 6         NA         0                       1.5   177 2   0          0
##   INJURIES PROPDMG PROPDMGEXP CROPDMG CROPDMGEXP WFO STATEOFFIC ZONENAMES
## 1       15    25.0          K       0                                    
## 2        0     2.5          K       0                                    
## 3        2    25.0          K       0                                    
## 4        2     2.5          K       0                                    
## 5        2     2.5          K       0                                    
## 6        6     2.5          K       0                                    
##   LATITUDE LONGITUDE LATITUDE_E LONGITUDE_ REMARKS REFNUM
## 1     3040      8812       3051       8806              1
## 2     3042      8755          0          0              2
## 3     3340      8742          0          0              3
## 4     3458      8626          0          0              4
## 5     3412      8642          0          0              5
## 6     3450      8748          0          0              6
```
