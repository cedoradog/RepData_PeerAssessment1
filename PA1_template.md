# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Show any code that is needed to

1. Load the data (i.e. read.csv())

2. Process/transform the data (if necessary) into a format suitable for your analysis



```r
#setwd(".../RepData_PeerAssessment1")
#There is the option to download the data from the internet
#url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
#download.file(url, "steps.csv")
#But we prefer work with the data from the repo
unzip("activity.zip")
activity <- read.csv("activity.csv")
library(lubridate)
#activity$time <- (strptime(paste(date, as.character(interval)), "%Y-%m-%d %H%M"))
print(names(activity))
```

```
## [1] "steps"    "date"     "interval"
```

## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Make a histogram of the total number of steps taken each day

2. Calculate and report the mean and median total number of steps taken per day

```r
stepsPerDate <- aggregate(steps ~ date, activity, sum, na.rm=T)
#Calculate mean and median
meanStepsPerDay <- mean(stepsPerDate$steps)
medianStepsPerDay <- median(stepsPerDate$steps)
#Plot the histogram
hist(stepsPerDate$steps, breaks = 1000 * (0:22), main = "Frequency of steps per day", xlab = "Steps per day")
abline(v = meanStepsPerDay, col = "blue", lw = 2)
```

![](PA1_template_files/figure-html/histogram-1.png) 

```r
cat("The mean is", meanStepsPerDay, "steps.")
```

```
## The mean is 10766.19 steps.
```

```r
cat("The median is", medianStepsPerDay, "steps.")
```

```
## The median is 10765 steps.
```

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
stepsPerInterval <- aggregate(steps ~ interval, activity, mean, na.rm=T)
stepsPerInterval$newInterval <- as.character(sprintf("%04d", stepsPerInterval$interval))
stepsPerInterval$time <- strptime(stepsPerInterval$newInterval, "%H%M")
maxPerDay <- max(stepsPerInterval$steps, na.rm=TRUE)
maxTime <- stepsPerInterval$time[stepsPerInterval$steps == maxPerDay]
plot(stepsPerInterval$time, stepsPerInterval$steps, type="l", xlab="Hour", ylab="Mean number of steps", main="Activity during day")
```

![](PA1_template_files/figure-html/day-1.png) 

```r
cat(paste("The interval with more activity, in average, is ", maxTime$hour, ":", maxTime$min, ".", sep=""))
```

```
## The interval with more activity, in average, is 8:35.
```

## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
#Counting
activity$isNA <- is.na(activity$steps)
numberOfNAs <- table(activity$isNA)["TRUE"]

newActivity <- activity

#Imputation strategy
for(i in 1:nrow(newActivity)){
  if(newActivity$isNA[i]){
    meanPerInterval <- stepsPerInterval$steps[stepsPerInterval$interval == newActivity$interval[i]]
    #if(activity$date[i] %in% stepsPerDate$date){
     # meanPerDate <- stepsPerDate$steps[stepsPerDate$date == activity$date[i]])}
    #activity$steps[i] <- (meanPerDate + meanPerInterval) / 2
    newActivity$steps[i] <- meanPerInterval
  }
}
#Now with the new data
stepsPerDate <- aggregate(steps ~ date, newActivity, sum, na.rm=T)
#Calculate mean and median
meanStepsPerDay <- mean(stepsPerDate$steps)
medianStepsPerDay <- median(stepsPerDate$steps)
#Plot the histogram
hist(stepsPerDate$steps, breaks = 1000 * (0:22), main = "Frequency of steps per day", xlab = "Steps per day")
abline(v = meanStepsPerDay, col = "blue", lw = 2)
```

![](PA1_template_files/figure-html/imputation-1.png) 

```r
cat("The mean is", meanStepsPerDay, "steps.")
```

```
## The mean is 10766.19 steps.
```

```r
cat("The median is", medianStepsPerDay, "steps.")
```

```
## The median is 10766.19 steps.
```

## Are there differences in activity patterns between weekdays and weekends?
