---
title: "Getting and Cleaning Data – Final Assignment"
author: "Siddharth Samant"
date: "4/29/2020"
output:
  html_document:
    toc: yes
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```



=======================================================================
  

  
  
## The Assignment^1^  


One of the most exciting areas in all of data science right now is wearable computing - see for example this article. Companies like Fitbit, Nike, and Jawbone Up are racing to develop the most advanced algorithms to attract new users. The data linked to from the course website represent data collected from the accelerometers from the Samsung Galaxy S smartphone. A full description is available at the site where the data was obtained:

http://archive.ics.uci.edu/ml/datasets/Human+Activity+Recognition+Using+Smartphones

Here are the data for the project:

https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip


You should create one R script called run_analysis.R that does the following:
1.	Merges the training and the test sets to create one data set.

2.	Extracts only the measurements on the mean and standard deviation for each measurement.

3.	Uses descriptive activity names to name the activities in the data set.

4.	Appropriately labels the data set with descriptive variable names.

5.	From the data set in step 4, creates a second, independent tidy data set with the average of each variable for each activity and each subject.  


=======================================================================


## Understanding The Original Dataset^2^  


The experiments have been carried out with a group of 30 volunteers within an age bracket of 19-48 years. Each person performed six activities (WALKING, WALKING_UPSTAIRS, WALKING_DOWNSTAIRS, SITTING, STANDING, LAYING) wearing a smartphone (Samsung Galaxy S II) on the waist. Using its embedded accelerometer and gyroscope, we captured 3-axial linear acceleration and 3-axial angular velocity at a constant rate of 50Hz. The experiments have been video-recorded to label the data manually. The obtained dataset has been randomly partitioned into two sets, where 70% of the volunteers was selected for generating the training data and 30% the test data. 

The sensor signals (accelerometer and gyroscope) were pre-processed by applying noise filters and then sampled in fixed-width sliding windows of 2.56 sec and 50% overlap (128 readings/window). The sensor acceleration signal, which has gravitational and body motion components, was separated using a Butterworth low-pass filter into body acceleration and gravity. The gravitational force is assumed to have only low frequency components, therefore a filter with 0.3 Hz cutoff frequency was used. From each window, a vector of features was obtained by calculating variables from the time and frequency domain. See 'features_info.txt' for more details. 

For each record it is provided:

- Triaxial acceleration from the accelerometer (total acceleration) and the estimated body acceleration.
- Triaxial Angular velocity from the gyroscope. 
- A 561-feature vector with time and frequency domain variables. 
- Its activity label. 
- An identifier of the subject who carried out the experiment.

The dataset includes the following files:
- 'README.txt'
- 'features_info.txt': Shows information about the variables used on the feature vector.
- 'features.txt': List of all features.
- 'activity_labels.txt': Links the class labels with their activity name.
- 'train/X_train.txt': Training set.
- 'train/y_train.txt': Training labels.
- 'test/X_test.txt': Test set.
- 'test/y_test.txt': Test labels.

The following files are available for the train and test data. Their descriptions are equivalent:
*    train/subject_train.txt: Each row identifies the subject who performed the activity for each window sample. Its range is from 1 to 30. 
*    train/Inertial Signals/total_acc_x_train.txt: The acceleration signal from the smartphone accelerometer X axis in standard gravity units 'g'. Every row shows a 128 element vector. The same description applies for the 'total_acc_x_train.txt' and 'total_acc_z_train.txt' files for the Y and Z axis. 
*    train/Inertial Signals/body_acc_x_train.txt: The body acceleration signal obtained by subtracting the gravity from the total acceleration. *    train/Inertial Signals/body_gyro_x_train.txt: The angular velocity vector measured by the gyroscope for each window sample. The units are radians/second. 

Notes: 

- Features are normalized and bounded within [-1,1].
- Each feature vector is a row on the text file.  


=======================================================================


## The Script  


```{r run_analysis, eval = FALSE}

###Installing and Loading the "tidyverse" set of packages, plus "stringr", and "lubridate" packages

#install.packages("tidyverse") #Remove "#" if not already installed
library(tidyverse)
#install.packages("stringr") #Remove "#" if not already installed
library(stringr)
#install.packages("lubridate") #Remove "#" if not already installed
library(lubridate)



