---
title: "Course Project 1"
author: "Author - Guneet Kalsi"
date: "Date - 01/06/2020"
output: html_document
---

## **1. The first step of the analysis is to unzip the source file, load suitable libraries and preprocess data**
```{r load, echo=TRUE}
library(ggplot2)
library(dplyr)
setwd("E:/datasciencecoursera")
zipfile <- "E:/datasciencecoursera/repdata_data_activity.zip"
outdir <- "E:/datasciencecoursera"
unzip(zipfile, exdir = outdir)
activity <- read.csv("activity.csv")
str(activity)
```
The dataset includes three variables namely steps, date and interval.

## **2. Histogram of the total number of steps taken each day**
***Number of steps per day***
```{r steps, echo=TRUE}
stepsperday <- aggregate(activity$steps, list(activity$date), FUN=sum)
colnames(stepsperday) <- c("Date", "Steps")
stepsperday
```
***Histogram for total number of steps taken each day***
```{r hist, echo=TRUE}
g <- ggplot(stepsperday, aes(Steps))
g+geom_histogram(boundary=0, binwidth=2500, col="darkblue", fill="lightblue")+ggtitle("Histogram of steps per day")+xlab("Steps")+ylab("Frequency")+theme(plot.title = element_text(face="bold", size=12))+scale_x_continuous(breaks=seq(0,25000,2500))+scale_y_continuous(breaks=seq(0,18,2))
```

## **3. Mean and median number of steps taken each day**
```{r m, echo=TRUE}
mean(stepsperday$Steps, na.rm = TRUE)
median(stepsperday$Steps, na.rm = TRUE)
```

## **4. Average daily activity pattern**
***Time series plot of the 5 minute interval (x) and averaged number of steps taken averaged across all days (y)***
```{r avg, echo=TRUE}
stepspertime <- aggregate(steps~interval,data=activity,FUN=mean,na.action=na.omit)
stepspertime$time <- stepspertime$interval/100
h <- ggplot(stepspertime,aes(time, steps))
h+geom_line(col="orange")+ggtitle("Average steps per time interval")+xlab("Time")+ylab("Steps")+theme(plot.title = element_text(face="bold", size=12))
```

## **5. The 5-minute interval that, on average, contains the maximum number of steps**
```{r max, echo=TRUE}
ST <- tbl_df(stepspertime)
ST %>% select(time, steps) %>% filter(steps==max(ST$steps))
```

## **6. Imputing missing data**
***Total number of missing values in the dataset***
```{r na, echo=TRUE}
act <- tbl_df(activity)
act %>% filter(is.na(steps)) %>% summarize(missing_values = n())
```

***Replace missing values***
```{r replace, echo=TRUE}
activity$CompleteSteps <- ifelse(is.na(activity$steps), round(stepspertime$steps[match(activity$interval, stepspertime$interval)],0), activity$steps)
```

***New dataset with missing data filled in***
```{r new, echo=TRUE}
activityfull <- data.frame(steps=activity$CompleteSteps, interval=activity$interval, date=activity$date)
head(activityfull, n=10)
```

## **7. Histogram of the total number of steps taken each day after missing values are imputed**
```{r histogram, echo=TRUE}
stepsperdayfull <- aggregate(activityfull$steps, list(activityfull$date), FUN=sum)
colnames(stepsperdayfull) <- c("Date", "Steps")
g <- ggplot(stepsperdayfull, aes(Steps))
g+geom_histogram(boundary=0, binwidth=2500, col="brown", fill="pink")+ggtitle("Histogram of steps per day")+xlab("Steps")+ylab("Frequency")+theme(plot.title = element_text(face="bold", size=12))+scale_x_continuous(breaks=seq(0,25000,2500))+scale_y_continuous(breaks=seq(0,26,2))
```

## **8. Panel plot comparing the average number of steps taken per 5-minute interval across weekdays and weekends**
***Two time series plot of the 5-minute interval (x) and the average number of steps taken averaged across weekday days or weekend days (y)***
```{r comapre, echo=TRUE}
activityfull$RealDate <- as.Date(activityfull$date, format = "%Y-%m-%d")
activityfull$weekday <- weekdays(activityfull$RealDate)
activityfull$DayType <- ifelse(activityfull$weekday=='Saturday' | activityfull$weekday=='Sunday', 'weekend','weekday')
stepspertimedt<- aggregate(steps~interval+DayType,data=activityfull,FUN=mean,na.action=na.omit)
stepspertimedt$time <- stepspertime$interval/100
j <- ggplot(stepspertimedt, aes(time, steps))
j+geom_line(col="red")+ggtitle("Average steps per time interval: weekdays vs. weekends")+xlab("Time")+ylab("Steps")+theme(plot.title = element_text(face="bold", size=12))+facet_grid(DayType ~ .)
```