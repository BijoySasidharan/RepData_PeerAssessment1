---

#         Reproducible Research-Course Project

----

#### Author: Bijoy Sasidharan

----

## 1.0 Loading and preprocessing the data


----

#### 1.1 Setting up the necessary libraries




First step in this project is to setup the necessary libraries which includes
[**knitr**](https://cran.r-project.org/web/packages/knitr/index.html) (for generating single file markdown file in html/latex/pdf format), 
[**markdown**](https://cran.r-project.org/web/packages/markdown/index.html) (provides R bindings to the Sundown markdown rendering library) , 
[**ggplot**](http://ggplot2.org/) (for ploting the data),
[**HMisc**](http://http://www.inside-r.org/packages/cran/hmisc/docs/impute) (for using impute function)

```{libraries}
library(knitr)

library(data.table)

library(ggplot2)

library(HMisc)
```

----

#### 1.2 About the dataset



The data used in this analysis is generated from a personal activity monitoring device.This device collects data at 5 minute intervals through out the day. The data consists of two months of data from an anonymous individual collected during the months of October and November, 2012 and include the number of steps taken in 5 minute intervals each day.

----

#### 1.3 Load the dataset




For loading the dataset the following code is used.Since the dataset is in an extract form,it need to be uniziped before reading it for importing.

```{Loading}
rrdata <- read.csv(unz("activity.zip", "activity.csv"))
```

----

#### 1.4 Format the data




```{formatting}
rrdata$date <- as.Date(rrdata$date, format = "%Y-%m-%d")
rrdata$interval <- as.factor(rrdata$interval)
```

----

## 2.0 What is mean total number of steps taken per day?

----

#### 2.1 Calculate the total steps taken per day



```{steps}
stepsperDay<- aggregate(steps ~ date, rrdata, sum)
View(stepsperDay)
```

----

#### 2.2 Plot the histogram (Steps per day vs Frequency in a day)



```{histogram}
ggplot(stepsperDay, aes(x = steps)) + 
       geom_histogram(fill = "lightblue", binwidth = 1000) + 
        labs(title="Histogram of Steps Taken per Day", 
             x = "Steps Taken per Day", y = "Frequency in a day") + theme_grey() 
```            


![alt text](https://raw.githubusercontent.com/BijoySasidharan/RepData_PeerAssessment1/master/figure/Image_1_Hist.png)

----

#### 2.3 Calculate the mean and median



```  {Mean,Median}
stepsMean   <- mean(stepsperDay$steps, na.rm=TRUE)
stepsMedian <- median(stepsperDay$steps, na.rm=TRUE)
```  

----

#### 2.4 Result


* Mean : **`10766.19`**
* Median :  **`10765`**


----

## 3.0 What is the average daily activity pattern?

----


#### 3.1 Calculate the mean steps taken per 5-minutes interval

An intermediate table is created with the mean steps taken for each 5-minutes interval time period.

```  {activity}
dailypattern <- aggregate(x=list(meanSteps=rrdata$steps), by=list(interval=rrdata$interval), FUN=mean, na.rm=TRUE)
```  

----

#### 3.2 Make a time series plot (5-minute Intervals vs Average Steps)



  
```{histogram}
ggplot(dailypattern, aes(y = meanSteps , x=interval)) + 
       geom_line(color = "lightgreen", size = 1) + 
        labs(title="Average Daily Activity Pattern", 
             x = "5-minute Intervals ", y = "Average Steps") + theme_bw() 
```    

![alt text](https://raw.githubusercontent.com/BijoySasidharan/RepData_PeerAssessment1/master/figure/Image_2_Line.png)

----

## 4.0 Imputing missing values

----

#### 4.1 Calculate and report the total number of missing values in the dataset



**is.na()** method is used to check the missing criteria in the steps field.
Then its summed into a logical vector named as "missing".

```{missing}
missing <- sum(is.na(rrdata$steps))
```


Number of missing values : **2304**

----

#### 4.2 Strategy for filling in all of the missing values in the dataset


Strategy is to fill in the missing values with mean values for 5-minute intervals across days.

----

#### 4.3 Create a new dataset that is equal to the original dataset but with the missing data filled in.




```{filling}
rrdatafilled <- rrdata
rrdatafilled$steps <- impute(rrdata$steps, fun=mean)
```

----

#### 4.4.0 Make a histogram of the total number of steps taken each day with filled values.



##### 4.4.1 Calculate the total steps taken per day with filled data


```{stepsfilled}
stepsperDayfilled<- aggregate(steps ~ date, rrdatafilled, sum)
View(stepsperDayfilled)
```

----

##### 4.4.2 Plot the histogram with filled data(Steps per day vs Frequency in a day)




```{histogram}
ggplot(stepsperDayfilled, aes(x = steps)) + 
       geom_histogram(fill = "lightblue", binwidth = 1000) + 
        labs(title="Histogram of Steps Taken per Day (Filled Data)", 
             x = "Steps Taken per Day", y = "Frequency in a day") + theme_grey() 
```            


![alt text](https://raw.githubusercontent.com/BijoySasidharan/RepData_PeerAssessment1/master/figure/Image_3_Hist_filled.png)

----

#### 4.5.0 Calculate and report the mean and median total number of steps taken per day after filling the missing values.





##### 4.5.1 Calculate the mean and median



```  {Mean&Median}
stepsMeanfilled   <- mean(stepsperDayfilled$steps, na.rm=TRUE)
stepsMedianfilled <- median(stepsperDayfilled$steps, na.rm=TRUE)
```  



##### 4.5.2 Result

* Mean(Filled) : **`10766.19`**
* Median(Filled) :  **`10766.19`**


----

#### 4.6 Do these values differ from the estimates from the first part of the assignment? 


Mean value remained the same wheras the Median value increased after filling the missing values.The mean and median values are equal now after filling the missing values.


----

#### 4.7 What is the impact of imputing missing data on the estimates of the total daily number of steps?


* Mean and median values became equal.

* Peak value of the histogram increased.

----


## 5.0 Are there differences in activity patterns between weekdays and weekends?

----

#### 5.1 Create a new factor variable in the dataset with two levels – “weekday” and “weekend” indicating whether a given date is a weekday or weekend day.



````{Weekdayorweekend}
rrdatafilled$dateType <-  ifelse(as.POSIXlt(rrdatafilled$date)$wday %in% c(0,6), 'weekend', 'weekday')
````


----

#### 5.2 Make a panel plot containing a time series plot



````{Plot-weekday/weekend}
averagerrdatafilled <- aggregate(steps ~ interval + dateType, data=rrdatafilled, mean)

ggplot(averagerrdatafilled, aes(interval, steps)) + 
    geom_line() + 
    facet_grid(dateType ~ .) +
    xlab("5-minute interval") + 
    ylab("Number of steps")
````


![alt text](https://raw.githubusercontent.com/BijoySasidharan/RepData_PeerAssessment1/master/figure/Image_4_Double.png)
----
#### 5.3 Inference

From the plot we can infer that there is obvious difference in activity pattern between weekdays and weekends. We can infer that the user had taken more steps during the Weekends compared to the weekdays.Also from both graphs, we can infer that the peak periods of activity (for both weekdays and weekends)is between 7:30 and 10:00 in the morning.


