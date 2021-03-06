---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---
## Loading and preprocessing the data

Loading the data

```r
data <- read.csv("activity.csv")
```

## What is mean total number of steps taken per day?
First, group the number of steps taken by date.

```r
total.steps <- tapply(data$steps, data$date, sum, na.rm = TRUE, simplify = TRUE)
```
Create histogram of the total number of steps taken each day

```r
hist(total.steps, xlab = "number of steps", main = "Histogram of the total number of steps taken each day")
```

![plot of chunk unnamed-chunk-3](figure/unnamed-chunk-3-1.png) 

Calculate the mean and median total number of steps taken per day.

```r
mean <- mean(total.steps)
print(mean)
```

```
## [1] 9354.23
```

```r
median <- median(total.steps)
print(median)
```

```
## [1] 10395
```
The mean and median total number of steps taken per day are 9354.2295082 and 10395

## What is the average daily activity pattern?

Calculate the average number of of steps taken in each 5-minute interval across all days.

```r
data$interval <- as.factor(data$interval)
step_avg <- tapply(data$steps, data$interval, sum, na.rm = TRUE, 
    simplify = TRUE)/length(levels(data$date))
```
Make a plot(time series) of the average number of steps taken.

```r
plot(x = levels(data$interval), y = step_avg, type = "l", xlab = "time", 
    ylab = "number of steps", main = "Average steps in 5-minute interval across all days")
```

![plot of chunk unnamed-chunk-7](figure/unnamed-chunk-7-1.png) 

Finding 5-minute interval containing maximum number of steps on average across all the days.

```r
max(step_avg)
```

```
## [1] 179.1311
```

```r
names(which.max(step_avg))
```

```
## [1] "835"
```

The maximum number of steps on average across all the days is 179.1311475 in the interval number 835.

## Imputing missing values
Calculate the total number of missing values in the dataset.

```r
na_num <- sum(!complete.cases(activity$steps))
```
The total number of missing values in the dataset is 0.

Use the mean/median of a 5-minute interval that has NAs to fill in all of the missing values in that interval. 


```r
data1 <- activity
for (i in 1:length(activity$steps)) {
    if (is.na(activity$steps[i])) {
        for (j in 1:length(step_average)) {
            if (as.character(activity$interval[i]) == names(step_average[j])) 
                activity$steps[i] = step_average[j]
        }
    }
}
data2 <- activity
clean_step_sum <- tapply(data2$steps, data2$date, sum, na.rm = TRUE, 
    simplify = TRUE)
```

Creating histogram for clean data with NAs imputed.

```r
hist(clean_step_sum, xlab = "number of steps", main = "Histogram of the total number of steps taken each day")
```

![plot of chunk unnamed-chunk-12](figure/unnamed-chunk-12-1.png) 

Calculate the mean and median total number of steps taken per day for the clean data after imputing NAs.


```r
mean(clean_step_sum)
```

```
## [1] 10581.01
```

```r
median(clean_step_sum)
```

```
## [1] 10395
```


The mean and median total number of steps taken per day are 1.0581014 &times; 10<sup>4</sup> and 1.0395 &times; 10<sup>4</sup>. Compared with the estimates from the first part of the assignment, the difference of mean and median value are -1226.7841978 and 0. We find that the shape of two histograms change a little, especially for the number of steps below 10000. The median value have no difference from the estimates from the first part of the assignment. However, the mean value increases significantly.

## Are there differences in activity patterns between weekdays and weekends?

Creating a new factor variable in the dataset with levels <U+0093>weekday<U+0094> and <U+0093>weekend<U+0094> indicating whether a given date is a weekday or weekend day.


```r
weekday <- weekdays(as.Date(data1$date, "%Y-%m-%d"))
for (i in 1:length(weekday)) {
    if ((weekday[i] == "Saturday") | (weekday[i] == "Sunday")) 
        weekday[i] = "weekend" else weekday[i] = "weekday"
}
data1$weekday <- as.factor(weekday)
```
Using the lattice package to create a panel plot containing a time series plot of the average number of of steps taken in each 5-minute interval across all weekday days or weekend days.


```r
data12 <- split(data1, data1$weekday)
weekday <- data12$weekday
weekend <- data12$weekend
step_average_weekday <- tapply(weekday$steps, weekday$interval, sum, simplify = TRUE)/(length(weekday$weekday)/288)
step_average_weekend <- tapply(weekend$steps, weekend$interval, sum, simplify = TRUE)/(length(weekend$weekday)/288)
output <- data.frame(steps = c(step_average_weekday, step_average_weekend), 
    interval = c(levels(activity$interval), levels(activity$interval)), weekday = as.factor(c(rep("weekday", 
        length(step_average_weekday)), rep("weekend", length(step_average_weekend)))))
library(lattice)
xyplot(steps ~ interval | weekday, data = output, layout = c(1, 2), ylab = "number of steps", 
    main = "Average number of steps for all weekday days or weekend days")
```

![plot of chunk unnamed-chunk-17](figure/unnamed-chunk-17-1.png) 
