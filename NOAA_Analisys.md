# NOAA Storm Database Severe Weather Events Analysis
Ryan Rush  
April 29, 2017  

# Synopsis
The goal of this analysis is to explore the NOAA Storm Database to answer questions relating to severe weather events.  We are interested in answering the questions: 1) Across the United States, which types of events are most harmful with respect to population health? and 2) Across the United States, which types of events have the greatest economic consequences?

# Setup
This analysis requires the dplyr, stringdist and ggplot2 libraries:

```r
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(stringdist))
suppressPackageStartupMessages(library(ggplot2))
```

For purposes of clarity, we'll print information on the current environment.

```r
sessionInfo()
```

```
## R version 3.4.0 (2017-04-21)
## Platform: x86_64-w64-mingw32/x64 (64-bit)
## Running under: Windows 10 x64 (build 14393)
## 
## Matrix products: default
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
## [1] ggplot2_2.2.1      stringdist_0.9.4.4 dplyr_0.5.0       
## 
## loaded via a namespace (and not attached):
##  [1] Rcpp_0.12.10     knitr_1.15.1     magrittr_1.5     munsell_0.4.3   
##  [5] colorspace_1.3-2 R6_2.2.0         stringr_1.2.0    plyr_1.8.4      
##  [9] tools_3.4.0      parallel_3.4.0   grid_3.4.0       gtable_0.2.0    
## [13] DBI_0.6-1        htmltools_0.3.6  yaml_2.1.14      lazyeval_0.2.0  
## [17] assertthat_0.2.0 rprojroot_1.2    digest_0.6.12    tibble_1.3.0    
## [21] evaluate_0.10    rmarkdown_1.5    stringi_1.1.5    compiler_3.4.0  
## [25] scales_0.4.1     backports_1.0.5
```

# Data Processing
The data for this analysis comes from the U.S. National Oceanic and Atmospheric Administration's (NOAA) storm database. This database tracks characteristics of major storms and weather events in the United States, including when and where they occur, as well as estimates of any fatalities, injuries, and property damage.

### Obtaining the data
We obtained the database from the <a href="https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2">Reproducible Research Course site</a>, downloaded as a bz2 compressed file.

```r
# Check if data directory exists; create it if not
dataDirectory <- "./data"
if (!file.exists(dataDirectory)) dir.create(dataDirectory)

# download source file
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2FStormData.csv.bz2"
dataFile <- paste(dataDirectory, "storm_data.bz2", sep='/')
download.file(fileUrl, destfile = dataFile)
```

