Reproducible Research: Peer Assessment 1"
=========================================

## Loading and preprocessing the data

Show any code that is needed to

1. Load the data (i.e. read.csv())

2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
library(ggplot2)
actdata <- read.csv("activity.csv")
actdata[, c(1,3)] <- sapply(actdata[, c(1,3)], as.numeric)
```

## What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day


```r
stepsbyday <- aggregate(cbind(steps) ~ date, data=actdata, sum)
```

2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day


```r
hist(stepsbyday$steps, breaks=20)
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

3. Calculate and report the mean and median of the total number of steps taken per day


```r
mean(stepsbyday$steps)
```

```
## [1] 10766.19
```

```r
median(stepsbyday$steps)
```

```
## [1] 10765
```

## What is the average daily activity pattern?

1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
timeseries <- aggregate(cbind(steps) ~ interval, data=actdata, mean)
plot(timeseries, type = "l")
```

![plot of chunk unnamed-chunk-5](figure/unnamed-chunk-5-1.png) 

2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
timeseries[which.max(timeseries$steps),1]
```

```
## [1] 835
```

## Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)


```r
sum(is.na(actdata$steps))
```

```
## [1] 2304
```

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
cleandata <- actdata
cleandata <- merge(cleandata,timeseries, by.x=c("interval"), by.y=c("interval"),
        all.x=TRUE)
cleandata$steps.x[is.na(cleandata$steps.x)] <- cleandata$steps.y[is.na(cleandata$steps.x)]

cleanstepsbyday <- aggregate(cbind(steps.x) ~ date, data=cleandata, sum)
```

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
hist(cleanstepsbyday$steps.x, breaks=20)
```

![plot of chunk unnamed-chunk-9](figure/unnamed-chunk-9-1.png) 

```r
mean(cleanstepsbyday$steps.x)
```

```
## [1] 10766.19
```

```r
median(cleanstepsbyday$steps.x)
```

```
## [1] 10766.19
```

The mean does not change, as expected, because I filled in the missing values with the daily mean.

The median changes slightly, also as expected.

Consider this dataset:

1, 2, 3, 10, NA

The mean is 4, the median is 2.5

If I replace the NA with the mean I have:

1, 2, 3, 4, 10.

The new mean still 4. The new mwdian is 3.
 
## Are there differences in activity patterns between weekdays and weekends?

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.


```r
cleandata$date <- as.Date(cleandata$date)

weekdays1 <- c('Monday', 'Tuesday', 'Wednesday', 'Thursday', 'Friday')

cleandata$wDay <- factor((weekdays(cleandata$date) %in% weekdays1), 
        levels=c(FALSE, TRUE), labels=c('weekend', 'weekday'))
```

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
ggplot(cleandata, aes(x = interval, y = steps.x)) +
     facet_grid(wDay ~., scales = "free") +
     geom_line() +
     ylab("Steps") +
     ggtitle("Steps by Day Type")
```

![plot of chunk unnamed-chunk-11](figure/unnamed-chunk-11-1.png) 
