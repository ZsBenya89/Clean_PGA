---
title: "Markdown for run_analysis.r"
author: "Bence Zsámboki"
date: '2018 január 30 '
output: html_document
---

# Instructions
## Getting and Cleaning Data Course Project 
> The purpose of this project is to demonstrate your ability to collect, work with, and clean a data set. The goal is to prepare tidy data that can be used for later analysis. You will be graded by your peers on a series of yes/no questions related to the project. You will be required to submit: 1) a tidy data set as described below, 2) a link to a Github repository with your script for performing the analysis, and 3) a code book that describes the variables, the data, and any transformations or work that you performed to clean up the data called CodeBook.md. You should also include a README.md in the repo with your scripts. This repo explains how all of the scripts work and how they are connected.

> One of the most exciting areas in all of data science right now is wearable computing - see for example this article . Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:  
> http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones  

> Here are the data for the project:  

> https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip


> You should create one R script called run_analysis.R that does the following.
>
1. Merges the training and the test sets to create one data set.
+Done
2. Extracts only the measurements on the mean and standard deviation for each measurement.
+Done
3. Uses descriptive activity names to name the activities in the data set
+Done
4. Appropriately labels the data set with descriptive variable names.
+Done
5. From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.
+Done
>Let's get it started!

# Preps
## Loading the packages


```r
packages <- c("data.table", "reshape2", "knitr", "memisc", "markdown")
sapply(packages, library, character.only = T, quietly = T, logical.return = T, warn.conflicts = F)
```

```
## data.table   reshape2      knitr     memisc   markdown 
##       TRUE       TRUE       TRUE       TRUE       TRUE
```

## Set the path


```r
path <- getwd()
print(path)
```

```
## [1] "C:/Users/Nobody/Documents/testdir/Clean_PGA"
```

#Getting and loading the data

> Download the data and put these into the data folder (also check if it is completed)


```r
URL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
File <- "Dataset.zip"
if (!file.exists(path)) {dir.create(path)}
if (!file.exists(File)) {download.file(URL, file.path(path, File))}
```

> Unzip files (also check if it is completed)


```r
datalib <- "UCI HAR Dataset"
if (!file.exists(datalib)) {unzip(zipfile = file.path(path, File))}
```

> Set the data folder (UCI HAR Dataset), and check the files


```r
datalib2 <- file.path(path, datalib)
list.files(datalib2, recursive = T)
```

```
##  [1] "activity_labels.txt"                         
##  [2] "features.txt"                                
##  [3] "features_info.txt"                           
##  [4] "README.txt"                                  
##  [5] "test/Inertial Signals/body_acc_x_test.txt"   
##  [6] "test/Inertial Signals/body_acc_y_test.txt"   
##  [7] "test/Inertial Signals/body_acc_z_test.txt"   
##  [8] "test/Inertial Signals/body_gyro_x_test.txt"  
##  [9] "test/Inertial Signals/body_gyro_y_test.txt"  
## [10] "test/Inertial Signals/body_gyro_z_test.txt"  
## [11] "test/Inertial Signals/total_acc_x_test.txt"  
## [12] "test/Inertial Signals/total_acc_y_test.txt"  
## [13] "test/Inertial Signals/total_acc_z_test.txt"  
## [14] "test/subject_test.txt"                       
## [15] "test/X_test.txt"                             
## [16] "test/y_test.txt"                             
## [17] "train/Inertial Signals/body_acc_x_train.txt" 
## [18] "train/Inertial Signals/body_acc_y_train.txt" 
## [19] "train/Inertial Signals/body_acc_z_train.txt" 
## [20] "train/Inertial Signals/body_gyro_x_train.txt"
## [21] "train/Inertial Signals/body_gyro_y_train.txt"
## [22] "train/Inertial Signals/body_gyro_z_train.txt"
## [23] "train/Inertial Signals/total_acc_x_train.txt"
## [24] "train/Inertial Signals/total_acc_y_train.txt"
## [25] "train/Inertial Signals/total_acc_z_train.txt"
## [26] "train/subject_train.txt"                     
## [27] "train/X_train.txt"                           
## [28] "train/y_train.txt"
```


> Loading the data into R

>Read activity files


```r
dataacttrain <- fread(file.path(datalib2, "train", "Y_train.txt"))
dataacttest <- fread(file.path(datalib2, "test", "Y_test.txt"))
```

>Read subject files


```r
datasubtrain <- fread(file.path(datalib2, "train", "subject_train.txt"))
datasubtest <- fread(file.path(datalib2, "test", "subject_test.txt"))
```

>Loading data files


```r
datatrain <- fread(file.path(datalib2, "train", "X_train.txt"))
datatest <- fread(file.path(datalib2, "test", "X_test.txt"))
```

#Merging datasets

> Connect datatables into one table


```r
datasub <- rbind(datasubtest, datasubtrain)
setnames(datasub, "V1", "subject")
dataact <- rbind(dataacttest, dataacttrain)
setnames(dataact, "V1", "activityNumb")
data <- rbind(datatest,datatrain)
data <- cbind(datasub, dataact, data)
```

> Set the key


```r
setkey(data, subject, activityNumb)
```

#Extracts the mean and st deviation

>First read the feature file and select the mean and st deviation codes. With these codes choose the appropirate mean and st dev. values.

>Read feature file.