### Reading in the data
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
head(stormDataRaw[,1:10])
```

```
##   STATE__           BGN_DATE BGN_TIME TIME_ZONE COUNTY COUNTYNAME STATE
## 1       1  4/18/1950 0:00:00     0130       CST     97     MOBILE    AL
## 2       1  4/18/1950 0:00:00     0145       CST      3    BALDWIN    AL
## 3       1  2/20/1951 0:00:00     1600       CST     57    FAYETTE    AL
## 4       1   6/8/1951 0:00:00     0900       CST     89    MADISON    AL
## 5       1 11/15/1951 0:00:00     1500       CST     43    CULLMAN    AL
## 6       1 11/15/1951 0:00:00     2000       CST     77 LAUDERDALE    AL
##    EVTYPE BGN_RANGE BGN_AZI
## 1 TORNADO         0        
## 2 TORNADO         0        
## 3 TORNADO         0        
## 4 TORNADO         0        
## 5 TORNADO         0        
## 6 TORNADO         0
```

### Subset Dataset to what is needed for analysis

Since the dataset is large, we'll want to subset the data for processing to only that which we need.  First, there are only a few of the variables we need for analysis, so we'll create a copy of the dataset with just the needed variables.

```r
workingData <- stormDataRaw %>% select(EVTYPE, BGN_DATE, FATALITIES, INJURIES, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP)
glimpse(workingData)
```

```
## Observations: 902,297
## Variables: 8
## $ EVTYPE     <fctr> TORNADO, TORNADO, TORNADO, TORNADO, TORNADO, TORNA...
## $ BGN_DATE   <fctr> 4/18/1950 0:00:00, 4/18/1950 0:00:00, 2/20/1951 0:...
## $ FATALITIES <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1, 0, 0, 4, 0, ...
## $ INJURIES   <dbl> 15, 0, 2, 2, 2, 6, 1, 0, 14, 0, 3, 3, 26, 12, 6, 50...
## $ PROPDMG    <dbl> 25.0, 2.5, 25.0, 2.5, 2.5, 2.5, 2.5, 2.5, 25.0, 25....
## $ PROPDMGEXP <fctr> K, K, K, K, K, K, K, K, K, K, M, M, K, K, K, K, K,...
## $ CROPDMG    <dbl> 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, ...
## $ CROPDMGEXP <fctr> , , , , , , , , , , , , , , , , , , , , , , , ,
```

We know we want to measure the financial and health impacts, so we can subset the dataset to only those records which have impact on the financial or health variables.

```r
workingData <- workingData[(workingData$PROPDMG != 0 & workingData$CROPDMG != 0) | (workingData$FATALITIES != 0 & workingData$INJURIES != 0),]
dim(workingData)
```

```
## [1] 18781     8
```

Since we want to analyze impact by type of event, we should exclude data prior to 1996 since not all event types were tracked before that.

```r
# First convert the beginning date to POSIXct datetype to more easily filter with
workingData$BGN_DATE <- as.POSIXct(workingData$BGN_DATE, format="%m/%d/%Y %H:%M:%S")
# Then subset the data to just those after 1/1/1996
workingData <- workingData %>% filter(BGN_DATE >= "1996-01-01")
dim(workingData)
```

```
## [1] 14738     8
```

```r
summary(workingData$BGN_DATE)
```

```
##                  Min.               1st Qu.                Median 
## "1996-01-01 00:00:00" "1999-06-06 00:00:00" "2004-03-17 00:00:00" 
##                  Mean               3rd Qu.                  Max. 
## "2003-11-21 17:28:16" "2008-06-04 00:00:00" "2011-11-28 00:00:00"
```

### Tidying the Data for processing

Since we'll be relying heavily on the Event Type values (EVTYPE), we need to make sure we understand their values and that they are standard.  There are 48 official event types, but looking at the unique number of values in the dataset we see there are 76

First, let's reset the factor levels since we've subsetted the dataset

```r
workingData$EVTYPE <- factor(workingData$EVTYPE)
str(workingData$EVTYPE)
```

```
##  Factor w/ 76 levels "AVALANCHE","BLACK ICE",..: 74 64 26 26 26 26 16 64 64 64 ...
```

This is most likely due to mis-spellings of the official event types when the data was entered.  We can get a sense of that by getting the number unique event types after converting them all to upper case and removing leading and trailing spaces, which gives us 74.

```r
workingData$EVTYPE <- factor(toupper(trimws(workingData$EVTYPE)))
str(workingData$EVTYPE)
```

```
##  Factor w/ 74 levels "AVALANCHE","BLACK ICE",..: 72 62 25 25 25 25 16 62 62 62 ...
```

Another way to to isolate the event types is by getting rid of those which are statistically insignificant.  If we look at event types where there are at least 2 entries, we see there are only 51.

```r
tabledEventTypes <- table(workingData$EVTYPE)
evtypKeepers <- names(tabledEventTypes[tabledEventTypes >= 2])
length(evtypKeepers)
```

```
## [1] 51
```

Now we can subset the working dataset to only those records with one of the more common event types.

```r
workingData <- workingData[workingData$EVTYPE %in% evtypKeepers,]
workingData$EVTYPE <- factor(workingData$EVTYPE)
str(workingData$EVTYPE)
```

```
##  Factor w/ 51 levels "AVALANCHE","BLIZZARD",..: 49 40 17 17 17 17 11 40 40 40 ...
```

Now, we need to normalize the various event types to the standard types so we can get accurate analysis.  We first need to get the official event type names, which we'll do just by hard-coding them into a vector. (The list was pulled from the <a href="https://d396qusza40orc.cloudfront.net/repdata%2Fpeer2_doc%2Fpd01016005curr.pdf">Storm Data Documentation</a> sheet)

```r
officialEventTypes <- c('Astronomical Low Tide','Avalanche','Blizzard','Coastal Flood',
                        'Cold/Wind Chill','Debris Flow','Dense Fog','Dense Smoke','Drought',
                        'Dust Devil','Dust Storm','Excessive Heat','Extreme Cold/Wind Chill',
                        'Flash Flood','Flood','Frost/Freeze','Funnel Cloud',
                        'Freezing Fog','Hail','Heat','Heavy Rain','Heavy Snow','High Surf',
                        'High Wind','Hurricane (Typhoon)','Ice Storm','Lake-Effect Snow',
                        'Lakeshore Flood','Lightning','Marine Hail','Marine High Wind',
                        'Marine Strong Wind','Marine Thunderstorm Wind','Rip Current',
                        'Seiche','Sleet','Storm Surge/Tide','Strong Wind','Thunderstorm Wind',
                        'Tornado','Tropical Depression','Tropical Storm','Tsunami',
                        'Volcanic Ash','Waterspout','Wildfire','Winter Storm','Winter Weather'
                        )
