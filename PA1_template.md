# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Show any code that is needed to

1. Load the data (i.e. read.csv())

2. Process/transform the data (if necessary) into a format suitable for your analysis



```r
#Set the working directory, if necessary
#setwd(".../RepData_PeerAssessment1")
library(lubridate)
#There is the option to download the data from the internet
#url <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
#download.file(url, "steps.csv")
#But we prefer work with the data from the repo
unzip("activity.zip")
activity <- read.csv("activity.csv")
activity$date <- as.Date(activity$date)
#Format the variable time (saved as an instant of the current day) to specify the time of day that variable interval is referring, then every data has a date and a time instant associated with it, which is necessary for aggrupation
activity$interval <- as.character(sprintf("%04d", activity$interval))
activity$time <- strptime(activity$interval, "%H%M")
#Finally, format the interval variable as a character easy to understand
activity$interval <- format(activity$time, format="%H:%M")
print(str(activity))
```

```
## 'data.frame':	17568 obs. of  4 variables:
##  $ steps   : int  NA NA NA NA NA NA NA NA NA NA ...
##  $ date    : Date, format: "2012-10-01" "2012-10-01" ...
##  $ interval: chr  "00:00" "00:05" "00:10" "00:15" ...
##  $ time    : POSIXlt, format: "2015-01-18 00:00:00" "2015-01-18 00:05:00" ...
## NULL
```

## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Make a histogram of the total number of steps taken each day

2. Calculate and report the mean and median total number of steps taken per day

```r
stepsPerDate <- aggregate(steps ~ date, activity, sum, na.rm=T)
#Calculate mean and median
meanStepsPerDay <- round(mean(stepsPerDate$steps), digits=1)
medianStepsPerDay <- round(median(stepsPerDate$steps), digits=1)
#Plot the histogram
hist(stepsPerDate$steps, breaks = 1000 * (0:22), main = "Frequency of steps per day", xlab = "Steps per day")
#Add a vertical line indicating the mean number of steps per day
abline(v = meanStepsPerDay, col = "blue", lw = 2)
```

![](PA1_template_files/figure-html/histogram-1.png) 

**The mean is 10766.2 steps per day.**

**The median is 10765 steps per day.**

## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
#Use the interval (character) variable to aggregete (time variables don't allow aggregation)
stepsPerInterval <- aggregate(steps ~ interval, activity, mean, na.rm=T)
#Create a variable time (character variable don't allow ploting)
stepsPerInterval$time <- strptime(stepsPerInterval$interval, "%H:%M")
maxPerDay <- max(stepsPerInterval$steps, na.rm=TRUE)
maxTime <- stepsPerInterval$time[stepsPerInterval$steps == maxPerDay]
#Make plot steps vs. time
with(stepsPerInterval, plot(time, steps, type="l", xlab="Hour", ylab="Mean number of steps", main="Activity during day", xaxt="n"))
tickMarks = as.POSIXct(as.character(seq(0, 24, by=2)), format="%H")
axis.POSIXct(1, at = tickMarks)
```

![](PA1_template_files/figure-html/day-1.png) 

**In average, the time interval with more activity along the day is from 8:35 hs to 8:40 hs.**

## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

3. Create a new dataset that is equal to the original dataset but with the missing data filled in.

4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
#Counting the NAs records
activity$isNA <- is.na(activity$steps)
numberOfNAs <- table(activity$isNA)["TRUE"]
#Make a copy to imput data
newActivity <- activity
#Imputation strategy: 
#For each NA record, take the mean of every data with the same date, and the mean of every data with the same interval (time of the day) and take the average of both means. If every information for date is absent, imput the mean of the time of day.
for(i in 1:nrow(newActivity)){
  #For every NA data...
  if(newActivity$isNA[i]){
    #1) found mean per interval (time of the day)
    meanPerInterval <- stepsPerInterval$steps[stepsPerInterval$interval == newActivity$interval[i]]
    #2) If there's data for the date, ...
    if(activity$date[i] %in% stepsPerDate$date){
      #... found mean per date and average both means; ...
      meanPerDate <- stepsPerDate$steps[stepsPerDate$date == activity$date[i]]
      newActivity$steps[i] <- (meanPerDate + meanPerInterval) / 2}
    #... if there's not data for the date
      else{
        #... just take mean per interval
        newActivity$steps[i] <- meanPerInterval}}}
#Now with the new data, repeat the first chunk
stepsPerDate <- aggregate(steps ~ date, newActivity, sum, na.rm=T)
#Calculate mean and median
meanStepsPerDay <- round(mean(stepsPerDate$steps), digits=1)
medianStepsPerDay <- round(median(stepsPerDate$steps), digits=1)
#Plot the histogram
hist(stepsPerDate$steps, breaks = 1000 * (0:22), main = "Frequency of steps per day", xlab = "Steps per day")
#Add a vertical line indicating the mean number of steps per day
abline(v = meanStepsPerDay, col = "blue", lw = 2)
```

![](PA1_template_files/figure-html/imputation-1.png) 

**Originally there was 2304 missing data.**

Missing data were imputated averaging the mean of every data in the selected time interval and the mean of every data in the selected date (if available).

**The mean, having imputed data, is 10766.2 steps per day.**

**The median, having imputed data, is 10766.2 steps per day.**

The differences with respect to non-imputed data (above) are negligible.

## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
#Weekday or weekend
kindOfDay <- function(date){
  #Take a date and return "weekday" for Mon to Fri or "weekend" for Sat and Sun
  sat = weekdays(as.Date("2015-01-03"))
  sun = weekdays(as.Date("2015-01-04"))
  if(weekdays(date) %in% c(sat, sun)){
    return("weekend")}
  else{
    return("weekday")}}
#Add a variable indicating the kind of each date
for(i in 1:nrow(newActivity)){
  newActivity$kindOfDay[i] <- kindOfDay(as.Date(newActivity$date[i]))}
#Use the interval (character) variable and kindOfDay to aggregete (time variables don't allow aggregation) and repeat plotting-along-day chunk
stepsPerInterval <- aggregate(steps ~ interval + kindOfDay, newActivity, mean, na.rm=T)
#Create a variable time (character variable don't allow ploting)
stepsPerInterval$time <- strptime(stepsPerInterval$interval, "%H:%M")
#Make two plots steps vs. time, by kindOfDay
tickMarks = as.POSIXct(as.character(seq(0, 24, by=4)), format="%H")
par(mfrow=c(1, 2))
with(stepsPerInterval[stepsPerInterval$kindOfDay == "weekday",], plot(time, steps, type="l", xlab="Hour", ylab="Mean number of steps", ylim=c(0, 240), main="Weekdays", xaxt="n"))
axis.POSIXct(1, at=tickMarks)
with(stepsPerInterval[stepsPerInterval$kindOfDay == "weekend",], plot(time, steps, type="l", xlab="Hour", ylab="Mean number of steps", ylim=c(0, 240), main="Weekends", xaxt="n"))
axis.POSIXct(1, at=tickMarks)
```

![](PA1_template_files/figure-html/week-1.png) 

The maximum level of activity during weekends tends to be lesser than the maximum level of activity during weekdays. However, during worktime (9 - 17 hs), the weekends tend to be more active.
