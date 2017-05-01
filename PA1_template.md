# Reproducible Research: Peer Assessment 1


## Loading and preprocessing the data
Show any code that is needed to

1. Load the data (i.e. read.csv())
2. Process/transform the data (if necessary) into a format suitable for your analysis


```r
activity <-read.csv("activity/activity.csv")
```


## What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.

1. Calculate the total number of steps taken per day
2. If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day
3. Calculate and report the mean and median of the total number of steps taken per day

```r
stepsPerDay <-aggregate(activity$steps, list(activity$date), sum)
names(stepsPerDay) <- c("date", "totalSteps")
hist(stepsPerDay$totalSteps, breaks = 10, ylab="Total Steps", xlab="5 minute Interal", main="Daily total Steps by 5 minute interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-1-1.png)<!-- -->

```r
stepsPerDayAvg <- mean(stepsPerDay$totalSteps, na.rm=TRUE)
stepsPerDayAvgMedian <- median(stepsPerDay$totalSteps, na.rm=TRUE)
```
The Average number of steps taken every day is 1.0766189\times 10^{4}</br>
The Median of steps taken every day is 10765


## What is the average daily activity pattern?
1. Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)
2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
avgStepsPerInterval <- aggregate(activity$steps, list(activity$interval), mean, na.rm=TRUE)
names(avgStepsPerInterval) <- c("interval", "averageSteps")
plot(avgStepsPerInterval$interval, avgStepsPerInterval$averageSteps, type="l", xlab="5 Minutes interval", ylab="Daily Average Steps", main="Daily Averaged Steps by 5 minute interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

```r
maxStepsInterval <- avgStepsPerInterval[avgStepsPerInterval$averageSteps==max(avgStepsPerInterval$averageSteps),1]
```
Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps? Answer is 835.


## Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)
2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.
3. Create a new dataset that is equal to the original dataset but with the missing data filled in.
4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

Answer:
Total number of rows with NAs is: 2304
In order to fill NAs rows with values, I've created a new dataset replaceing the NAs with the averaged by the 5-minute interval:

```r
newActivity <- activity
i <- 1
for (i in 1:length(newActivity[,1])){
        if(is.na(newActivity[i,1]))
                newActivity[i,1] <- avgStepsPerInterval[avgStepsPerInterval$interval==newActivity[i,3],2]
}
stepsPerDayFixed <-aggregate(newActivity$steps, list(newActivity$date), sum)
names(stepsPerDayFixed) <- c("date", "totalSteps")
hist(stepsPerDayFixed$totalSteps, breaks = 10, ylab="Total Steps", xlab="5 minute Interal", main="Daily Total Steps by 5 minute interval (Imputed)")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

```r
stepsPerDayFixedAvg <- mean(stepsPerDayFixed$totalSteps, na.rm=TRUE)
stepsPerDayAvgFixedMedian <- median(stepsPerDayFixed$totalSteps, na.rm=TRUE)
```
After Fixing the the NAs:</br>
The Average number of steps taken every day is 1.0766189\times 10^{4} </br>
The Median of steps taken every day is 1.0766189\times 10^{4} </br>

No big difference between NA-included and NA-fixed mean and median.





## Are there differences in activity patterns between weekdays and weekends?
For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

1. Create a new factor variable in the dataset with two levels - "weekday" and "weekend" indicating whether a given date is a weekday or weekend day.
2. Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
newActivity["weekend"] <- weekdays(as.Date(newActivity$date))
newActivity$weekend[newActivity$weekend %in% c("Saturday","Sunday")] <- "weekend"
newActivity$weekend[newActivity$weekend != "weekend"] <- "midweek"
library("ggplot2")
newActivityByDay <- aggregate(newActivity$steps~newActivity$interval + newActivity$weekend, newActivity, mean)
names(newActivityByDay) <- c("interval","weekend", "steps")
ggplot(data=newActivityByDay, aes(interval, steps))+facet_grid(weekend~.)+geom_line()+labs(title="Daily Total Steps by 5 minute interval (Imputed)", subtitle="weekday vs midweek")
```

![](PA1_template_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

