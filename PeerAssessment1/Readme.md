---
title: "Reproducible Research PA1"
author: "Faisal Sardar"
date: "Saturday, May 16, 2015"
output: pdf_document
---
```{r,echo=FALSE, warning = FALSE}
library(data.table)
library(ggplot2)
library(plyr)
library(lattice)
#setwd("C:/Users/FS/Dropbox/DataScience/05-ReproducibleResearch/PA1")
setwd("C:/Users/Faisal/Dropbox/DataScience/05-ReproducibleResearch/PeerAssessment1")
```

<br>

**Loading and preprocessing the data**

Show any code that is needed to

- Load the data (i.e. read.csv())

- Process/transform the data (if necessary) into a format suitable for your analysis

```{r,results='hide'}
if(!file.exists("data")){
  dir.create("data")
}

if(!"Activity.zip" %in% dir("./data/")){
  URLFile<-"https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
  download.file(URLFile,destfile="./data/Activity.zip")
  unzip("./data/Activity.zip", files = NULL, list = FALSE, overwrite = TRUE,
        junkpaths = FALSE, exdir = "./data", unzip = "internal",
        setTimes = FALSE)
  dateDownloaded<-date()
} else {print("Already downloaded!")}


AMonitor<-data.table(read.csv("./data/activity.csv"))
head(AMonitor)
str(AMonitor)

#ignoring rows with na
AMonitor<-na.omit(AMonitor)
AMonitor <- AMonitor[, date := as.Date(date)]
setkey(AMonitor, date, interval)
head(AMonitor)
```
<br>
**What is mean total number of steps taken per day?**

For this part of the assignment, you can ignore the missing values in the dataset.

- Calculate the total number of steps taken per day
```{r}
TotalDailySteps <- AMonitor[, list(DailySteps = sum(steps)), date]
TotalDailySteps
```

- If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
```{r}
ggplot(TotalDailySteps, aes(x=DailySteps)) +
  geom_histogram(alpha=.5)
```

- Calculate and report the mean and median of the total number of steps taken per day
```{r}
mean(TotalDailySteps$DailySteps)
median(TotalDailySteps$DailySteps)
```

<br>
**What is the average daily activity pattern?**

- Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
```{r}
steps_by_interval <- aggregate(steps ~ interval, data=AMonitor, mean)
plot(steps_by_interval$interval,steps_by_interval$steps, type="l", 
     xlab="Interval", ylab="# Steps",main="Average Steps by 5 min intervals")
```

- Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```{r}
MeanStepsPerInterval<-ddply(AMonitor, c("interval"),summarise,meansteps = mean(steps))
```

Max Interval:
```{r}
MeanStepsPerInterval[which(MeanStepsPerInterval$meansteps
                   ==max(MeanStepsPerInterval$meansteps)), "interval"]
```
Max Steps:
```{r}
MeanStepsPerInterval[which(MeanStepsPerInterval$meansteps
                   ==max(MeanStepsPerInterval$meansteps)), "meansteps"]
```

<br>
**Imputing missing values**

Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

- Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
```{r}
MissingValues<-data.table(read.csv("./data/activity.csv"))
CountMV <- sum(!complete.cases(MissingValues))
CountMV
```

<br>
**Are there differences in activity patterns between weekdays and weekends?**

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

- Create a new factor variable in the dataset with two levels -- "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.

```{r, results='hide'}
DOWLevels <- c("Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday")
WDWELevels <- c("Weekend", "Weekday", "Weekday","Weekday","Weekday","Weekday","Weekday","Weekend")
DOWComp <- AMonitor[, DOW := factor(weekdays(date), levels=DOWLevels)]
DOWComp <- AMonitor[, WDWE := factor(WDWELevels[DOW])]
DOWComp[, .N, list(WDWE, DOW)]
WDWEIntervals <- DOWComp[, list(meanSteps = mean(steps)), list(WDWE, interval)]
```
```{r,echo=FALSE}
head(DOWComp,7)
head(WDWEIntervals,7)
```

- Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). The plot should look something like the following, which was creating using simulated data:

```{r}
xyplot(meanSteps ~ interval | WDWE, 
       data=WDWEIntervals, type="l", layout=c(1,2), 
       ylab = "Mean Steps", main="Weekday vs. Weekend - Mean Steps Per 5-Minute Interval")
```