###Downloading the zip file, and initial working on the files inside the zipfile

#Downloading the zip file and unzipping the contents in my working directory
URL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
download.file(URL, destfile = "UCI_HAR.zip")
unzip("UCI_HAR.zip", files = NULL, list = FALSE, overwrite = TRUE,
      junkpaths = FALSE, exdir = ".", unzip = "internal",
      setTimes = FALSE)



#Reading the FEATURES file
features <- read_delim("./UCI HAR Dataset/features.txt", delim = " ",
                       col_names = FALSE)
#Adding suitable column names
colnames(features) <- c("featureID", "featureName")
#Making the observations of variable "featureName" unique, since there are some repeating observations
features$featureName <- make.names(features$featureName, unique = TRUE) #***STEP 4***#



#Reading the ACTIVITY_LABELS file
activity_labels <- read_delim("./UCI HAR Dataset/activity_labels.txt", delim = " ",
                       col_names = FALSE)
#Adding suitable column names
colnames(activity_labels) <- c("activityID", "activityName")
#Changing the observations in "activityName" variable to a fixed format
activity_labels$activityName <- tolower(activity_labels$activityName)
activity_labels$activityName <- sub("_u"," u", activity_labels$activityName)
activity_labels$activityName <- sub("_d"," d", activity_labels$activityName)
activity_labels$activityName <- sub("$"," while wearing a smartphone on the waist", 
                                    activity_labels$activityName) #***STEP 3 P1***#



#Reading the X_TEST file
x_test <- read_delim("./UCI HAR Dataset/test/X_test.txt", delim = " ",
           col_names = FALSE)
#Removing white spaces
x_test <- apply(x_test, 2, str_trim)
x_test <- as.data.frame(x_test)
#Using "apply" function to change variable type from Character to Double
x_test <- apply(x_test, 2, as.double)
x_test <- as.data.frame(x_test)
#Changing colNames of all variables to match the features that they are capturing 
colnames(x_test) <- features$featureName #***STEP 3 P2***#



#Reading the Y_TEST file
y_test <- read_delim("./UCI HAR Dataset/test/y_test.txt", delim = " ",
                     col_names = FALSE)
#Adding a column name to Y_TEST
colnames(y_test) <- c("activityID")
#Merging Y_TEST with ACTIVITY_LABELS to add "activityName" variable to Y_TEST 
y_test <- y_test %>%
   left_join(activity_labels, by = "activityID")



#Reading the SUBJECT_TEST file
subject_test <- read_delim("./UCI HAR Dataset/test/subject_test.txt", delim = " ",
                     col_names = FALSE)
colnames(subject_test) <- c("subjectID")



#Reading the X_TRAIN file
x_train <- read_delim("./UCI HAR Dataset/train/X_train.txt", delim = " ",
                     col_names = FALSE)
#Removing white spaces
x_train <- apply(x_train, 2, str_trim)
x_train <- as.data.frame(x_train)
#Using "apply" function to change variable type from Character to Double
x_train <- apply(x_train, 2, as.double)
x_train <- as.data.frame(x_train)
#Changing colNames of all variables to match the features that they are capturing 
colnames(x_train) <- features$featureName #***STEP 3 P3***#



#Reading the Y_TRAIN file
y_train <- read_delim("./UCI HAR Dataset/train/y_train.txt", delim = " ",
                     col_names = FALSE)
colnames(y_train) <- c("activityID")
#Merging Y_TRAIN with ACTIVITY_LABELS to add "activityName" variable to Y_TRAIN
y_train <- y_train %>%
   left_join(activity_labels, by = "activityID")



#Reading the SUBJECT_TRAIN file
subject_train <- read_delim("./UCI HAR Dataset/train/subject_train.txt", delim = " ",
                           col_names = FALSE)
colnames(subject_train) <- c("subjectID")



###Computations on X_TEST

#Removing !(mean) & !(standard deviation) columns from X_TEST and X_TRAIN
x_test <- x_test %>%
   select(contains (".mean.") | contains ("std")) #***STEP 2 P1***#
