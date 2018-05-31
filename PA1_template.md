---
title: "Reproducible-Research-Proj1"
author: "Tejus Maduskar"
date: "May 29, 2018"
output: 
  html_document: 
    keep_md: yes
---



# This is the R markdown doc that describes the complete solution to the course project 1 

##(a) Preliminary work (done prior to the R code described in this document)
*(a1) Download file from course website*
*(a2) Unzip (extract) file called activity.csv*
*(a3) Move file to /data folder so it can be processed by R*

##(b) Loading and pre-processing data
*(b1) Load the data*

```r
actDF <- read.csv("C:/Users/Tejus Madusker/Desktop/R-Coursera/data/activity.csv")
```

##(c) What is mean total no. of steps taken per day? (ignore missing values in the dataset) 
*(c1) Calculate the total no. of steps taken per day*

```r
totStepsByDay <- aggregate(actDF$steps, list(actDF$date), sum)
names(totStepsByDay) <- c("Date", "Tot.Steps")
totStepsByDay
```

```
##          Date Tot.Steps
## 1  2012-10-01        NA
## 2  2012-10-02       126
## 3  2012-10-03     11352
## 4  2012-10-04     12116
## 5  2012-10-05     13294
## 6  2012-10-06     15420
## 7  2012-10-07     11015
## 8  2012-10-08        NA
## 9  2012-10-09     12811
## 10 2012-10-10      9900
## 11 2012-10-11     10304
## 12 2012-10-12     17382
## 13 2012-10-13     12426
## 14 2012-10-14     15098
## 15 2012-10-15     10139
## 16 2012-10-16     15084
## 17 2012-10-17     13452
## 18 2012-10-18     10056
## 19 2012-10-19     11829
## 20 2012-10-20     10395
## 21 2012-10-21      8821
## 22 2012-10-22     13460
## 23 2012-10-23      8918
## 24 2012-10-24      8355
## 25 2012-10-25      2492
## 26 2012-10-26      6778
## 27 2012-10-27     10119
## 28 2012-10-28     11458
## 29 2012-10-29      5018
## 30 2012-10-30      9819
## 31 2012-10-31     15414
## 32 2012-11-01        NA
## 33 2012-11-02     10600
## 34 2012-11-03     10571
## 35 2012-11-04        NA
## 36 2012-11-05     10439
## 37 2012-11-06      8334
## 38 2012-11-07     12883
## 39 2012-11-08      3219
## 40 2012-11-09        NA
## 41 2012-11-10        NA
## 42 2012-11-11     12608
## 43 2012-11-12     10765
## 44 2012-11-13      7336
## 45 2012-11-14        NA
## 46 2012-11-15        41
## 47 2012-11-16      5441
## 48 2012-11-17     14339
## 49 2012-11-18     15110
## 50 2012-11-19      8841
## 51 2012-11-20      4472
## 52 2012-11-21     12787
## 53 2012-11-22     20427
## 54 2012-11-23     21194
## 55 2012-11-24     14478
## 56 2012-11-25     11834
## 57 2012-11-26     11162
## 58 2012-11-27     13646
## 59 2012-11-28     10183
## 60 2012-11-29      7047
## 61 2012-11-30        NA
```

*(c2) Make a histogram of the total number of steps taken each day*

```r
hist(totStepsByDay$Tot.Steps, col="green", 
     main = "Histogram of Total Steps by Day", 
     xlab = "Total Daily Steps")
```

![](C:\Users\Tejus Madusker\Desktop\R-Coursera\Practice Code\output\PA1_template_files/figure-html/Histogram-1.png)<!-- -->

*(c3) Calculate and report the mean and median of the total number of steps taken per day*

```r
summary(totStepsByDay$Tot.Steps)["Mean"]
```

```
##     Mean 
## 10766.19
```

```r
summary(totStepsByDay$Tot.Steps)["Median"]
```

```
## Median 
##  10765
```

##(d) What is the average daily activity pattern?
*(d1) Make a time series plot (i.e. type = "l") of the 5-minute interval (x-axis) and average* 
*no. of steps taken, averaged across all days (y-axis)*

```r
totStepsByInterval <- aggregate(actDF$steps, list(actDF$interval), mean, na.rm = TRUE)
names(totStepsByInterval) <- c("Interval", "Avg.Steps")
plot(totStepsByInterval$Interval, totStepsByInterval$Avg.Steps, 
     type = "l", 
     main = "Avg. No. of Steps by 5-Minute Interval", 
     xlab = "5-Minute Interval (HHMM)", 
     ylab = "Avg. No. of Steps")
```

