---
title: "Reproducible Research, Peer Assessment 1"
author: "Mike McK"
date: "Monday, August 10, 2015"
output: html_document
---
  
  load data into R studio and do a few tests:
  
  ```{r initital investigations and manipulations}
act1 <- read.csv("activity.csv")
library(lubridate)
par(mfrow = c(1,1))
datesAsChar <- c(as.character(act1$date))
#datesAsList <- as.list(datesAsChar)
allDates <- ymd(datesAsChar)
# more appropriate Date column
act1$date <- allDates
wday <- wday(allDates, label =TRUE)
# day-of-the-week column added:
act1$wday <- wday
# check that everything is alright:
str(act1)
```

What is mean total number of steps taken per day?
=================================================
  For this part of the assignment, you can ignore the missing values in the dataset.
1. Calculate the total number of steps taken per day.
2. Make a histogram of the total number of steps taken each day.
3. Calculate and report the mean and median of the total number of steps taken per day.
```{r plot1}
sumStepsAllDays <- tapply(act1$steps, act1$date, sum)
sumStepsAllDays2<- as.table(sumStepsAllDays)
# histogram of number of steps per day
hist(sumStepsAllDays, breaks = 20, xlab="Sum of Steps per day", ylab="Number of days")
# median steps:
medianSteps<- median(sumStepsAllDays, na.rm = T)
medianSteps
# average steps:
meanSteps<- mean(sumStepsAllDays, na.rm = T)
meanSteps
```
Very interestingly, we can see that on most days the total number of steps is in the range 10,000 to 11,000, which is where both the median and mean are located, too. 

What is the average daily activity pattern?
=============================================
  1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?
```{r plot2}
stepsPerInterval <- tapply(act1$steps, act1$interval, mean, na.rm=T)
# number of average steps per five-minute time interval: 
plot(act1$interval[1:288], stepsPerInterval, type = "l", xlab="time interval", ylab="number of steps")

# same plot alternatively with ggplot:
library(ggplot2)
plt2 <- matrix(0, ncol = 2, nrow = 288)
plt2 <- data.frame(plt2)
names(plt2) <- c("interval", "meanSteps")
plt2$interval <- act1$interval[1:288]
plt2$meanSteps <- stepsPerInterval
ggplot(data = plt2, aes(x = interval, y = meanSteps)) + geom_line()
# number of average steps is still the same: 
# mean(plt2$meanSteps)
# the maximum number of steps is at the time interval 8:35-8:40am: 
plt2$interval[which(grepl(max(plt2$meanSteps), plt2$meanSteps))]
```
On average, the user is most active in the mornings (around 8-9am)! 
  
  
  
  
  Imputing missing values
=============================
  1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?
```{r missing values}
# Total number of missing values:
sum(is.na(act1$steps))
# rows where steps has value NA:
allNas <- act1[which(is.na(act1$steps)), ]
#these are the days of the week where we have NAs:
naDays <- allNas[which(which(is.na(act1$steps))%%288 == 1), 4]
summary(naDays)
# a test ((allNas-1)%/%288)%%7
#just the row numbers where steps is NA:
naRowNums <- which(is.na(act1$steps))
# one variable in plt2 was the means for the interval; this needs to be extracted and duplicated for eight days total (see summary(naDays)).
library(dplyr)
# simple copy of data:
meanVals <- plt2
# two days
meanVals <- bind_rows(meanVals, meanVals)
# for four days
meanVals <- bind_rows(meanVals, meanVals)
# now we have the mean steps data ready for eight days:
meanVals <- bind_rows(meanVals, meanVals)
names(meanVals)
#select only needed columns:
meanVals <- select(meanVals, meanSteps) 
# replace the NA values with the means for the 5 minute intervals, for all eight days:
actNew <- act1
actNew[naRowNums, 1] <- meanVals
# redo all calculations from part1:
sumStepsAllDaysPt3 <- tapply(actNew$steps, act1$date, sum)
hist(sumStepsAllDaysPt3, breaks = 20)
medianStepsPt3<- median(sumStepsAllDays, na.rm = T)
medianStepsPt3
meanStepsPt3 <- mean(sumStepsAllDays, na.rm = T)
meanStepsPt3

```
With the strategy chosen - to replace the missing steps values (NA) with the means of the five-minute intervals - there is no impact on the mean or the median, but the histogram changes: Since the eight missing days now have average values, they all fall into the middle column (number of steps is 10,000 to 11,000). All other columns are a bit smaller accordingly. So it's still a gauss disctibution, but the standard deviation will be smaller now. 




Are there differences in activity patterns between weekdays and weekends?
============================================================================
1.Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.
```{r weekdays vs weekends}
library(dplyr)
# filter all Weekend days (Sat, Sun):
allWE<- filter(actNew, wday == "Sat" |wday == "Sun")
# separately, filter all NON-weekend days (Mon thru Fri):
allWD<- filter(actNew, !(wday == "Sat" |wday == "Sun"))
#the average steps on weekend days (Sat, Sun): 
meanWE <- tapply(allWE$steps, allWE$interval, mean, na.rm=T)
par(mar = c(4,5,2,1))
par(mfrow = c(1,2))
# average steps per time interval, for the weekend: 
plot(actNew$interval[1:288], meanWE, type = "l", xlab = "five minute time interval (time of day)", ylab = "Average Number of Steps\n on Weekends")
# do the same evaluations and plot for the weekdays (Monday thru Friday), and plot these side by side:
meanWD <- tapply(allWD$steps, allWD$interval, mean, na.rm=T)
plot(actNew$interval[1:288], meanWD, type = "l", xlab = "five minute time interval (time of day)", ylab = "Average Number of Steps\n on Weekdays")
```

Examining the two plots side by side, we can draw some significant conclusions:
On weekends, any activities are much more distributed over the whole day, whereas on weekdays (Monday thru Friday) the most significant time for activities is in the morning (around 8am) and much less going on for the rest of the day.
