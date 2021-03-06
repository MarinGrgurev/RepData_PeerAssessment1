# Reproducible Research: Peer Assessment 1 - Personal Activity Monitoring
Marin Grgurev  
January 17, 2015  






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
For handling data in this assignment __data.table__ package was used. This package was used because of its SQL-ish approach to handling regular dataframes, speed and ease of use. It is in no way required to correctly finish this assignment. Its sole purpose here is to practice how to use it as author of this report is in the process of learning it :). Thus regarding this assignment a regular `data.frame` functions could have been used. 
The data have been read with `fread()` function:


```r
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

To compactly display internal strucuture of the data (i.e. data structure, variable naming scheme, classes etc.) `str()` function was used:


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

As evident from the result, the _date_ column is of `character` class. To correctly plot steps across time, _date_ column needs to be converted from character representation to `Date` class:


```r
data[, date := as.Date(date, "%Y-%m-%d")]
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
As stated in the assignment, missing values can be ignored so for this part no special subsetting actions to remove NA from data set will be performed in order to plot histogram and calculate total, mean and median values of steps taken per day.

To create histogram of the total number of steps taken each day total number of steps was summed and then aggregated by date: 


```r
data[, .(totalStepsDay = sum(steps)), by = date][1:5]  # showing only first 5 values
```

```
##          date totalStepsDay
## 1: 2012-10-01            NA
## 2: 2012-10-02           126
## 3: 2012-10-03         11352
## 4: 2012-10-04         12116
## 5: 2012-10-05         13294
```

Histogram shows distribution of total number of steps taken each day:


```r
ggplot(data[, .(totalStepsDay = sum(steps)), by = date], aes(x = totalStepsDay)) +
        geom_histogram(breaks = seq(0, 22500, 2500), colour = "black", fill = "gray") +
        labs(x = "Total number of steps per day", y = "Frequency") +
        ggtitle("Frequency of total number of steps taken each day") +
        theme_bw(10) +
        theme(text = element_text(colour = "grey25"), 
              plot.title = element_text(face = "bold", vjust = 1.5))
```

<img src="figures/HistogramMeanTotalStepsDay-1.png" title="" alt="" style="display: block; margin: auto;" />

Mean and median were calculated by applying `mean()` and `median()` functions to the total number of steps taken per day:


```r
data[, .(totalStepsDay = sum(steps)), by = date][, .(mean = mean(totalStepsDay, na.rm = TRUE), median = median(totalStepsDay, na.rm = TRUE))]
```

```
##        mean median
## 1: 10766.19  10765
```

As evident from the above result the mean total number of steps taken per day is 10766.19. The median total number of steps taken per day is 10765.

### What is the average daily activity pattern?
To create a time series plot of the 5-minute interval and the average number of steps taken averaged across all days, average number of steps across all days have been calculated and then grouped by interval to get average number of steps: 


```r
ggplot(data[, .(total = mean(steps, na.rm = TRUE)), by = interval], aes(x = interval, y = total)) +
        geom_line() +
        geom_vline(xintercept = data[, .(total = mean(steps, na.rm = TRUE)), by = interval][max(total) == total, interval], colour = "green") +
        scale_x_continuous(breaks = c(0, 500, 835, 1000, 1500, 2000)) +
        labs(x = "5-minute interval", y = "Average number of steps taken") +
        ggtitle("Average number of steps taken per each 5-minute interval") +
        theme_bw(10) +
        theme(text = element_text(colour = "grey25"), 
              plot.title = element_text(face = "bold", vjust = 1.5))
```

<img src="figures/DailyActivityPattern-1.png" title="" alt="" style="display: block; margin: auto;" />

On average, across all the days in the dataset interval 835 contains maximum number of steps (206.1698).

## Imputing missing values
First, quick summary for missing values across each column in dataset is given:


```r
data[, lapply(data, function(x) {sum(is.na(x))})]
```

```
##    steps date interval
## 1:  2304    0        0
```

There is 2304 rows that have missing values in _steps_ column while other two columns don't have missing values.

For imputation strategy replacing missing data with mean number of steps for that interval across all days was used. To achieve that the position of each NA was found and replaced with the mean number of steps from that particular interval:


