install.packages("reshape2")
library(reshape2)
install.packages("downloader")
library(downloader)

setwd("C:/Users/Alexandre/Desktop/R_directory")
zip_name <- "getdata_dataset.zip"

# Download and unzip the dataset:
if (!file.exists(zip_name)){
  fileURL <- "https://d396qusza40orc.cloudfront.net/getdata%2Fprojectfiles%2FUCI%20HAR%20Dataset.zip"
  download.file(fileURL, zip_name)
  unzip(zip_name)
}  

# Load activity labels features
activityLabels <- read.table("UCI HAR Dataset/activity_labels.txt")
activityLabels
activityLabels[,2] <- as.character(activityLabels[,2])
activityLabels[,2]

# Load activity features
features <- read.table("UCI HAR Dataset/features.txt")
features
features[,2] <- as.character(features[,2])
features[,2]

# Extract only the data on mean and standard deviation
featuresWanted <- grep(".*mean.*|.*std.*", features[,2])
featuresWanted
featuresWanted.names <- features[featuresWanted,2]
featuresWanted.names = gsub('-mean', 'Mean', featuresWanted.names)
featuresWanted.names = gsub('-std', 'Std', featuresWanted.names)
featuresWanted.names <- gsub('[-()]', '', featuresWanted.names)


# Load the datasets
train <- read.table("UCI HAR Dataset/train/X_train.txt")[featuresWanted]
train
trainActivities <- read.table("UCI HAR Dataset/train/Y_train.txt")
trainActivities
trainSubjects <- read.table("UCI HAR Dataset/train/subject_train.txt")
train <- cbind(trainSubjects, trainActivities, train)
# result
head(train,1)


test <- read.table("UCI HAR Dataset/test/X_test.txt")[featuresWanted]
testActivities <- read.table("UCI HAR Dataset/test/Y_test.txt")
testSubjects <- read.table("UCI HAR Dataset/test/subject_test.txt")
test <- cbind(testSubjects, testActivities, test)
# result
head(test,1)

# merge datasets and add labels

allData <- rbind(train, test)
colnames(allData) <- c("subject", "activity", featuresWanted.names)
# result
head(allData,1)

# turn activities & subjects into factors
allData$activity <- factor(allData$activity, levels = activityLabels[,1], labels = activityLabels[,2])
allData$subject <- as.factor(allData$subject)

allData.melted <- melt(allData, id = c("subject", "activity"))
allData.melted

allData.mean <- dcast(allData.melted, subject + activity ~ variable, mean)
allData.mean

#store the tidy file
write.table(allData.mean, "tidy.txt", row.names = FALSE, quote = FALSE)

