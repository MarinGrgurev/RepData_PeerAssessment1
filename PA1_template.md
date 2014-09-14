# Reproducible Research: Peer Assessment 1 - Personal Activity Monitoring
Marin Grgurev  
September 14, 2014  
## Introduction
This report is generated as a part of the Assignment (Peer Assessment 1) for the Coursera course Reproducible Research. Complete code of this report as well as figures and rest of the material is available on [GitHub](https://github.com/MarinGrgurev/RepData_PeerAssessment1).

In accordance with the Coursera Honor Code, I certify that this report and code written  here are my own work, and that I have appropriately acknowledged all external sources (if any) that were used in this work.

## Data
An activity tracker is a device or application for monitoring and tracking fitness-related metrics such as distance walked or run, calorie consumption, and in some cases heartbeat and quality of sleep. The term is now primarily used for dedicated electronic monitoring devices that are synced, in many cases wirelessly, to a computer or smartphone for long-term data tracking, an example of wearable technology (https://en.wikipedia.org/wiki/Activity_tracker). 

These type of devices are part of the "quantified self" movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. But these data remain under-utilized both because the raw data are hard to obtain and there is a lack of statistical methods and software for processing and interpreting the data.

This assignment makes use of data from a personal activity monitoring device. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

The data for this assignment can be downloaded from the course web site:

* Dataset: [Activity monitoring data](https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip) [52K]

The variables included in this dataset are:

* steps - Number of steps taking in a 5-minute interval (missing values are coded as NA)

* date - The date on which the measurement was taken in YYYY-MM-DD format

* interval - Identifier for the 5-minute interval in which measurement was taken

The dataset is stored in a comma-separated-value (CSV) file and there are a total of 17,568 observations in this dataset.

## Assignment
### Loading and preprocessing the data
For handling data in this assignment `data.table` package was used. The data have been read with `fread()` function and assigned to `data` object:


```r
library(data.table)
library(ggplot2)
data <- fread("data/activity.csv")
data
```

```
##        steps       date interval
##     1:    NA 2012-10-01        0
##     2:    NA 2012-10-01        5
##     3:    NA 2012-10-01       10
##     4:    NA 2012-10-01       15
##     5:    NA 2012-10-01       20
##    ---                          
## 17564:    NA 2012-11-30     2335
## 17565:    NA 2012-11-30     2340
## 17566:    NA 2012-11-30     2345
## 17567:    NA 2012-11-30     2350
## 17568:    NA 2012-11-30     2355
```

To compactly display internal strucuture of a `data` object (i.e. data structure, variable naming scheme, classes etc.) `str()` function is used:


```r
str(data)
```

```
## Classes 'data.table' and 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : chr  "2012-10-01" "2012-10-01" "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

As evident from the `str()` result, the _date_ column is of `"character"` class and _interval_ is of class `"integer"`. To correctly plot steps across time, _date_ column needed to be converted from character representation to class `"Date"`:


```r
data[,date := as.Date(date, "%Y-%m-%d")]
```

Last check-out for internal structure to confirm that dataset is ready for analysis:


```r
str(data)
```

```
## Classes 'data.table' and 'data.frame':	17568 obs. of  3 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: int  0 5 10 15 20 25 30 35 40 45 ...
##  - attr(*, ".internal.selfref")=<externalptr>
```

### What is mean total number of steps taken per day?
As stated in the assignment missing values can be ignored for this part and thus rows with missing value are excluded from calculations. First, one quick summary for missing values across each column in dataset:


```r
data[,lapply(data, function(x) {sum(is.na(x))})]
```

```
##    steps date interval
## 1:  2304    0        0
```

There is 2304 rows that have missing values in _steps_ column while other two columns don't have missing values. With function `complete.cases()` missing values are excluded from calculations:


```r
data.NA.omit <- data[complete.cases(data),]
data.NA.omit
```

```
##        steps       date interval
##     1:     0 2012-10-02        0
##     2:     0 2012-10-02        5
##     3:     0 2012-10-02       10
##     4:     0 2012-10-02       15
##     5:     0 2012-10-02       20
##    ---                          
## 15260:     0 2012-11-29     2335
## 15261:     0 2012-11-29     2340
## 15262:     0 2012-11-29     2345
## 15263:     0 2012-11-29     2350
## 15264:     0 2012-11-29     2355
```

To calculate total number of steps taken per day, first total number of steps was calculated with `sum()` and then aggregated by _date_: 


```r
data.NA.omit[,list(total=sum(steps)), by="date"][1:6]
```

```
##          date total
## 1: 2012-10-02   126
## 2: 2012-10-03 11352
## 3: 2012-10-04 12116
## 4: 2012-10-05 13294
## 5: 2012-10-06 15420
## 6: 2012-10-07 11015
```

Histogram shows us distribution of total number of steps taken each day:


```r
ggplot(data.NA.omit[,list(total=sum(steps)), by="date"], aes(x=total))+
        geom_histogram(breaks=seq(0,22500,2500), colour="black", fill="gray")+
        labs(x="Total number of steps per day", y="Frequency")+
        ggtitle("Frequency of total number of steps taken each day")+
        theme_bw()
```

![plot of chunk HistogramMeanTotalStepsDay](./PA1_template_files/figure-html/HistogramMeanTotalStepsDay.png) 

To calculate mean and median were calculated by applying `mean()` and `median()` functions to the total number of steps taken per day (second column in `data.NA.omit` object:


```r
mean(data.NA.omit[,list(total=sum(steps)), by="date"][[2]])
```

```
## [1] 10766
```

```r
median(data.NA.omit[,list(total=sum(steps)), by="date"][[2]])
```

```
## [1] 10765
```

The mean total number of steps taken per day is 1.0766 &times; 10<sup>4</sup>.  
The median total number of steps taken per day is 10765.

Last thing to do for this part of the assignment is to save histogram figure in `figure/` folder:


```r
dev.copy(png,"./figures/histogram.png", width=480, height=480, bg="transparent")
dev.off()
```

### What is the average daily activity pattern?
To create a time series plot of the 5-minute interval and the average number of steps taken, averaged across all days dataset without missing values from previous part (`data.NA.omit`) was used. By not grouping by date average number of steps across all days have been calculated and then grouped by interval to get average number of steps per interval: 


```r
ggplot(data.NA.omit[,list(total=mean(steps)), by="interval"], aes(x=interval, y=total))+
        geom_line()+
        geom_vline(xintercept=data.NA.omit[,list(total=mean(steps)), by="interval"][total==max(total)][[1]], colour="green")+
        scale_x_continuous(breaks=c(0, 500, 835, 1000, 1500, 2000))+
        labs(x="5-minute interval", y="Average number of steps taken")+
        ggtitle("Average number of steps taken per each 5-minute interval")+
        theme_bw()
```

![plot of chunk DailyActivityPattern](./PA1_template_files/figure-html/DailyActivityPattern.png) 

On average, across all the days in the dataset 835 contains maximum number of steps (206.1698).

Saving the time-series plot figure in `figure/` folder:


```r
dev.copy(png,"./figures/AvgStepsInterval.png", width=480, height=480, bg="transparent")
dev.off()
```

## Imputing missing values



## Are there differences in activity patterns between weekdays and weekends?
