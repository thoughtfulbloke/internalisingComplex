---
title: "Traffic Safety Data"
author: "David Hood"
date: "11/21/2017"
output: 
  word_document: 
    keep_md: yes
---



## Traffic Data

This repository is for the R code for obtaining various public sources of data around traffic safety (mainly from a New Zealand perspective).

For the benefit of non-R users, I am making a .csv copy of the obtained data in the traffic_data folder.

### R packages used


```r
library(OECD)
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

and a few configuration things


```r
dataFolder= "traffic_data"
```


## OECD data

OECD data is obtained using the OECD package in R, and by doing a get_datasets() command, you can then look through for the IDs or relevvant data sets


```r
dataset_list <- get_datasets()
# search_dataset("accidents", data = dataset_list)
```

### ITF_ROAD_ACCIDENTS

csv obtained 2017-11-20

The OECD ITF_ROAD_ACCIDENTS dataset is road accident information supplied by the ITF


```r
dataset <- "ITF_ROAD_ACCIDENTS"
csvfile <- paste0(dataFolder,"/OECD_",dataset, ".csv")
if(!file.exists(csvfile)){
  dstruc <- get_data_structure(dataset)
  df <- get_dataset(dataset)
  df2 <- merge(df, dstruc$UNIT, by.x="UNIT", by.y="id")
  names(df2)[9] <- "UNIT_label"
  df3 <- merge(df2, dstruc$VARIABLE, by.x="VARIABLE", by.y="id")
  names(df3)[10] <- "VARIABLE_label"
  write.csv(df3, file=csvfile, row.names = FALSE)
}
```


### ITF_PASSENGER_TRANSPORT

csv obtained 2017-11-20

The OECD ITF_PASSENGER_TRANSPORT dataset is passenger transport supplied by the ITF


```r
dataset <- "ITF_PASSENGER_TRANSPORT"
csvfile <- paste0(dataFolder,"/OECD_",dataset, ".csv")
if(!file.exists(csvfile)){
  dstruc <- get_data_structure(dataset)
  df <- get_dataset(dataset)
  df2 <- merge(df, dstruc$UNIT, by.x="UNIT", by.y="id")
  names(df2)[9] <- "UNIT_label"
  df3 <- merge(df2, dstruc$VARIABLE, by.x="VARIABLE", by.y="id")
  names(df3)[10] <- "VARIABLE_label"
  write.csv(df3, file=csvfile, row.names = FALSE)
}
```


### ITF_INV-MTN_DATA

csv obtained 2017-11-20

The OECD ITF_INV-MTN_DATA dataset is transport infrastructure investment and maintenance spending supplied by the ITF


```r
dataset <- "ITF_INV-MTN_DATA"
csvfile <- paste0(dataFolder,"/OECD_",dataset, ".csv")
if(!file.exists(csvfile)){
  dstruc <- get_data_structure(dataset)
  df <- get_dataset(dataset)
  df2 <- merge(df, dstruc$UNIT, by.x="UNIT", by.y="id")
  names(df2)[9] <- "UNIT_label"
  df3 <- merge(df2, dstruc$VARIABLE, by.x="VARIABLE", by.y="id")
  names(df3)[10] <- "VARIABLE_label"
  write.csv(df3, file=csvfile, row.names = FALSE)
}
```

### IRTAD_CASUAL_BY_AGE

csv obtained 2017-11-21

The OECD IRTAD_CASUAL_BY_AGE dataset is Casualties by age & road user, not as up to date or as many countries as the headline ITF_ROAD_ACCIDENTS data, but more detail where available. As this is a huge dataset from all of the detail, this query was specifically for deaths 2005+


```r
dataset <- "IRTAD_CASUAL_BY_AGE"
csvfile <- paste0(dataFolder,"/OECD_",dataset, ".csv")
if(!file.exists(csvfile)){
  dstruc <- get_data_structure(dataset)
  df <- get_dataset(dataset, filter="AUS+AUT+BEL+CAN+CZE+DNK+FIN+FRA+DEU+GRC+HUN+ISL+IRL+ISR+ITA+JPN+KOR+LTU+LUX+NLD+NZL+NOR+POL+PRT+SVN+ESP+SWE+CHE+GBR+GB-IRTAD+GBR-NIR+USA+ARG+CHL.740.110+120+130+140+150+190+230+240+250+260+270+280+290+310+370.1010+1050+1030+1040+1060+1100+1110+1120+1140.NUMBER.CORRECTED", start_time = 2005, end_time = 2016, pre_formatted = TRUE)
  df2 <- merge(df, dstruc$INJURY_TYPE, by.x="INJURY_TYPE", by.y="id")
  names(df2)[11] <- "INJURY_TYPE_label"
  df3 <- merge(df2, dstruc$AGE_GROUP, by.x="AGE_GROUP", by.y="id")
  names(df3)[12] <- "AGE_GROUP_label"
  df4 <- merge(df3, dstruc$TRAFFIC_PARTICIPATION, by.x="TRAFFIC_PARTICIPATION", by.y="id")
  names(df4)[13] <- "TRAFFIC_PARTICIPATION_label"
  write.csv(df4, file=csvfile, row.names = FALSE)
}
```

## New Zealand Ministry of Health

### Hospital Discharge records

csv obtained 2017-11-21

The New Zealand Ministry of Health publishes hospital discharge data for cases where someone has been admitted to public hospitals for a day or more at http://www.health.govt.nz/nz-health-statistics/health-statistics-and-data-sets/hospital-event-data-and-stats coded by cause. In the case of transport the external causes use the codes V00 to V99 depending on admitted persons mode of transport and accident cause. As the data is on different pages for each year, this script downloads and combines it into one dataset. For the two most recent years raw data is published as zip compressed text files. As it stands, the script discards the non-V series (traffic) data.


```r
dataset <- "Hospital_Discharges"
csvfile <- paste0(dataFolder,"/NZ_MoH_",dataset, ".csv")
if(!file.exists(csvfile)){
y1415zip <- "http://www.health.govt.nz/system/files/documents/publications/publicly-funded-discharges-2014-15-data.zip"
y1314zip <- "http://www.health.govt.nz/system/files/documents/publications/publicly-funded-discharges-2013-14-data.zip"
temp <- tempfile()
download.file(y1415zip,temp)
y1415data <- read.csv(unz(temp, "PubFund_DischargesInjury.txt"), sep="|", stringsAsFactors = FALSE)
unlink(temp)
temp <- tempfile()
download.file(y1314zip,temp)
y1314data <- read.csv(unz(temp, "PubFund_Data/PubFund_DischargesInjury.txt"), sep="|", stringsAsFactors = FALSE)
unlink(temp)
names(y1314data) <- names(y1415data)
discharges <- bind_rows(y1314data, y1415data)
traffic_discharges <- discharges[grep("V", discharges$ICDCode),]
write.csv(traffic_discharges, file=csvfile, row.names = FALSE)
}
```

I think there is a lot of interesting stuff in there




