---
title: "Activity Analysis (Final Submission)"
author: "Konrad Zdeb"
date: "19th July 2014"
output: html_document
---

This is an R Markdown document. The Markdown document fulfils the requirements 
of the assignment.

# Getting the Data

## Sourcing the data

The code below checks whether the required data files exists in the working directory and if the file doesn't exists it attempts to source the the zip file and unzip it. The Last bits of code remove objects that were created for the purpose of getting the data. If the CSV file is available in the working directory there is no need to run the code in this section and the one can progress to the section on reading data.


```r
# I'm using the RCurl package to download the file if not saved in the directory
# The code below will install it if not available
if (!require("RCurl",character.only = TRUE))
    {
      install.packages("RCurl",dep=TRUE)
        if(!require("RCurl",character.only = TRUE)) stop("Package not found. 
                                                          Please make sure that 
                                                          you have the Lattice
                                                          package installed")
    }
```

```
## Loading required package: RCurl
## Loading required package: bitops
```

```r
# Now i check whether the data exists and download if it doesn't exists
if (!file.exists("activity.csv")) {
        # Check wthether the zip archive is available.
        if (!file.exists("repdata_data_activity.zip")) {
                # Define data source.
                dataurl <- "https://d396qusza40orc.cloudfront.net/repdata%2Fdata%2Factivity.zip"  
                # Download, save and unzip the file.
                library(RCurl)
                bin <- getBinaryURL(dataurl, ssl.verifypeer=FALSE, 
                                    noprogress = FALSE)
                con <- file("repdata_data_activity.zip",
                            open = "wb")
                writeBin(bin, con)
                close(con)
                unzip("repdata_data_activity.zip")
        } else
                unzip("repdata_data_activity.zip")
}
file <- list.files(path=getwd(), pattern = "activity.csv", all.files = TRUE,
                    recursive = TRUE, full.names = TRUE)
# Clean redundant objects after download
if (exists("bin")) rm(bin)
if (exists("con")) rm(con)
if (exists("dataurl")) rm(dataurl)
```
## Reading Data

The code reads and prepares the data. The date column is translated to the R date format.


```r
# Read the file
act.dta <- read.csv(file = file)
#Change date to the R date format
act.dta$date <- as.Date(act.dta$date, "%Y-%m-%d") 
```

# Analysis
The sections here correspond to the questions in the assignment with each section answering one of the requests. It assumed that the master data is available in the act.dta file, if the data is not available or object is renamed the code below can be run to create the master data file with the required file.

## Setps per Day
The section is concerned with answering the first question *What is mean total number of steps taken per day?*

### Histogram
The code below generates the requested histogram. I added nice curve to make it more interesting.


```r
# Create values with sum of steps per day
act.dta.sum <- aggregate(x = act.dta[c("steps")], FUN = sum, 
                         by = list(date = act.dta$date))
# Create histogram
stepshist <- hist(x = act.dta.sum$steps, plot = FALSE)
# Create nice normal curve
multiplier <- stepshist$counts / stepshist$density
stpsdnst <- density(act.dta.sum$steps, na.rm = TRUE, adjust = 2)
stpsdnst$y <- stpsdnst$y * multiplier[1]
# Plot both graphics
plot(stepshist, main = "Histogram: Steps", xlab = "Steps", col="lightgreen")
lines(stpsdnst, col="darkblue", lwd=2)
```

