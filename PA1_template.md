---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: true
---


## Loading and preprocessing the data


```r
library(chron)
library(dplyr, quietly = TRUE, warn.conflicts = FALSE)
library(lattice)
if(!file.exists('activity.csv')) {
  unzip('activity.zip')
}
activity <- read.csv('activity.csv')
```

## What is mean total number of steps taken per day?


```r
steps_per_date <- aggregate(steps ~ date, data=activity, sum, na.rm=TRUE)
```


```r
hist(steps_per_date$steps, 
     main = "Histogram of steps/day",
     xlab = "Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->


```r
mean_steps_daily <- mean(steps_per_date$steps)
median_steps_daily <- median(steps_per_date$steps)
print(paste("Mean steps per day: ", mean_steps_daily))
```

```
## [1] "Mean steps per day:  10766.1886792453"
```

```r
print(paste("Median steps per day: ", median_steps_daily))
```

```
## [1] "Median steps per day:  10765"
```

## What is the average daily activity pattern?


```r
activity_pattern_agg <- aggregate(steps ~ interval, 
                                  data = activity,
                                  mean,
                                  na.rm = TRUE)
plot(activity_pattern_agg$interval, 
     activity_pattern_agg$steps, 
     type='l',
     main='Avg. steps per time interval',
     xlab='Interval',
     ylab='Avg. steps')
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->


```r
max_rows <- activity_pattern_agg[which.max(activity_pattern_agg$steps),]
print(paste("Interval with maximum average steps: ", max_rows$interval))
```

```
## [1] "Interval with maximum average steps:  835"
```

## Imputing missing values

The `date` and `interval` columns are complete, however the `steps` column is missing some values.


```r
# Confirming that we have no missing date or interval columns.
missing_dates <- sum(is.na(activity$date))
missing_intervals <- sum(is.na(activity$interval))

# We are missing some steps values though.
missing_steps <- sum(is.na(activity$steps))
print(paste("Number of missing step values: ", missing_steps))
```

```
## [1] "Number of missing step values:  2304"
```

To fill in the missing values, we will populate the `NA` values with the medians for that time interval.


```r
imputed_activity <- activity %>%
  mutate(steps=as.double(steps)) %>%
  inner_join(activity_pattern_agg, by="interval") %>%
  mutate(steps=coalesce(steps.x, steps.y)) %>%
  select(date, interval, steps)

steps_per_date_imputed <- aggregate(steps ~ date, 
                                    data = imputed_activity, 
                                    sum)
hist(steps_per_date_imputed$steps,
     main = "Histogram of steps/day", 
     xlab = "Number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

Clearly, imputing the missing values alters the frequency of the daily step counts.  Of note is the fact that the distribution remains roughly the same.


```r
mean_steps_daily_imputed <- mean(steps_per_date_imputed$steps)
median_steps_daily_imputed <- median(steps_per_date_imputed$steps)
print(paste("Mean steps per day (imputed): ", mean_steps_daily_imputed))
```

```
## [1] "Mean steps per day (imputed):  10766.1886792453"
```

```r
print(paste("Median steps per day (imputed): ", median_steps_daily_imputed))
```

```
## [1] "Median steps per day (imputed):  10766.1886792453"
```

Imputing the missing value with this method does not cause any significant changes to the daily mean/median values.

## Are there differences in activity patterns between weekdays and weekends?


```r
weekday_aggregations <- imputed_activity %>%
  mutate(date = as.Date(date), weekend = is.weekend(date)) %>%
  mutate(weekend = ifelse(is.weekend(date), "weekend", "weekday"))

xyplot(steps ~ interval | weekend, data=weekday_aggregations, aspect=1/2, type="l")  
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png)<!-- -->