```r
datafeature <- fread(file.path(datalib2, "features.txt"))
setnames(datafeature, names(datafeature), c("featureNumb", "featureName"))
```

>Select the mean and st dev codes


```r
datafeature <- datafeature[grepl("mean\\(\\)|std\\(\\)", featureName)]
```

> Insert V in front of the code to match with the data file


```r
datafeature$featureCode <- datafeature[, paste0("V", featureNumb)]
head(datafeature)
```

```
##    featureNumb       featureName featureCode
## 1:           1 tBodyAcc-mean()-X          V1
## 2:           2 tBodyAcc-mean()-Y          V2
## 3:           3 tBodyAcc-mean()-Z          V3
## 4:           4  tBodyAcc-std()-X          V4
## 5:           5  tBodyAcc-std()-Y          V5
## 6:           6  tBodyAcc-std()-Z          V6
```

> Print the codes


```r
datafeature$featureCode
```

```
##  [1] "V1"   "V2"   "V3"   "V4"   "V5"   "V6"   "V41"  "V42"  "V43"  "V44" 
## [11] "V45"  "V46"  "V81"  "V82"  "V83"  "V84"  "V85"  "V86"  "V121" "V122"
## [21] "V123" "V124" "V125" "V126" "V161" "V162" "V163" "V164" "V165" "V166"
## [31] "V201" "V202" "V214" "V215" "V227" "V228" "V240" "V241" "V253" "V254"
## [41] "V266" "V267" "V268" "V269" "V270" "V271" "V345" "V346" "V347" "V348"
## [51] "V349" "V350" "V424" "V425" "V426" "V427" "V428" "V429" "V503" "V504"
## [61] "V516" "V517" "V529" "V530" "V542" "V543"
```

>Select mean and st dev values from data file


```r
select <- c(key(data), datafeature$featureCode)
data <- data[, select, with = F]
```

#Uses descriptive activity names to name the activities in the data set

>Read the "activity_labels.txt" to use it as descriptive activity names


```r
dataactnames <- fread(file.path(datalib2, "activity_labels.txt"))
setnames(dataactnames, names(dataactnames), c("activityNumb", "activityName"))
```

#ppropriately labels the data set with descriptive variable names.

>Merge activity labels


```r
data <- merge(data, dataactnames, by = "activityNumb", all.x = T)
```

>Set the keys


```r
setkey(data, subject, activityNumb, activityName)
```

>Reshape the table.


```r
data <- data.table(melt(data, key(data), variable.name = "featureCode"))
```

>Merging activity names


```r
data <- merge(data, datafeature[, list(featureNumb, featureCode, featureName)], by = "featureCode", all.x = T)
```

>Creating factor class variables for "activityName" and "featureName"


```r
data$activity <- factor(data$activityName)
data$feature <-  factor(data$featureName)
```

>Seperate features from "featureName" using the helper function "grepthis".



```r
grepthis <- function(regex) {
    grepl(regex, data$feature)
}
## Features with 2 categories
n <- 2
y <- matrix(seq(1, n), nrow = n)
x <- matrix(c(grepthis("^t"), grepthis("^f")), ncol = nrow(y))
data$featDomain <- factor(x %*% y, labels = c("Time", "Freq"))
x <- matrix(c(grepthis("Acc"), grepthis("Gyro")), ncol = nrow(y))
data$featInstrument <- factor(x %*% y, labels = c("Accelerometer", "Gyroscope"))
x <- matrix(c(grepthis("BodyAcc"), grepthis("GravityAcc")), ncol = nrow(y))
data$featAcceleration <- factor(x %*% y, labels = c(NA, "Body", "Gravity"))
x <- matrix(c(grepthis("mean()"), grepthis("std()")), ncol = nrow(y))
data$featVariable <- factor(x %*% y, labels = c("Mean", "SD"))
## Features with 1 category
data$featJerk <- factor(grepthis("Jerk"), labels = c(NA, "Jerk"))
data$featMagnitude <- factor(grepthis("Mag"), labels = c(NA, "Magnitude"))
## Features with 3 categories
n <- 3
y <- matrix(seq(1, n), nrow = n)
x <- matrix(c(grepthis("-X"), grepthis("-Y"), grepthis("-Z")), ncol = nrow(y))
data$featAxis <- factor(x %*% y, labels = c(NA, "X", "Y", "Z"))
```


>Check to make sure all possible combinations of "feature" are accounted for by all possible combinations of the factor class variables.



```r
r1 <- nrow(data[, .N, by = c("feature")])
r2 <- nrow(data[, .N, by = c("featDomain", "featAcceleration", "featInstrument", 
    "featJerk", "featMagnitude", "featVariable", "featAxis")])
r1 == r2
```

```
## [1] TRUE
```


>Yes, I accounted for all possible combinations. "feature" is now redundant.

#Create a tidy dataset


```r
setkey(data, subject, activity, featDomain, featAcceleration, featInstrument, 
    featJerk, featMagnitude, featVariable, featAxis)
dataTidy <- data[, list(count = .N, average = mean(value)), by = key(data)]
```

>Make a codebook


```r
Write(codebook(dataTidy), file = "Codebook.md")
markdownToHTML("Codebook.md", "codebook.html")
```

> Write the output file


```r
write.table(dataTidy, "Tidydataset.txt", row.names = F)
```
