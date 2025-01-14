---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document: 
    keep_md: yes
---
## Analysis of Step Data from October and November 2012

## Loading and preprocessing the data


```r
## load data
activity_data <- read.csv("activity.csv", na = "NA")
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

The data was provided in a csv formatted file.  It contained three columns of data.

1.  The steps that were taken during the specified interval
2.  The date of the interval
3.  The time of the start of the five minute interval during which the measurment was made.  The last two digits in this number is the minute within the hour (from 0 to 55) of the interval start and the first one or two digits is the hour in the day(from 0 to 24). 

## What is mean total number of steps taken per day?

###For this part of the assignment, you can ignore the missing values in the dataset.

### Make a histogram of the total number of steps taken each day

Note - the NAs are all associated with specific days and for those days, all of the measurements were NA.  So calcualting the total per day means that those days will be ignored.


```r
# Calculate, provide histogram and calculate mean and median of total number of steps taken each day 
# note that the NA's are all assoicated with certain days and for those days all intervals are NA
date_data <- tapply(activity_data$steps, activity_data$date, sum)
hist(date_data[!is.na(date_data)], main = "Histogram of Total Steps per Day (ignoring NAs)", xlab = "Total Steps per Day")
```

![](PA1_template_files/figure-html/histogram_ignore_nas-1.png)<!-- -->

### Calculate and report the mean and median total number of steps taken per day



```r
mean_by_date_ignore_na <- mean(date_data, na.rm = TRUE)
median_by_date_ignore_na <- median(date_data, na.rm = TRUE)
```

The mean of the total steps taken per day is 1.0766189\times 10^{4} and the median is 10765.

## What is the average daily activity pattern?

### Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

Since each interval has some NAs, I have removed the NAs for this part of the analysis (i.e., to ignore them)


```r
# Remove NAs from data set to allow calculation of interval averages and calculate interval averages
ignore_na_data <- activity_data[!is.na(activity_data$steps),]
interval_data <- tapply(ignore_na_data$steps, ignore_na_data$interval, mean)

# Plot the average steps per interval and identify the maximum interval
plot(interval_data, type ="l", main = "Average Number of Steps Taken per 5 Minute Interval in Day", ylab = "Average Number of Steps per Interval", xlab = "Interval")
```

![](PA1_template_files/figure-html/by_interval_line_plot-1.png)<!-- -->

###Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
start_max_interval <- as.integer(names(which.max(interval_data)))
start_max_interval_hour <- floor(start_max_interval/100)
start_max_interval_minute <- start_max_interval - 100*start_max_interval_hour
max_interval_number <- which.max(interval_data)
max_interval <- max_interval_number[[1]]
```

The 104th five minute interval of the day, starting at 8:35 contains the maximum average number of steps. 

## Imputing missing values

###Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)



```r
# calculate number of NA's, and then 
# impute the missing values using the average for the interval (from the previously calculated interval_data)
missing <- is.na(activity_data$steps)
number_missing <- sum(missing)
```

There are `r number_missing' missing values in the data set.

###Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

###Create a new dataset that is equal to the original dataset but with the missing data filled in.

I've decided to use the average by interval across all days (for which there are data) to impute values for the NAs.  Average for the day won't work since the days for which we have NAs have all NAs and the average by interval has already been calculated.


```r
imputed_data <- activity_data
for (i in 1:length(imputed_data$date)) {
  if (is.na(imputed_data[i,1])) {
    imputed_data[i,1] <- interval_data[as.character(imputed_data[i,3])]
  }
}
```

###Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Similar to above but now using the imputed data set


```r
# calculate total per day on the imputed datat set and graph and calculate the mean and median for the reulsts 
date_imputed_data <- tapply(imputed_data$steps, imputed_data$date, sum)
hist(date_imputed_data, main = "Histogram of Total Steps per Day (imputing NAs)", xlab = "Total Steps per Day")
```

![](PA1_template_files/figure-html/histogram_impute_nas-1.png)<!-- -->

```r
mean_by_date_impute_na <- mean(date_imputed_data)
median_by_date_impute_na <- median(date_imputed_data)
```

The mean is 1.0766189\times 10^{4} and the median is 1.0766189\times 10^{4}.  So the median is slightly different while the mean is right on (which makes a little sense since the imputed days ended up being at the median).

## Are there differences in activity patterns between weekdays and weekends?

###Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

I added a column to the imputed data showing the day of the week and based on that, I subsetted the data to weekend and weekday days adding a column that specified what type and then recombine the data


```r
imputed_data2 <- mutate(imputed_data, day = weekdays(as.Date(as.character(date))))

weekend_data <- imputed_data2[imputed_data2$day == "Saturday" |imputed_data2$day == "Sunday" ,]
weekday_data <- imputed_data2[imputed_data2$day == "Monday" |imputed_data2$day == "Tuesday" | imputed_data2$day == "Wednesday" | imputed_data2$day == "Thursday"  | imputed_data2$day == "Friday" ,]
weekend_data2 <- mutate(weekend_data, wday = "Weekend")
weekday_data2 <- mutate(weekday_data, wday = "Weekday")


amended_data <- rbind(weekday_data2, weekend_data2)
```

###Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

I decided to use Lattice and used group_by to get the data into a tidy package for plotting.


```r
# Use group-_by tidy the data set for the plot function

amended_data <- transform(amended_data, wday = factor(wday))
amended_data <- transform(amended_data, interval = factor(interval))
b <- amended_data %>% group_by(interval, wday) %>% summarize(steps = mean(steps))
b <- transform(b, interval = as.numeric(interval))

# Use lattice for the panel plot

library(lattice)

xyplot(steps ~ interval | wday, data = b, layout = c(1,2), type = "l", ylab = "Number of Steps")
```

![](PA1_template_files/figure-html/weekend_weekday_comparison-1.png)<!-- -->

Looks like other than early morning (getting ready for work?; walking to/ from the train station?), this person is more active on the weekends.  Though he/she does seem to sleep in a bit on the weekend.