```r
setkey(data, interval)
data[which(is.na(steps)), steps := as.integer(data[which(is.na(steps))][data[, .(total = round(mean(steps, na.rm = TRUE))), by = interval]][, total])]
```

Another check for missing values in _steps_ column:


```r
sum(is.na(data[, steps]))
```

```
## [1] 0
```

Histogram now shows distribution of total number of steps taken each day but with imputed missing values:


```r
ggplot(data[, .(total = sum(steps)), by = date], aes(x = total)) +
        geom_histogram(breaks = seq(0, 22500, 2500), colour = "black", fill = "gray") +
        labs(x = "Total number of steps per day", y = "Frequency") +
        ggtitle("Frequency of total number of steps taken each day (Imputed NAs)") +
        theme_bw(10) +
        theme(text = element_text(colour = "grey25"), 
              plot.title = element_text(face = "bold", vjust = 2))
```

<img src="figures/HistogramMeanTotalStepsDayImputed-1.png" title="" alt="" style="display: block; margin: auto;" />

Mean and median values were calculated same as above by applying `mean()` and `median()` functions but now to the data set with imputed missing values):


```r
data[, .(totalStepsDay = sum(steps)), by = date][, .(mean = mean(totalStepsDay, na.rm = TRUE), median(totalStepsDay, na.rm = TRUE))]
```

```
##        mean    V2
## 1: 10765.64 10762
```

Again as above plot is showing the mean total number of steps taken per day is 10765.64, while the median total number of steps taken per day is 10762.

As clear from the results the imputation did not change the values of mean and median considerably, although on the histogram it's clear that dataset with imputed missing values have higher number of steps for all the bins.

## Are there differences in activity patterns between weekdays and weekends?
To explore if there is any difference in activity patterns between weekdays and weekends a panel plot was created. First, additional column is added to the dataset by calling `sapply()` function on _date_ column to check if the date is in the list of working day names. Based on the result weekday or weekend factor value was assigned in new column named *weekEndDay*:


```r
data[, weekEndDay := as.factor(sapply(data[, date], function(x) {ifelse(weekdays(x) %in% weekdays(Sys.Date()+0:4), "weekday", "weekend")}))]
```

Then, panel plot containing a time series plot of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days is created. Because the example plot described in assignment is not perfect to see the differences in activity pattern, two plots were created. First plot compares two activity patterns placed on same plotting area representing weekdays and weekend activity patterns:


```r
ggplot(data[, .(total = mean(steps)), by = "interval,weekEndDay"], aes(x = interval, y = total, group = weekEndDay)) +
        geom_line(aes(color = weekEndDay)) +
        labs(x = "5-minute interval", y = "Average number of steps taken") +
        ggtitle("Comparison of activity patterns between weekdays and weekends") +
        theme_bw(10) +
        theme(text = element_text(colour = "grey25"), 
              plot.title = element_text(face = "bold", vjust = 1.5),
              legend.title = element_blank(), 
              legend.position=c(0.99, 0.70), 
              legend.justification = c(1, 0),
              legend.background = element_rect(fill = "transparent"))
```

<img src="figures/WeekEndDayCompare1-1.png" title="" alt="" style="display: block; margin: auto;" />

The second plot represent same comparison of activity pattern but on two different plotting areas (i.e. panel plot): 


```r
ggplot(data[, .(total = mean(steps)), by = "interval,weekEndDay"], aes(x = interval, y = total, group = weekEndDay)) +
        geom_line(aes(color = weekEndDay)) +
        facet_grid(.~weekEndDay) +
        labs(x = "5-minute interval", y = "Average number of steps taken") +
        ggtitle("Comparison of activity patterns between weekdays and weekends") +
        theme_bw(10) +
        theme(text = element_text(colour = "grey25"), 
              plot.title = element_text(face = "bold", vjust = 1.5),
              legend.position="none")
```

<img src="figures/WeekEndDayCompare2-1.png" title="" alt="" style="display: block; margin: auto;" />

By looking at created plots it is evident that the difference in activity patterns between weekdays and weekends really exists which is not surprising given the fact that humans tend to use weekends for various recreational activities during the course of a day and they are definitely not rushing to the work in the morning. Also very interesting to note is steep curve climb around 5.15 - 5.45 in the weekday mornings - phenomena which is probably influenced by alarm clocks :)