#Adding Y_TEST to X_TEST
x_test <- cbind(y_test, x_test)
#Adding SUBJECT_TEST to X_TEST
x_test <- cbind(subject_test, x_test)
#Adding "group" identifier on X_TEST
x_test$group <- "testing"
#Reordering variables so that "group" comes first
x_test <- x_test[,c(70, 1:69)]



###Computations on X_TRAIN

#Removing !(mean) & !(standard deviation) columns from X_TEST and X_TRAIN
x_train <- x_train %>%
   select(contains (".mean.") | contains ("std")) #***STEP 2 P2***#
#Adding Y_TRAIN to X_TRAIN
x_train <- cbind(y_train, x_train)
#Adding SUBJECT_TRAIN to X_TRAIN
x_train <- cbind(subject_train, x_train)
#Adding "group" identifier on X_TRAIN
x_train$group <- "training"
#Reordering variables so that "group" comes first
x_train <- x_train[,c(70, 1:69)]



###Combining X_TEST and X_TRAIN

#Creating the new dataset SMARTPHONERECORDS
smartphone_records <- rbind(x_test, x_train) #***STEP 1***#
#Arranging SMARTPHONERECORDS by variable "subjectID" followed by "activityID"
smartphone_records <- smartphone_records %>%
   arrange(subjectID, activityID)



###Making the final dataset

#Creating the new dataset SMARTPHONERECORDSAVERAGE
smartphone_records_average <- smartphone_records %>%
   group_by(subjectID, activityID, group, activityName) %>%
   summarise_all(funs(mean), na.rm = TRUE) #***STEP 5***#

#Writing the final dataset into my working directory
write_csv(smartphone_records_average, "Smartphone Records Average.csv")
write_delim(smartphone_records_average, "Smartphone Records Average.txt")