![plot of chunk histogram](https://raw.githubusercontent.com/konradedgar/RepData_PeerAssessment1/master/blob/master/figure/histogram.png) 

### Mean and Median Steps
The code below calculates and reports mean and median steps using the summary data for the steps. Following [the discussion](https://class.coursera.org/repdata-004/forum/thread?thread_id=25) on the course forum the utilised syntax assumes that we are obtaining one mean and one one median for the total steps per day.


```r
# Force R not to use scientific notation, and keep two decimal places.
options("scipen"=100, "digits"=4)
options(digits=2)
# Obtain mean
mean(act.dta.sum$steps, na.rm = TRUE)
```

```
## [1] 10766
```

```r
# Obtain median
median(act.dta.sum$steps, na.rm = TRUE)
```

```
## [1] 10765
```

#### Answer
The mean value is **10766.19** and the median value is **10765**.

## Average Daily Activity Pattern
This section is concerned with answering the questions on the average daily activity patterns. 

### 5-min interval plot
The time series plot below corresponds to the 5 minute interval and uses the *interval* variable provided in the data set.


```r
# Create summary data
act.dta.avg <- aggregate(x = act.dta[c("steps")], FUN = mean, 
                         by = list(interval = act.dta$interval), 
                         na.rm = TRUE)
# Make the plot
plot(x = act.dta.avg$interval, y = act.dta.avg$steps, type = "l", lwd = "2",
     col="darkblue", main = "Mean Steps across Interval (5min)", 
     col.main = "black", font.main=2,
     ylab = "Mean Steps", xlab = "5 Min Interval")
```

![plot of chunk line.plot](https://raw.githubusercontent.com/konradedgar/RepData_PeerAssessment1/master/blob/master/figure/line.plot.png) 

### Maximum steps
The code below identifies interval with the maximum number of steps.


```r
# Return maximum steps
max(act.dta.avg$steps)
```

```
## [1] 206
```

```r
# Find and display the row.
m.row <- which.max(x = act.dta.avg$steps)
m.row
```

```
## [1] 104
```

```r
# Return the corresponding interval value
act.dta.avg$interval[m.row]
```

```
## [1] 835
```

#### Answer
The maximum value for the mean steps is **206.17** and corresponds to **835** value of the 5-minute interval.

## Inputing Missing Values
This section deals with the problem of imputing missing values. 

### Total Number of Missing Values
The code below counts the total number of missing values and total number of missing values per day.

```r
# Missing values per column
sapply(act.dta, function(x) sum(is.na(x)))
```

```
##    steps     date interval 
##     2304        0        0
```

#### Answer
The total number of missing values in the data is **2304**.

### Inputing Missing Values
The code below imputes missing values using the group mean. 


```r
# Make copy of the data
act.dta.nomiss <- act.dta
# Go each row of the data and impute with an corresponding average if figures missing
i <- 1
for (i in 1:length(act.dta.nomiss$steps)) {
        if(is.na(act.dta.nomiss[i,1])) {
                # Replace with corresponding value form the avg data set
                act.dta.nomiss[i,1] <- act.dta.avg[which(
                        act.dta.nomiss[i,3] == act.dta.avg$interval),2]
        }
}
# Check whether the loop worked fine and there is no missing data.
sapply(act.dta.nomiss, function(x) sum(is.na(x)))
```

```
##    steps     date interval 
##        0        0        0
```

### Creating Histogram and Summary
The section deals with creating the summaries on the new data set. The section is very similar to the one described in the Steps per Day section.


```r
# Aggregate new data set with no missing values
act.dta.nomiss.sum <- aggregate(x = act.dta.nomiss[c("steps")], FUN = sum, 
                         by = list(date = act.dta.nomiss$date))
# Same as previously, create histogram on new data.
stepshist.nomiss <- hist(x = act.dta.nomiss.sum$steps, plot = FALSE)
# Create nice normal curve
multiplier.nomiss <- stepshist.nomiss$counts / stepshist.nomiss$density
stpsdnst.nomiss <- density(act.dta.nomiss.sum$steps, adjust = 2)
stpsdnst.nomiss$y <- stpsdnst.nomiss$y * multiplier.nomiss[1]
# Plot both graphics
plot(stepshist.nomiss, main = "Histogram: Steps \n Missing data was removed with use of the Amelia II package", xlab = "Steps", 
     col="lightblue", cex.main = 0.8)
lines(stpsdnst, col="red", lwd=2)
```

![plot of chunk new.histogram](https://raw.githubusercontent.com/konradedgar/RepData_PeerAssessment1/master/blob/master/figure/new.histogram.png) 

```r
# Obtain mean and median on the new data
# Obtain mean
mean(act.dta.nomiss.sum$steps)
```

```
## [1] 10766
```

```r
# Obtain median
median(act.dta.nomiss.sum$steps)
```

```
## [1] 10766
```

#### Answer 
After imputing missing data the mean value for the total steps is **10766.19** and the median value is **10766.19**. The mean value for the new data is **0** different than the value obtained from the data with missing values, which was 10766.19. The new median value is **1.19** different from the median value derived from the data set with the missing values, which was 10765. 

## Weekdays and Weekends
This section looks at the differences in activity patterns between weekdays and weekends.

### Factor variable
The code below generates factor variable for weekday (0) and weekend (1).

```r
# Factor variable with value 1 for weekend and 0 for weekday is created
act.dta.nomiss$weekend <- weekdays(x = act.dta.nomiss$date) %in% c('Sunday','Saturday')
```

### Panel Plot
The code below generates the panel plot with two lines one for weekends and the other for weekdays.

```r
# Creates average data set for interval and accounting for the weekend variable.
act.dta.nomiss.avg <- aggregate(x = act.dta.nomiss[c("steps")], FUN = mean, 
                         by = list(interval = act.dta.nomiss$interval, 
                                   weekend = act.dta.nomiss$weekend), 
                         na.rm = TRUE)
# Check if the lattice package is available and if it isn't try to get it
if (!require("lattice",character.only = TRUE))
    {
      install.packages("lattice",dep=TRUE)
        if(!require("lattice",character.only = TRUE)) stop("Package not found. 
                                                          Please make sure that 
                                                          you have the Lattice
                                                          package installed")
    }
```

```
## Loading required package: lattice
```

```r
# This piece of code only defines some colours to use in the plot.
plot.cols <- list(
  superpose.polygon=list(col="blue", border="transparent"),
  strip.background=list(col="lightblue"),
  strip.border=list(col="black"))
# Draw the plot as requested
xyplot( steps ~ interval | weekend, data=act.dta.nomiss.avg, type='l', 
       layout = c(1, 2), lwd=2, strip=strip.custom(factor.levels=c("Weekend",
                                                                   "Weekday")),
       main = list(label = "Mean Activity by Interval Over Weekdays and Weeknds",
                   cex = 1.2, col = "darkblue"),
       ylab = list(label = "Steps", col ="darkblue", cex = 1.1),
       xlab = list(label = "Interval", col ="darkblue", cex = 1.1),
       scales=list(alternating=1), 
       par.settings = plot.cols)
```

![plot of chunk days.plot](https://raw.githubusercontent.com/konradedgar/RepData_PeerAssessment1/master/blob/master/figure/days.plot.png) 