![](C:\Users\Tejus Madusker\Desktop\R-Coursera\Practice Code\output\PA1_template_files/figure-html/Plot.Steps.By.Interval-1.png)<!-- -->

*(d2) Which 5-minute interval, on average across all days, contains the max no. of steps?*

```r
maxAvg <- max(totStepsByInterval$Avg.Steps)
maxIndex <- which(totStepsByInterval$Avg.Steps > as.integer(maxAvg))
whichInterval <- totStepsByInterval[maxIndex, ]
whichInterval$Interval
```

```
## [1] 835
```

##(e) Imputing missing values
There are a no. of days/intervals with missing values (coded as NA). The presence of missing days 
may introduce bias into some calculations or summaries of the data.
*(e1) Calculate and report the total no. of missing values (i.e. total no. of rows with NAs)*

```r
sum(is.na(actDF$steps))
```

```
## [1] 2304
```

*(e2) Devise a strategy for filling in all missing values. E.g., use the mean/median for that day,* 
*or the mean for that 5-minute interval, etc.*
Strategy for (e2): 
Since there are some ENTIRE days with missing values (e.g., Oct 1/8, Nov 1/4/9/10/14/30), imputing 
with daily averages may not work well since the dataset only covers two months (i.e., there is not
enough data to calculate "good" daily averages for each calendar day of the month). Therefore, it 
may be better to impute NAs with average values for those specific intervals where the NAs occur.

*(e3) Create a new dataset that is equal to original dataset but with missing data filled in*

```r
meanStepsByInterval <- aggregate(actDF$steps, list(actDF$interval), mean, na.rm = TRUE)
names(meanStepsByInterval) <- c("Interval", "Mean.Steps")
imputedDF <- actDF
naIndex <- which(is.na(imputedDF$steps))
for(i in naIndex){
    interval <- imputedDF[i, 3]
    intervalAvg <- meanStepsByInterval[meanStepsByInterval$Interval == interval, 2]
    imputedDF[i, 1] <- intervalAvg
}
```

*(e4) Make a histogram of total no. of steps taken each day and calculate and report the mean and* 
*median total no. of steps taken per day.*
*Do these values differ from the estimates from the first part of the assignment?*
*What is the impact of imputing missing data on the estimates of total daily number of steps?*

Answer to (e4): 
Here's the historgram, mean and median for the **orginal, unimputed dataframe**:

```r
totStepsByDay <- aggregate(actDF$steps, list(actDF$date), sum)
names(totStepsByDay) <- c("Date", "Tot.Steps")
totStepsByDay
```

```
##          Date Tot.Steps
## 1  2012-10-01        NA
## 2  2012-10-02       126
## 3  2012-10-03     11352
## 4  2012-10-04     12116
## 5  2012-10-05     13294
## 6  2012-10-06     15420
## 7  2012-10-07     11015
## 8  2012-10-08        NA
## 9  2012-10-09     12811
## 10 2012-10-10      9900
## 11 2012-10-11     10304
## 12 2012-10-12     17382
## 13 2012-10-13     12426
## 14 2012-10-14     15098
## 15 2012-10-15     10139
## 16 2012-10-16     15084
## 17 2012-10-17     13452
## 18 2012-10-18     10056
## 19 2012-10-19     11829
## 20 2012-10-20     10395
## 21 2012-10-21      8821
## 22 2012-10-22     13460
## 23 2012-10-23      8918
## 24 2012-10-24      8355
## 25 2012-10-25      2492
## 26 2012-10-26      6778
## 27 2012-10-27     10119
## 28 2012-10-28     11458
## 29 2012-10-29      5018
## 30 2012-10-30      9819
## 31 2012-10-31     15414
## 32 2012-11-01        NA
## 33 2012-11-02     10600
## 34 2012-11-03     10571
## 35 2012-11-04        NA
## 36 2012-11-05     10439
## 37 2012-11-06      8334
## 38 2012-11-07     12883
## 39 2012-11-08      3219
## 40 2012-11-09        NA
## 41 2012-11-10        NA
## 42 2012-11-11     12608
## 43 2012-11-12     10765
## 44 2012-11-13      7336
## 45 2012-11-14        NA
## 46 2012-11-15        41
## 47 2012-11-16      5441
## 48 2012-11-17     14339
## 49 2012-11-18     15110
## 50 2012-11-19      8841
## 51 2012-11-20      4472
## 52 2012-11-21     12787
## 53 2012-11-22     20427
## 54 2012-11-23     21194
## 55 2012-11-24     14478
## 56 2012-11-25     11834
## 57 2012-11-26     11162
## 58 2012-11-27     13646
## 59 2012-11-28     10183
## 60 2012-11-29      7047
## 61 2012-11-30        NA
```