officialEventTypes <- toupper(officialEventTypes)
length(officialEventTypes)
```

```
## [1] 48
```

We first use distince matching to match the majority of event types to the official descriptions.  We used a maximum distance of 3 which gave the best results.

```r
combinedEventTypes <- as.data.frame(evtypKeepers) %>%
    rename(EVTYPE = evtypKeepers) %>%
    mutate(officialEVTYPE = officialEventTypes[amatch(evtypKeepers, officialEventTypes, maxDist=3)])
table(IsMatched=!is.na(combinedEventTypes$officialEVTYPE))
```

```
## IsMatched
## FALSE  TRUE 
##    18    33
```

Now there are only 40 unmapped event types.  We will need to handle these manually.

```r
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('LANDSLIDE'), ]$officialEVTYPE <- 'AVALANCHE'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('DRY MICROBURST'), ]$officialEVTYPE <- 'DUST DEVIL'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('EXTREME COLD'), ]$officialEVTYPE <- 'EXTREME COLD/WIND CHILL'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('RIVER FLOOD','RIVER FLOODING','URBAN/SML STREAM FLD'), ]$officialEVTYPE <- 'FLOOD'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('FREEZING RAIN','FREEZING DRIZZLE'), ]$officialEVTYPE <- 'FREEZING FOG'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('HEAVY SURF/HIGH SURF'), ]$officialEVTYPE <- 'HIGH SURF'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('WIND'), ]$officialEVTYPE <- 'HIGH WIND'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('HURRICANE','TYPHOON'), ]$officialEVTYPE <- 'HURRICANE (TYPHOON)'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('ICY ROADS'), ]$officialEVTYPE <- 'ICE STORM'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('STORM SURGE','TIDAL FLOODING'), ]$officialEVTYPE <- 'STORM SURGE/TIDE'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('GUSTY WIND','GUSTY WINDS'), ]$officialEVTYPE <- 'STRONG WIND'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('TSTM WIND','TSTM WIND (G45)','TSTM WIND/HAIL'), ]$officialEVTYPE <- 'THUNDERSTORM WIND'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('WILD/FOREST FIRE'), ]$officialEVTYPE <- 'WILDFIRE'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('MIXED PRECIPITATION','WINTER WEATHER/MIX','WINTRY MIX'), ]$officialEVTYPE <- 'WINTER WEATHER'
combinedEventTypes[combinedEventTypes$EVTYPE %in% c('FOG'), ]$officialEVTYPE <- 'DENSE FOG'

table(IsMatched=!is.na(combinedEventTypes$officialEVTYPE))
```

```
## IsMatched
## TRUE 
##   51
```

Now that we have the event types mapped to the official event types, we'll update the working dataset with the official types.

```r
workingData <- workingData %>%
    left_join(combinedEventTypes) %>%
    mutate(EVTYPE2 = ifelse(is.na(officialEVTYPE),EVTYPE,officialEVTYPE)) %>%
    select(-EVTYPE,-officialEVTYPE) %>%
    rename(EVTYPE = EVTYPE2)
