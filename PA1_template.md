# Reproducible Research: Peer Assessment 1


### Loading and preprocessing the data

Read activity.csv file into "activity" variable

```r
activity <- read.csv(file="C:/Users/BlackBeast/Documents/GitHub/RepData_PeerAssessment1/activity/activity.csv")
```
Convert date field from factor to date


```r
activity$date <- as.Date(activity$date)
```
Load reshape2 library to get melt & dcast functions


```r
library(reshape2)
```
Melt data frame to prep for casting by date -- by setting the id variable to date and the measure variable to steps, we get a table with multiple values for steps taken within each day


```r
actMeltDate <- melt(activity, id.vars="date", measure.vars="steps", na.rm=FALSE)
```
Cast data frame to see steps per day -- this sums the steps by date to give us a table of 3 columns x 61 rows


```r
actCastDate <- dcast(actMeltDate, date ~ variable, sum)
```
Plot histogram with frequency of steps by day and add a red line showing the mean value

```r
plot(actCastDate$date, actCastDate$steps, type="h", main="Histogram of Daily Steps", xlab="Date", ylab="Steps per Day", col="blue", lwd=8)
abline(h=mean(actCastDate$steps, na.rm=TRUE), col="red", lwd=2)
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)
png("instructions_fig/histodaily.png")

### What is mean total number of steps taken per day?

Calculate mean and median of daily steps

```r
paste("Mean Steps per Day =", mean(actCastDate$steps, na.rm=TRUE))
```

```
## [1] "Mean Steps per Day = 10766.1886792453"
```
[1] "Mean Steps per Day = 10766.1886792453"

```r
paste("Median Steps per Day =", median(actCastDate$steps, na.rm=TRUE))
```

```
## [1] "Median Steps per Day = 10765"
```
[1] "Median Steps per Day = 10765"

### What is the average daily activity pattern?

Re-melt data frame to prep for casting by interval, including removing NA values so we can take the mean a little later

```r
actMeltInt <- melt(activity, id.vars="interval", measure.vars="steps", na.rm=TRUE)
```
Cast data frame to see mean steps per interval

```r
actCastInt <- dcast(actMeltInt, interval ~ variable, mean)
```
Plot line chart with average frequency of steps by interval and add line with mean

```r
plot(actCastInt$interval, actCastInt$steps, type="l", main="Frequency of Steps Taken at Each Interval", xlab="Interval ID", ylab="Steps", col="orange", lwd=2)
abline(h=mean(actCastInt$steps, na.rm=TRUE), col="red", lwd=2)
```

![](PA1_template_files/figure-html/unnamed-chunk-11-1.png)
png("instructions_fig/StepFreq.png")

The plot shows the peak at somewhere in the 800-900 interval range so let's find out exactly which interval has the max value and what that maximum value is
Output interval that has max value along with the max value

```r
paste("Interval with max value =", actCastInt$interval[which(actCastInt$steps == max(actCastInt$steps))])
```

```
## [1] "Interval with max value = 835"
```
[1] "Interval with max value = 835"

```r
paste("Maximum interval mean steps =", max(actCastInt$steps))
```

```
## [1] "Maximum interval mean steps = 206.169811320755"
```
[1] "Maximum interval mean steps = 206.169811320755"

### Imputing missing values
Calculate number of rows in activity data set with NA rows

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```
[1] 2304

Data frame with mean steps per interval - just renaming to be more descriptive

```r
stepsPerInt <- actCastInt
```
Create data frame that we will remove NAs from

```r
actNoNA <- activity
```
Merge activity data set with stepsPerInt data set

```r
actMerge = merge(actNoNA, stepsPerInt, by="interval", suffixes=c(".act", ".spi"))
```
Get list of indexes where steps value = NA

```r
naIndex = which(is.na(actNoNA$steps))
```
Replace NA values with value from steps.spi

```r
actNoNA[naIndex,"steps"] = actMerge[naIndex,"steps.spi"]
```
Melt new data frame to prep for casting by date

```r
actMeltDateNoNA <- melt(actNoNA, id.vars="date", measure.vars="steps", na.rm=FALSE)
```
Cast data frame to see steps per day

```r
actCastDateNoNA <- dcast(actMeltDateNoNA, date ~ variable, sum)
```
Plot histogram with frequency of steps by day

```r
plot(actCastDateNoNA$date, actCastDateNoNA$steps, type="h", main="Histogram of Daily Steps (Imputted NA Values)", xlab="Date", ylab="Steps", col="gray", lwd=8)
abline(h=mean(actCastDateNoNA$steps), col="red", lwd=2)
```

![](PA1_template_files/figure-html/unnamed-chunk-22-1.png)
png("instructions_fig/HistoDailyNA.png")

Calculate mean and median of daily steps

```r
paste("Mean daily steps =", mean(actCastDateNoNA$steps, na.rm=TRUE))
```

```
## [1] "Mean daily steps = 10889.7992576554"
```
[1] "Mean daily steps = 10889.7992576554"

```r
paste("Median daily steps =", median(actCastDateNoNA$steps, na.rm=TRUE))
```

```
## [1] "Median daily steps = 11015"
```
[1] "Median daily steps = 11015"

Original dataset containing NAs: Mean Daily Steps = 10,766.19, Median Daily Steps = 10,765.

Dataset cleaned of NAs: Mean Daily Steps = 10890, Median Daily Steps = 11,015.

1.2% difference for mean, 2.3% difference for median. 

### Are there differences in activity patterns between weekdays and weekends?

For loop to create new column called "dayOfWeek" and insert whether each date corresponds to a weekday or weekend

```r
for (i in 1:nrow(actNoNA)) {
    if (weekdays(actNoNA$date[i]) == "Saturday" | weekdays(actNoNA$date[i]) == "Sunday") {
        actNoNA$dayOfWeek[i] = "weekend"
    } else {
        actNoNA$dayOfWeek[i] = "weekday"
    }
}
```
To create a plot, we must first subset the data

```r
actWeekday <- subset(actNoNA, dayOfWeek=="weekday")
actWeekend <- subset(actNoNA, dayOfWeek=="weekend")
```
Next, we need to process the data for our needs

```r
actMeltWeekday <- melt(actWeekday, id.vars="interval", measure.vars="steps")
actMeltWeekend <- melt(actWeekend, id.vars="interval", measure.vars="steps")
actCastWeekday <- dcast(actMeltWeekday, interval ~ variable, mean)
actCastWeekend <- dcast(actMeltWeekend, interval ~ variable, mean)
```
Load plot packages necessary to continue

```r
library(ggplot2)
library(gridExtra)
```
Set plot area to two rows and one column, and then plot charts with mean line in each

```r
plot1 <- qplot(actCastWeekday$interval, actCastWeekday$steps, geom="line", data=actCastWeekday, main="Steps by Interval - Weekday", xlab="Interval ID", ylab="Number of Steps")
plot2 <- qplot(actCastWeekend$interval, actCastWeekend$steps, geom="line", data=actCastWeekend, main="Steps by Interval - Weekend", xlab="Interval ID", ylab="Number of Steps")
grid.arrange(plot1, plot2, nrow=2)
```

![](PA1_template_files/figure-html/unnamed-chunk-29-1.png)
png("instructions_fig/Steps-by-Interval.png")