```r
hist(totStepsByDay$Tot.Steps, 
     col="green", 
     main = "Histogram of Total Steps by Day", 
     xlab = "Total Daily Steps")
```

![](C:\Users\Tejus Madusker\Desktop\R-Coursera\Practice Code\output\PA1_template_files/figure-html/Histogram.Original-1.png)<!-- -->

```r
summary(totStepsByDay$Tot.Steps)["Mean"]
```

```
##     Mean 
## 10766.19
```

```r
summary(totStepsByDay$Tot.Steps)["Median"]
```

```
## Median 
##  10765
```

And here's the histogram, mean and median **post imputation (filling NAs with interval averages)**:

```r
imputedTotStepsByDay <- aggregate(imputedDF$steps, list(imputedDF$date), sum)
names(imputedTotStepsByDay) <- c("Date", "Tot.Steps")
hist(imputedTotStepsByDay$Tot.Steps, 
     col="blue", 
     main = "Histogram of Total Steps by Day - Post Imputation", 
     xlab = "Total Daily Steps")
```

![](C:\Users\Tejus Madusker\Desktop\R-Coursera\Practice Code\output\PA1_template_files/figure-html/Histogram.New-1.png)<!-- -->

```r
summary(imputedTotStepsByDay$Tot.Steps)["Mean"]
```

```
##     Mean 
## 10766.19
```

```r
summary(imputedTotStepsByDay$Tot.Steps)["Median"]
```

```
##   Median 
## 10766.19
```

Clearly, the mean did not change after imputing, and the median changed very slightly, almost 
negligibly, in fact. Thus, imputing worked great!

##(f) Are there differences in activity patterns between weekdays and weekends?
For this part weekdays() function may be of help. Use the dataset with filled-in missing values

*(f1) Create a new factor variable in the dataset with two levels - "weekday" and "weekend"*
*indicating whether a given date is a weekday or weekend day.*

```r
imputedDF$Day.of.Week <- weekdays(as.Date(as.character(imputedDF$date)))
imputedDF$Day.of.Week <- factor(imputedDF$Day.of.Week)
for(i in 1:nrow(imputedDF)){
    if(imputedDF$Day.of.Week[i] %in% c("Saturday", "Sunday")){imputedDF$Day.Week[i] <- "Weekend"
    } else {imputedDF$Day.Week[i] <- "Weekday"}
}
imputedDF$Day.Week <- factor(imputedDF$Day.Week)
str(imputedDF)
```

```
## 'data.frame':	17568 obs. of  5 variables:
##  $ steps      : num  1.717 0.3396 0.1321 0.1509 0.0755 ...
##  $ date       : Factor w/ 61 levels "2012-10-01","2012-10-02",..: 1 1 1 1 1 1 1 1 1 1 ...
##  $ interval   : int  0 5 10 15 20 25 30 35 40 45 ...
##  $ Day.of.Week: Factor w/ 7 levels "Friday","Monday",..: 2 2 2 2 2 2 2 2 2 2 ...
##  $ Day.Week   : Factor w/ 2 levels "Weekday","Weekend": 1 1 1 1 1 1 1 1 1 1 ...
```

*(f2) Make a panel plot containing a time series plot (type="l") of the 5-minute interval (x-axis)* 
*and average no. of steps taken, averaged across all weekday days or weekend days (y-axis).* 
*See the README file in the GitHub repository for an example of what this plot should look like* 
*using simulated data.*

```r
library(lattice)
xyplot(steps ~ interval | Day.Week, 
       data = imputedDF, 
       main = "Avg. Steps by Interval: Weekdays vs. Weekends", 
       xlab = "Interval", ylab = "Number of Steps", 
       type = "l", 
       layout = c(1,2))
```

![](C:\Users\Tejus Madusker\Desktop\R-Coursera\Practice Code\output\PA1_template_files/figure-html/Panel.Plot-1.png)<!-- -->

# End of assignment solution
