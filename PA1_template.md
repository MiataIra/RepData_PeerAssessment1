---
title: "Reproducible Research: Peer Assessment 1"
output: 
  html_document:
    keep_md: TRUE
---



## 1. Loading and preprocessing the data

### 1.1. Load the data (i.e. read.csv())

Load the data (i.e. \color{red}{\verb|read.csv()|}read.csv())


```r
activity_data_original<-read.csv("./activity/activity.csv")
```

### 1.2. Transformation the data 

Process/transform the data (if necessary) into a format suitable for your analysis.



## 2. What is mean total number of steps taken per day?

For this part of the assignment, you can ignore the missing values in the dataset.


```r
activity_data_original_isna<-na.omit(activity_data_original)
```


### 2.1. Calculate the total number of steps taken per day


```r
total_per_day<-activity_data_original_isna%>%
  group_by(date) %>%
  summarise(steps=sum(steps))%>%
  as.data.frame()
```


### 2.2. Make a histogram of the total number of steps taken each day


```r
hist(total_per_day$steps, xlab = "Steps per Day", main = "Total number of steps taken per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

### 2.3. Calculate and report the mean and median of the total number of steps taken per day


```r
Summary<-summary(total_per_day$steps)
Summary["Mean"]
```

```
##     Mean 
## 10766.19
```

```r
Summary["Median"]
```

```
## Median 
##  10765
```

* Mean: 10766.2
* Median:  10765


## 3. What is the average daily activity pattern?


```r
average_per_interval <- activity_data_original_isna %>%
  group_by(interval) %>%
  summarize(mean_steps = mean(steps))
```

### 3.1. Make a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all days (y-axis)


```r
plot(x=average_per_interval$interval, y=average_per_interval$mean_steps, type = "l",col="red", xlab = "Intervals", ylab = "Total steps per interval", main = "Averaged number of steps per interval")
```

![](PA1_template_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

### 3.2. Which 5-minute interval, on average across all the days in the dataset, contains the maximum number of steps?


```r
interval_with_max<-average_per_interval[which.max(average_per_interval$mean_steps), "interval"]
```

* Interval that contains the maximun number of steps is: 835


## 4. Imputing missing values

Note that there are a number of days/intervals where there are missing values (coded as \color{red}{\verb|NA|}NA). The presence of missing days may introduce bias into some calculations or summaries of the data.

### 4.1. Calculate and report the total number of missing values in the dataset (i.e. the total number of rows with \color{red}{\verb|NA|}NAs)


```r
number_na<-colSums(is.na(activity_data_original))
```

* The total number of rows with NA is 2304.


### 4.2. Devise a strategy for filling in all of the missing values in the dataset. The strategy does not need to be sophisticated. For example, you could use the mean/median for that day, or the mean for that 5-minute interval, etc.

Filling in all of the missing values in the dataset with the mean for that 5-minute interval.

### 4.3. Create a new dataset that is equal to the original dataset but with the missing data filled in.


```r
filling_na<-inner_join(activity_data_original, average_per_interval)
```

```
## Joining, by = "interval"
```

```r
filling_na$steps[which(is.na(filling_na$steps))]<-filling_na$mean_steps[which(is.na(filling_na$steps))]
filling_na<-select(filling_na, steps, date, interval)
filling_na$steps<-sapply(filling_na$steps, function(x) round(x, digits = 0))
```

### 4.4. Make a histogram of the total number of steps taken each day and Calculate and report the mean and median total number of steps taken per day.

Do these values differ from the estimates from the first part of the assignment? What is the impact of imputing missing data on the estimates of the total daily number of steps?


```r
total_per_day_filling_na<-filling_na%>%
  group_by(date) %>%
  summarise(steps=sum(steps))%>%
  as.data.frame()
hist(total_per_day_filling_na$steps, xlab = "Steps per Day", main = "Total number of steps taken per day")
```

![](PA1_template_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

```r
ny_summary<-summary(total_per_day_filling_na$steps)
ny_summary["Mean"]
```

```
##     Mean 
## 10765.64
```

```r
ny_summary["Median"]
```

```
## Median 
##  10762
```

* Mean: 10765.6
* Median:  10762
The values differ a little from previously.

## 5. Are there differences in activity patterns between weekdays and weekends?

For this part the \color{red}{\verb|weekdays()|}weekdays() function may be of some help here. Use the dataset with the filled-in missing values for this part.


### 5.1. Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.


```r
filling_na$date <- ymd(filling_na$date)
filling_na <- mutate(filling_na, weektype = ifelse(weekdays(filling_na$date) == "lørdag" | weekdays(filling_na$date) == "søndag", "weekend", "weekday"))
filling_na$weektype <- as.factor(filling_na$weektype)
```


### 5.2. Make a panel plot containing a time series plot (i.e. \color{red}{\verb|type = "l"|}type="l") of the 5-minute interval (x-axis) and the average number of steps taken, averaged across all weekday days or weekend days (y-axis). 

See the README file in the GitHub repository to see an example of what this plot should look like using simulated data.


```r
filling_na_mean_per_interval <- filling_na %>%
  group_by(interval, weektype) %>%
  summarise(mean_steps = mean(steps))

ggplot(filling_na_mean_per_interval, aes(x=interval, y=mean_steps)) +
  geom_line(col="red") +
  facet_wrap(~weektype, ncol = 1, nrow=2)+
  labs(x = "Intervals", y = "Average Steps")
```

![](PA1_template_files/figure-html/unnamed-chunk-14-1.png)<!-- -->


