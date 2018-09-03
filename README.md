# Project-assignment-Getting-and-cleaning-Data
This file contains the script "run_analysis.R" which is submitted as part of the project assignment for course 3: Getting and Cleaning data

# run_analysis.R script:

library(data.table)
library(dplyr)


## Setting the working directory and downloading the dataset

setwd("C:/Users/boodeo1/Desktop/Coursera/data")

## fileUrl
fileUrl <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"

## Downloading the file
download.file(fileUrl, destfile = "./data/HARdata.zip")

## Uncompress the zip file
unzip(zipfile = "./data/HARdata.zip", exdir = "./data")

## Files unzipped to folder "UCI HAR Dataset"
# Reading the "features" and "activity" files

featureNames <- read.table("./data/UCI HAR Dataset/features.txt", header = FALSE)
activityLabels <- read.table("./data/UCI HAR Dataset/activity_labels.txt", header = FALSE)

# Reading Training data
subjectTrain <- read.table("./data/UCI HAR Dataset/train/subject_train.txt", header = FALSE)
activityTrain <- read.table("./data/UCI HAR Dataset/train/y_train.txt", header = FALSE)
featuresTrain <- read.table("./data/UCI HAR Dataset/train/X_train.txt", header = FALSE)

# Reading the Test data
subjectTest <- read.table("./data/UCI HAR Dataset/test/subject_test.txt", header = FALSE)
activityTest <- read.table("./data/UCI HAR Dataset/test/y_test.txt", header = FALSE)
featuresTest <- read.table("./data/UCI HAR Dataset/test/X_test.txt", header = FALSE)

## 1. Merging the training and test data
subject <- rbind(subjectTrain, subjectTest)
activity <- rbind(activityTrain, activityTest)
features <- rbind(featuresTrain, featuresTest)

# Naming and merging the columns
colnames(features) <- t(featureNames[2])
colnames(activity) <- "Activity"
colnames(subject) <- "Subject"
completeData <- cbind(features,activity,subject)

## 2. Extracting the meand and std deviation of each measurement
columnsWithMeanSTD <- grep(".*Mean.*|.*Std.*", names(completeData), ignore.case=TRUE)
requiredColumns <- c(columnsWithMeanSTD, 562, 563)

extractedData <- completeData[,requiredColumns]

## 3. Using descriptive names to name activities in the dataset
# Changing the type of the "activity" and "extractedData" from numeric to character so that activity names can be accepted
extractedData$Activity <- as.character(extractedData$Activity)
for (i in 1:6){
extractedData$Activity[extractedData$Activity == i] <- as.character(activityLabels[i,2])
}

extractedData$Activity <- as.factor(extractedData$Activity)

## 4. Labelling the dataset with descriptive variable names
names(extractedData) <- gsub("Acc", "Accelerometer", names(extractedData))
names(extractedData) <- gsub("Gyro", "Gyroscope", names(extractedData))
names(extractedData) <- gsub("BodyBody", "Body", names(extractedData))
names(extractedData) <- gsub("Mag", "Magnitude", names(extractedData))
names(extractedData) <- gsub("^t", "Time", names(extractedData))
names(extractedData) <- gsub("^f", "Frequency", names(extractedData))
names(extractedData) <- gsub("tBody", "TimeBody", names(extractedData))
names(extractedData) <- gsub("-mean()", "Mean", names(extractedData), ignore.case = TRUE)
names(extractedData) <- gsub("-std()", "STD Deviation", names(extractedData), ignore.case = TRUE)
names(extractedData) <- gsub("-freq()", "Frequency", names(extractedData), ignore.case = TRUE)

## 5. Creating a tidy dataset
# Setting "subject" as a factor variable
extractedData$Subject <- as.factor(extractedData$Subject)
extractedData <- data.table(extractedData)

# creating tidyData as a dataset with average values for each variable and ordering the tidyData starting with "subject" followed by "activity" and generating the tidy dataset as "tidyData.txt"
tidyData <- aggregate(.~Subject + Activity, extractedData, mean)
tidyData <- tidyData[order(tidyData$Subject,tidyData$Activity),]
write.table(tidyData, file = "tidyData.txt", row.names = FALSE)
