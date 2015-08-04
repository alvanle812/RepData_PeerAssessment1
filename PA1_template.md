# RepData_PeerAssessment1
Ivan Lee  
Aug 4, 2015  
# Reproducible Research: Peer Assessment 1

## 1. Loading and preprocessing the data
Show any code that is needed to 
1.1 Load the data (i.e. read.csv())
1.2 Process/transform the data (if necessary) into a format suitable for your analysis


```r
if(!file.exists("./data")){dir.create("./data")}
fileUrl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
download.file(fileUrl, destfile = "./data/activitydata.zip", method = "curl")
unzip("./data/activitydata.zip", exdir = "./data")
activitydata <- read.csv("./data/activity.csv")
```

## 2. What is mean total number of steps taken per day?
For this part of the assignment, you can ignore the missing values in the dataset.
2.1 Calculate the total number of steps taken per day

```r
totalsteps <- aggregate(steps ~ date, activitydata, sum)
```
2.2 If you do not understand the difference between a histogram and a barplot, research the difference between them. Make a histogram of the total number of steps taken each day

```r
hist(totalsteps$steps, main="Total Number of Steps per Day", xlab="Steps per Day",col="black", breaks = nrow(totalsteps))
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png) 
2.3 Calculate and report the mean and median of the total number of steps taken per day

```r
mean(totalsteps$steps)
```

```
## [1] 10766.19
```

```r
median(totalsteps$steps)
```

```
## [1] 10765
```
The mean total number of steps taken per day is 10766.19  
The median total number of steps taken per day is 10765

## 3. What is the average daily activity pattern?

3.1 Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)

```r
intervalstepavg <- aggregate(steps~interval, data=activitydata, FUN=mean)
plot(intervalstepavg, type="l", main="Average Daily Activity Pattern", xlab="5-minute Intervals Over Day", ylab="Average Steps Taken Over All Days")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png) 

3.2 Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?

```r
maxinterval <- intervalstepavg[which.max(intervalstepavg$steps), 1]
```
On average across all the days in the dataset, it is the 835 contains the maximum number of steps


## 4. Imputing missing values
Note that there are a number of days/intervals where there are missing values (coded as NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

4.1 Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
missingval <- sum(is.na(activitydata$steps))
```
The total number of missing values in the dataset is 2304 (i.e. the total number of rows with NAs)

4.2 Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

```r
# replace all the missing values with mean for that day; if is NA, replace with 0
meanstepoftheday <- aggregate(steps ~ date, data = activitydata, FUN = mean)
activitydatamg <- merge(activitydata, meanstepoftheday, by.x = "date", by.y = "date", all.x = TRUE)
```
4.3 Create a new dataset that is equal to the original dataset but with the missing data filled in.

```r
activitydatamg$steps.x[is.na(activitydatamg$steps.x)] <- activitydatamg$steps.y
```

```
## Warning in activitydatamg$steps.x[is.na(activitydatamg$steps.x)] <-
## activitydatamg$steps.y: number of items to replace is not a multiple of
## replacement length
```

```r
activitydatamg$steps.x[is.na(activitydatamg$steps.x)] <- 0
```
4.4 Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day. Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

```r
totalstepsnona <- aggregate(steps.x ~ date, data = activitydatamg, sum)
hist(totalstepsnona$steps.x, main="Total Number of Steps per Day", xlab="Steps per Day",col="black", breaks = nrow(totalsteps))
```

![](PA1_template_files/figure-html/unnamed-chunk-10-1.png) 

```r
meannona <- mean(totalsteps$steps)
mediannona <- median(totalsteps$steps)
```
The mean total number of steps taken per day  removing NAs is 10766.19  
The median total number of steps taken per day removing NAs is 10765

The impact of the imputation was a slight increase in the median total number of steps per day.

## 5. Are there differences in activity patterns between weekdays and weekends?

For this part the weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.

5.1 Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.

```r
    ## change date column from factor to Date
    
    activitydatamg$date <- as.Date(activitydatamg$date)

    ## create a new factor variable in the dataset with two levels 
    ## “weekday” and “weekend” indicating whether date is a weekday or weekend day.

    weekend.days <- c("Saturday","Sunday")
    activitydatamg$daytype <- as.factor(sapply(activitydatamg$date, function(x) ifelse(weekdays(x) %in% weekend.days,"weekend","weekday")))
```

5.2 Make a panel plot containing a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.

```r
    require(plyr)
```

```
## Loading required package: plyr
```

```r
    average.steps <- ddply(activitydatamg, .(interval, daytype), summarize, steps = mean(steps.x))

    require(lattice)
```

```
## Loading required package: lattice
```

```r
    xyplot(steps ~ interval | daytype, data = average.steps, layout = c(1, 2), type = "l", 
     xlab="5-minute Intervals Over Day", ylab="Number of Steps",
     main="Activity Patterns on Weekends and Weekdays")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png) 