###Appendix: In case you want to read the saved file SMARTPHONERECORDSAVERAGE, run the below code
#sraNew <- read.table("Smartphone Records Average.txt", sep = " ", header = TRUE)
#view(sraNew)
```
  

=======================================================================


## How the Script Works  


The following steps exhaustively detail the R Script file **run_analysis.R**:

1. I loaded the following packages:
    *	tidyverse
    *	stringr
    *	lubridate
    
2. I downloaded the  [zip file](https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip), and unzipped it in my Working Directory. The file was saved as **UCI HAR Dataset**, and it contains two sub-directories – “test” and “train”.

3. I read the “features” text file from the main directory of the dataset, saving it as the data frame “features”. 
    *	I added column names to the data frame.
    *	I used the make.names() function to make the observations of the variable “featureName” unique. These observations will provide the column names for our final dataset. 
    
4. I read the “activity_labels” text file from the main directory of the dataset, saving it as the data frame “activity_labels”.
    *	I added column names to the data frame.
    *	I used the tolower() and sub() functions to make the observations of the variable “activityName” descriptive. These observations will form a part of the final dataset.
    
5.	I read the “X_test” text file from the “test” sub-directory of the dataset, saving it as the data frame “x_test”.
    *	I used the apply() function to remove white spaces, and to convert the variable type for all variables to “double”.
    *	I added column names to the data frame – these were the observations from the “featureName” variable of the “features” data frame 
    
6.	I read the “y_test” text file from the “test” sub-directory of the dataset, saving it as the data frame “y_test”.
    *	I added column names to the data frame.
    *	I used the left_join() function to merge “y_test” with “activity_labels”, using the variable “activityID” as the key
    
7.	I read the “subject_test” text file from the “test” sub-directory of the dataset, saving it as the data frame “subject_test”.
    *	I added column names to the data frame.
    
8.	I read the “X_train” text file from the “train” sub-directory of the dataset, saving it as the data frame “x_train”.
    *	I used the apply() function to remove white spaces, and to convert the variable type for all variables to “double”.
    *	I added column names to the data frame – these were the observations from the “featureName” variable of the “features” data frame 
    
9.	I read the “y_train” text file from the “train” sub-directory of the dataset, saving it as the data frame “y_train”.
    *	I added column names to the data frame.
    *	I used the left_join() function to merge “y_train” with “activity_labels”, using the variable “activityID” as the key
    
10.	I read the “subject_train” text file from the “train” sub-directory of the dataset, saving it as the data frame “subject_train”.
    *	I added column names to the data frame.
    
11.	I made the following computations on the “x_test” data frame:
    *	I removed column names that did not compute either mean or standard deviation – I used the select() function to do this.
    *	I added the columns of “y_test” data frame to “x_test” using cbind() function. I saved the results in “x_test”.
    *	I added the columns of “subject_test” data frame to “x_test” using cbind() function. I saved the results in “x_test”.
    *	I added a variable named “group” to “x_test”. I gave the common value “testing” to each observation in this variable. 
    *	I re-ordered the variables of “x_test” so that the “order” variable came first.
    
12.	I made the following computations on the “x_train” data frame:
    *	I removed column names that did not compute either mean or standard deviation – I used the select() function to do this.
    *	I added the columns of “y_train” data frame to “x_train” using cbind() function. I saved the results in “x_train”.
    *	I added the columns of “subject_train” data frame to “x_train” using cbind() function. I saved the results in “x_train”.
    *	I added a variable named “group” to “x_train”. I gave the common value “training” to each observation in this variable. 
    *	I re-ordered the variables of “x_train” so that the “order” variable came first.
    
13.	I merged the “x_test” and “x_train” data frames into a new data frame called “smartphone_records” using rbind(). I arranged the observations in “smartphone_records” by the variables “subjectID” and “activityID”. 

14.	I created the final independent Tidy Dataset “smartphone_records_average” by doing the final computations:
    *	I grouped the table by “subjectID” and “activityID” using group_by() function
    *	I used the summarise() function to take the average value for all the mean and standard deviation variables
    
15.	I wrote the results of the final independent Tidy Dataset into a csv file and a text file, using the “write_csv” and “write_delim” functions respectively. The name of both files is “Smartphone Records Average”, and they are saved in my working directory.  


=======================================================================


## Making a Tidy Data Set – Justifications and Clarifications  


1.	The final dataset is tidy, because it follows the three most important principles of a Tidy Dataset^3^:
    +	Each variable has its own column.
    +	Each observation has its own row.
    +	Each value has its own cell.
    
2.	There is a sub-sub-directory called “Inertial Signals” inside both the “test” and “train” sub-directories. I have completely disregarded that data for the below reasons:
    +	No column names are provided for the datasets inside “Inertial Signals”, and there is no guide provided in the raw data files for determining which features from the “features” dataset apply as column names to those datasets, and in what order.
    +	Even if we add the columns to the “x_test” and “x_train” datasets with default column names, they will be removed before merging the datasets into “smartphone_records”, since it is not possible to determine if any of those columns contain data calculating the mean or the standard deviation.

3.  I have chosen to make little modifications to the column names of the final dataset – I have only used the make.names() function to make them unique. I will rely on an extensive explanation of each part of each column in the Code Book in order to help readers interpret the column names better.

4.	I have kept the names of the objects the same as they appear in the UCI HAR Dataset, except for converting them to lower-case. This has been done so that the end user can connect the objects to the files in the raw dataset. The notation used is “snake_case” – lower-case words separated by an underscore (ex. x_train, smartphone_records).  


=======================================================================


## The Final Dataset - A Partial Look  



```{r kable, echo=FALSE, message=FALSE, warning=FALSE}
library(tidyverse)
library(stringr)

smartphone_records_average <- read_csv("Smartphone Records Average.csv")

knitr::kable(
smartphone_records_average[1:20, 1:6],
caption = "smartphone_records_average"
)
```

  


=======================================================================


## References  

[1] Getting and Cleaning Data, John Hopkins University, Coursera.org, final assignment description.

[2] Davide Anguita, Alessandro Ghio, Luca Oneto, Xavier Parra and Jorge L. Reyes-Ortiz. Human Activity Recognition on Smartphones using a Multiclass Hardware-Friendly Support Vector Machine. International Workshop of Ambient Assisted Living (IWAAL 2012). Vitoria-Gasteiz, Spain. Dec 2012.
  
[3] R for Data Science, Hadley Wickham and Garrett Grolemund.  