```

```
## Joining, by = "EVTYPE"
```

```r
workingData$EVTYPE <- factor(workingData$EVTYPE)
str(workingData$EVTYPE)
```

```
##  Factor w/ 33 levels "AVALANCHE","BLIZZARD",..: 32 29 14 14 14 14 10 29 29 29 ...
```


# Results

Now that we have a tidy dataset, we can begin to answer our analysis questions.

### Which types of events are most harmful to population health
We first want to look at which type of events are the most harmful to a population's health.  The Fatalities and Injuries variables are good indicators of the effect an event has had on population health, so we'll create a dataset which contains only observations that have values in either of those variables and contains the sum of the two variables.

```r
popHealthData <- workingData %>%
    filter(FATALITIES + INJURIES != 0) %>%
    select(EVTYPE, FATALITIES, INJURIES) %>%
 #   transmute(TOTEFFECT = FATALITIES + INJURIES) %>%
    group_by(EVTYPE) %>%
    summarise(TOTEFFECT = sum(FATALITIES + INJURIES))
popHealthData$EVTYPE <- factor(popHealthData$EVTYPE)
glimpse(popHealthData)
```

```
## Observations: 31
## Variables: 2
## $ EVTYPE    <fctr> AVALANCHE, BLIZZARD, COLD/WIND CHILL, DENSE FOG, DR...
## $ TOTEFFECT <dbl> 168, 347, 5, 406, 4, 110, 5048, 102, 947, 6683, 15, ...
```

Now that we have the population health dataset, we'll plot it showing the top 10 event types by total effect.


```r
ggplot(top_n(popHealthData,10,TOTEFFECT),
       aes(x=reorder(EVTYPE,TOTEFFECT,sum),y=TOTEFFECT)) +
    geom_col() +
    coord_flip() +
    labs(x="Event Type",
         y="Total Population Health Effect",
         title="Top Event Types on Population Health"
    )
```

![](NOAA_Analisys_files/figure-html/pophealthplot-1.png)<!-- -->

We see that Tornadoes by far have the most effect on population health, followed by Floods and Excessing Heat.  It trails off after that.

### Which types of events have the greatest economic consequences
Next we'll look at which type of events are the most harmful to economic resources.  The PROPDMG (Property Damage) and CROPDMG (Crop Damage) variables are good indicators of the effect an event has had on economic resources.  These values are stored with an associated exponential value, so we'll first create a dataset which holds the calculated damage values taking into account the exponent value.

```r
econHealthData <- workingData %>%
    filter(PROPDMG + CROPDMG != 0) %>%
    select(EVTYPE, PROPDMG, PROPDMGEXP, CROPDMG, CROPDMGEXP)
econHealthData$PROPDMG <- case_when(
                        econHealthData$PROPDMGEXP == "K" ~ econHealthData$PROPDMG * 1000,
                        econHealthData$PROPDMGEXP == "M" ~ econHealthData$PROPDMG * 1000000,
                        econHealthData$PROPDMGEXP == "B" ~ econHealthData$PROPDMG * 1000000000,
                        TRUE ~ econHealthData$PROPDMG
                    )
econHealthData$CROPDMG <- case_when(
                        econHealthData$CROPDMGEXP == "K" ~ econHealthData$CROPDMG * 1000,
                        econHealthData$CROPDMGEXP == "M" ~ econHealthData$CROPDMG * 1000000,
                        econHealthData$CROPDMGEXP == "B" ~ econHealthData$CROPDMG * 1000000000,
                        TRUE ~ econHealthData$CROPDMG
                    )
econHealthData <- econHealthData %>%
    group_by(EVTYPE) %>%
    summarise(TOTEFFECT = sum(PROPDMG + CROPDMG))
econHealthData$EVTYPE <- factor(econHealthData$EVTYPE)
glimpse(econHealthData)
```

```
## Observations: 29
## Variables: 2
## $ EVTYPE    <fctr> AVALANCHE, BLIZZARD, DENSE FOG, DROUGHT, DUST DEVIL...
## $ TOTEFFECT <dbl> 41815000, 157665000, 7492000, 1446482000, 123000, 24...
```


Now that we have the population health dataset, we'll plot it showing the top 10 event types by total effect.


```r
ggplot(top_n(econHealthData,10,TOTEFFECT),
       aes(x=reorder(EVTYPE,TOTEFFECT,sum),y=TOTEFFECT)) +
    geom_col() +
    coord_flip() +
    labs(x="Event Type",
         y="Total Economic Effect",
         title="Top Event Types on Economy"
    )
```

![](NOAA_Analisys_files/figure-html/econhealthplot-1.png)<!-- -->


We see that Floods by far have the most effect on the economy, followed by Hurricanes (Typhoons) and Tornadoes.  It trails off after that.



