Course project 1

Load the data

```r
  unzip("activity.zip")
  activity<-read.csv("activity.csv")
```

Remove NAs


```r
activity_na_removed<-activity[!is.na(activity$steps),]
```

What is mean total number of steps taken per day?

```r
  total_steps<-tapply(activity_na_removed$steps, activity_na_removed$date,sum)
  hist(total_steps, xlab="Total number of steps taken each day", main="")
```

![](PA1_template_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Mean total number of steps take per day

```r
mean(total_steps, na.rm = TRUE)
```

```
## [1] 10766.19
```
Median total nubmer of steps taken per da


```r
median(total_steps, na.rm =TRUE)
```

```
## [1] 10765
```


What is the average daily activity pattern?

Calculate average numbers of steps per each interval across all days.
Then, make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
    average_per_int<-tapply(activity_na_removed$steps,activity_na_removed$interval,mean)
  plot(names(average_per_int),average_per_int, type="l", xlab="5-minute interval",  ylab="Average number of steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-6-1.png)<!-- -->


Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
names(average_per_int[average_per_int==max(average_per_int)])
```

```
## [1] "835"
```

Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with NAs)

```r
sum(is.na(activity$steps))
```

```
## [1] 2304
```

Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Here, I will use the mean for the corresponding 5-minute interval to impute missing data 

Creat dataframe with missing data

```r
na<-activity[is.na(activity$steps),]
```

Create dataframe with caluculated averages per 5-minute inreval for imputation

```r
average<-data.frame(interval=names(average_per_int), mean_steps=average_per_int)
```

Impute NAs with average numbers of steps from each corresponding 5-minute  interval

```r
na<-merge(na,average, by="interval")
library(dplyr)
```

```
## Warning: package 'dplyr' was built under R version 3.3.1
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

```r
na<-mutate(na,steps=mean_steps)
na<-na[,-4]
activity_imputed<-rbind(activity[!is.na(activity$steps),],na)
```

Calcuate and plot Histogram the total number of steps taken each day using imputed dataset

```r
total_steps1<-tapply(activity_imputed$steps,activity_imputed$date,sum)
hist(total_steps1, xlab="Total number of steps taken per day", main="")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

mean total number of steps taken per day


```r
mean(total_steps1)
```

```
## [1] 10766.19
```
Median total number of steps taken per day


```r
median(total_steps1)
```

```
## [1] 10766.19
```

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?

The mean and median total number of steps taken per day calculated by removing missing values are 10766.19 and 10765, respectively.
On the other hand, those calculated after imputaion are both 10766.19

Therefore, the means calucated before and after the imputation are equal while medians are  different only slightly. 
Overall, the imputation had only little impact on the estimates of the total daily number of steps.

Are there differences in activity patterns between weekdays and weekends?


```r
Sys.setlocale("LC_TIME", "English")
```

```
## [1] "English_United States.1252"
```

```r
activity_imputed$date<-as.Date(activity_imputed$date)
activity_imputed<-mutate(activity_imputed,weekdays=weekdays(activity_imputed$date))
weekday<- c("Monday","Tuesday","Wednesday", "Thursday", "Friday")  
weekend<-c("Saturday", "Sunday")
activity_imputed[activity_imputed$weekdays %in% weekday,"weekdays2"]<-"weekday"
activity_imputed[activity_imputed$weekdays %in% weekend,"weekdays2"]<-"weekend"
activity_imputed$weekdays2<-as.factor(activity_imputed$weekdays2)
average_weekdays<-aggregate(steps ~ interval + weekdays2,data=activity_imputed,mean)
```



```r
library(ggplot2)
qplot(interval, steps, data=average_weekdays, facets = weekdays2~., geom="line")
```

![](PA1_template_files/figure-html/unnamed-chunk-16-1.png)<!-- -->
