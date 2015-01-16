---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
# ASSIGNMENT 1 | COURSE: REPRODUCIBLE RESEARCH

This assignment makes use of **data from a personal activity monitoring device**. This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the **number of steps taken in 5 minute intervals each day**.

## Loading and preprocessing the data
This analysis can be openly accessed at [GitHub](https://github.com/MoritzoTheCoder/RepData_PeerAssessment1)

```r
    data <- read.csv("activity.csv", header=T)
    data$date <- as.Date(data$date)
```


## What is mean total number of steps taken per day?
The following code snippet first computes the sum of the steps for each day and than...
- calculates the *mean* over all days
- calculates the *median* over all days
- plots a *histogram* of the frequency distribution

```r
    sumup <- aggregate(x=data$steps, by=list(data$date), FUN=sum, simplify=T)
    mean(sumup$x, na.rm=T)
```

```
## [1] 10766.19
```

```r
    median(sumup$x, na.rm=T)
```

```
## [1] 10765
```

```r
    hist(sumup$x, breaks="FD", xlab="number of steps", main="Histogram of the number of steps per day (missings ignored)")
```

![plot of chunk average_number_of_steps](figure/average_number_of_steps-1.png) 


## What is the average daily activity pattern?
The following code snippet first computes the average steps recorded in each interval over all days and than plots those aggregated steps over the course of a day (i.e. intervals).

```r
    avg_each_interval <- aggregate(x=data$steps, by=list(data$interval), FUN=mean, na.rm=T, simplify=T)
    colnames(avg_each_interval) = c("interval", "steps")
    plot(x=avg_each_interval$interval, y=avg_each_interval$steps, type="l", xlab= "Interval", ylab= "Activity (steps over the course of daytime)", main= "Average daily activity pattern")
```

![plot of chunk activity_pattern](figure/activity_pattern-1.png) 

The following code snippet subsets the interval with on average highest activity (measured by steps recorded)

```r
    avg_each_interval[which.max(avg_each_interval[,2]),]
```

```
##     interval    steps
## 104      835 206.1698
```


## Imputing missing values
Missing values are replaced by the mean for the corresponding interval. Than the frequency distribution is analysed as in a previously section.
- calculate *mean* over all days
- calculate the *median* over all days
- plot a *histogram* of the frequency distribution

```r
    # function to replace missing values by mean
    impute.mean <- function(x) replace(x, is.na(x), mean(x, na.rm = TRUE))
    # Split data by interval
    library("plyr")
    data.2 <- ddply(data, ~interval, transform, steps = impute.mean(steps))
    sumup.2 <- aggregate(x=data.2$steps, by=list(data.2$date), FUN=sum, simplify=T)
    mean(sumup.2$x, na.rm=T)
```

```
## [1] 10766.19
```

```r
    median(sumup.2$x, na.rm=T)
```

```
## [1] 10766.19
```

```r
    hist(sumup.2$x, breaks="FD", xlab="number of steps", main="Histogram of the number of steps per day (missings replaced)")
```

![plot of chunk imputing missing values](figure/imputing missing values-1.png) 

The underlying question of this step is: Does the strategy to replace missings with the average for the corresponding measuring interval differ much from the output obtained from the analysis just ignoring the missings as shown in a previously section of this document?

## Are there differences in activity patterns between weekdays and weekends?

The following code splits the data for analysing the activity pattern separately for weekends and weekdays. It uses the data with mssing values replaced by the mean as processes in the previous code snipped.

```r
    #Create a variable indicating if weekday or weekend
    data.2$DayOfWeek <- weekdays(data.2$date, abbreviate=F)
    data.2$IsWeekend <- sapply(data.2$DayOfWeek, function(x) {if ((x=="Samstag")|(x=="Sonntag")) {"weekend"} else {"weekdays"}})
    #Split data frame in data frame for weekend and weekdays
    data.2.weekend <- data.2[data.2$IsWeekend=="weekend",]
    data.2.weekend <- aggregate(x=data.2.weekend$steps, by=list(data.2.weekend$interval), FUN=mean, na.rm=T, simplify=T)
    data.2.weekdays <- data.2[data.2$IsWeekend=="weekdays",]
    data.2.weekdays <- aggregate(x=data.2.weekdays$steps, by=list(data.2.weekdays$interval), FUN=mean, na.rm=T, simplify=T)
```
The next code snipped actually plots the activity patterns.

```r
#Plotting activity pattern for comparison between weekday and weekend
    par(mfrow = c(2, 1))
    plot( x=data.2.weekend$Group.1 , y=data.2.weekend$x, type="l", xlab="Interval", ylab="steps", main="weekend")
    plot( x=data.2.weekdays$Group.1 , y=data.2.weekdays$x, type="l", xlab="Interval", ylab="steps", main="weekdays")
```

![plot of chunk unnamed-chunk-2](figure/unnamed-chunk-2-1.png) 


