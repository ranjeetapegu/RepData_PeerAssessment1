Introduction
------------

It is now possible to collect a large amount of data about personal
movement using activity monitoring devices such as a Fitbit, Nike
Fuelband, or Jawbone Up.In this assignment,we will try assignment to
makes use of data from a personal activity monitoring device and try
answering few questions. The device collects data at 5 minute intervals
through out the day. The data consists of two months of data from an
anonymous individual collected during the months of October and
November, 2012 and include the number of steps taken in 5 minute
intervals each day.

### Loading and preprocessing the data

We will begin with the analysis by downloading the data from
[*here*](%22https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip%22).
Unzip the zip file and extract the data from the csv file into a dataset
named activity, readable in R.

    # set the working directory (optional) 
    setwd("/Users/ranjeetapegu/Documents/Coursera/Reproducible Result/week 2")
    if(!file.exists("./RepAct")){dir.create("./RepAct")}
    Dwurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"
    download.file(url =Dwurl, destfile = "./RepAct/RepAct.zip", mode ="wb")
    unzip("./RepAct/RepAct.zip", exdir = "./RepAct")
    activity <- read.csv("./RepAct/activity.csv", header = TRUE, sep =",")
    head(activity)

    ##   steps       date interval
    ## 1    NA 2012-10-01        0
    ## 2    NA 2012-10-01        5
    ## 3    NA 2012-10-01       10
    ## 4    NA 2012-10-01       15
    ## 5    NA 2012-10-01       20
    ## 6    NA 2012-10-01       25

**What is mean total number of steps taken per day?**  
Lets find out the mean of total number of steps taken per day, for
this,the missing values in the dataset will be ignored.The total number
of steps taken per day is calculated by taking the sum of steps group by
the date. A histogram of the total number of steps taken each day is
plotted.

    Daily.step <- aggregate(activity$steps, by = list(activity$date),FUN = sum, na.rm = T)
    names(Daily.step) <- c("date", "steps")
    # Make a histogram of the total number steps taken
    hist(Daily.step$steps, main ="Histogram of total nos of step taken each day",xlab ="Total steps per day", ylab= "Fequency",col = "green")

![](https://github.com/ranjeetapegu/RepData_PeerAssessment1/blob/master/Hist_DailyTotalSteps.png)

Lets find out the mean and median of the total number of steps taken per
day by using the summary function.

    summary(Daily.step$steps)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##       0    6778   10400    9354   12810   21190

**What is the average daily activity pattern?**  
To find the average daily activity pattern, we will make a time series
plot of the 5-minute interval (x-axis) and the average number of steps
taken, averaged across all days (y-axis).The red vertical line in the
plot shows the 5-minute interval (i.e. 835) which contains the maximum
number of steps.

    Avgstep.timeIn <- aggregate(activity$steps, by = list(activity$interval),FUN = mean, na.rm = T)
    names(Avgstep.timeIn) <- c("Interval","Steps")
    maxstep <- Avgstep.timeIn[Avgstep.timeIn$Steps == max(Avgstep.timeIn$Steps),c(1) ]
    plot( Avgstep.timeIn$Interval, Avgstep.timeIn$Steps , type ="l" , col ="blue" , 
          main ="Time Series Of Interval Vs Average Steps",
          xlab="Time Interval of 5 mins" ,
          ylab = "Average Steps")
    abline(v = maxstep, lty= 3, col ="red")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-4-1.png)

**Imputing missing values**

There are a number of days/intervals where there are missing values
coded as NA, this may introduce some bias into some calculations and
summaries of the data. Lets get the total number of missing values in
the dataset (i.e. the total number of rows with 𝙽𝙰s)

     summary(activity$steps)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max.    NA's 
    ##    0.00    0.00    0.00   37.38   12.00  806.00    2304

We see there is a total of 2304 missig values. We will fill the missing
value by mean for that 5 min interval for all days and make a new
dataset names New.Activity.

    Avgstep.timeIn <- aggregate(activity$steps, by = list(activity$interval), FUN = mean, na.rm = T)
    names(Avgstep.timeIn) <- c("interval","AvgSteps")
    New.Activity <- merge(activity, Avgstep.timeIn, by ="interval")
    New.Activity$steps[is.na(New.Activity$steps)] <- New.Activity$AvgSteps[is.na(New.Activity$steps)]
    New.Activity$steps <-round( New.Activity$steps)

Lets plot a histogram of the total number of steps taken each day using
the new dataset, to see the difference between the estimates from the
new dataset and old dataset.

    NDaily.step <- aggregate(New.Activity$steps, by = list(New.Activity$date),FUN = sum,  na.rm = T)
    names(NDaily.step) <- c("date", "steps")
    hist(NDaily.step$steps, main ="Histogram of total nos of step taken each day",xlab ="Total steps per day", ylab= "Number of times",col = "Blue")

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-7-1.png)

Summary of new dataset will give whether all missing have been replaced
and also give the new mean and median of total number of steps taken in
a day.

    summary(New.Activity$steps)

    ##    Min. 1st Qu.  Median    Mean 3rd Qu.    Max. 
    ##    0.00    0.00    0.00   37.38   27.00  806.00

Do these values differ from the estimates from the first part of the
assignment?  
After comparing the summaries, it is clear that there is no change in
mean, median , min and max number of steps each day after fillin the
missing values. But the comparision of graphs shows there is an increase
in number of days when the person has reached his/her maximum steps.

**Are there differences in activity patterns between weekdays and
weekends?**  
Using the dataset with the filled-in missing values , we create a new
factor variable day, two levels – “weekday” and “weekend” indicating
whether a given date is a weekday or weekend day. plot charts having
time series plot of the 5-minute interval (x-axis) and the average
number of steps taken, averaged across all weekday days or weekend days
(y-axis).

    New.Activity$d <- weekdays(as.Date(New.Activity$date))
    New.Activity$day <-  as.factor(ifelse(New.Activity$d %in% c("Saturday","Sunday"), "Weekend","Weekday") )
    head(New.Activity)

    ##   interval steps       date AvgSteps        d     day
    ## 1        0     2 2012-10-01 1.716981   Monday Weekday
    ## 2        0     0 2012-11-23 1.716981   Friday Weekday
    ## 3        0     0 2012-10-28 1.716981   Sunday Weekend
    ## 4        0     0 2012-11-06 1.716981  Tuesday Weekday
    ## 5        0     0 2012-11-24 1.716981 Saturday Weekend
    ## 6        0     0 2012-11-15 1.716981 Thursday Weekday

    Int_Avgstep <- aggregate(New.Activity$steps, by = list(New.Activity$interval,
                            New.Activity$day),FUN = mean, na.rm = T)
    names(Int_Avgstep) <- c("interval","day","steps")
    Int_Avgstep$steps <- round(Int_Avgstep$steps)
    library(ggplot2)
    ggplot(Int_Avgstep, aes(interval, steps )) + geom_line(colour = "blue") +
           xlab("interval ") + ylab ("Avg steps") + facet_grid(facets = day ~ .)

![](PA1_template_files/figure-markdown_strict/unnamed-chunk-9-1.png)

From the above charts we conclude that the average number of steps taken
in weekdays is significantly higher in time interval between 750 -1000,
whereas for weekends we did not observe such high peak during the same
interval period.
