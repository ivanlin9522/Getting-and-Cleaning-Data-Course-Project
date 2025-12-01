This script processes the **Human Activity Recognition (HAR) Using Smartphones Dataset** and creates a tidy dataset ready for analysis.

Here is the README and a detailed explanation of how the script works.

-----

# 📖 README: Getting and Cleaning Data Project

## 🎯 Overview

This repository contains the R script `run_analysis.R` (representing the provided code) which fulfills the requirements of the Coursera **"Getting and Cleaning Data"** course project. The script downloads, cleans, and transforms the **Human Activity Recognition (HAR) Using Smartphones Dataset** into a **tidy data set**.

## 🚀 Script Goals

The R script performs the following five main steps, as required by the project:

1.  **Merges** the training and the test sets to create one data set.
2.  **Extracts** only the measurements on the mean and standard deviation for each measurement.
3.  **Uses descriptive activity names** to name the activities in the data set.
4.  **Appropriately labels** the data set with descriptive variable names.
5.  **Creates a second, independent tidy data set** with the average of each variable for each activity and each subject.

## Prerequisites

  * **R** statistical computing environment.
  * **RStudio** (recommended IDE).
  * The following R packages must be installed: **`data.table`** and **`reshape2`**.

You can install them using:

```r
install.packages(c("data.table", "reshape2"))
```

## ⚙️ How to Run the Script

1.  Place the provided R code (e.g., save it as `run_analysis.R`) in your R working directory.
2.  Open R or RStudio.
3.  Set your working directory (if necessary): `setwd("/path/to/your/directory")`
4.  Source the script:
    ```r
    source("run_analysis.R")
    ```
5.  The script will automatically download the data, process it, and create the final tidy data file named **`tidyData.txt`** in your working directory.

-----

## 🔬 How the Script Works (Code Breakdown)

The script is executed in several logical blocks, primarily using functions from the `data.table` and `reshape2` packages for efficiency and ease of manipulation.

### 1\. Setup and Data Download

The script starts by loading necessary packages and downloading the data:

```r
packages <- c("data.table", "reshape2")
sapply(packages, require, character.only=TRUE, quietly=TRUE)
# ... code to set path, download, and unzip data ...
```

  * It ensures **`data.table`** and **`reshape2`** are loaded.
  * It downloads the zipped dataset from the specified URL and unzips it, creating the `UCI HAR Dataset` folder structure.

### 2\. Feature and Activity Label Loading (Steps 2 & 4 - Prep)

This section prepares the labels for features and activities.

```r
# Load activity labels + features
activityLabels <- fread(...)
features <- fread(...)
featuresWanted <- grep("(mean|std)\\(\\)", features[, featureNames])
measurements <- features[featuresWanted, featureNames]
measurements <- gsub('[()]', '', measurements) 
```

  * It loads the **activity names** (`activity_labels.txt`) and **feature names** (`features.txt`).
  * It finds the indices of features containing either `mean()` or `std()` using **`grep()`**. This fulfills **Step 2 (Extracts mean and std measurements)**.
  * It cleans the feature names by removing parentheses `()` using **`gsub()`**. This contributes to **Step 4 (Appropriately labels)**.

### 3\. Load, Filter, and Merge Training Data

The training data is processed and column-bound (`cbind`) with its activity and subject identifiers.

```r
# Load train datasets
train <- fread(file.path(path, "UCI HAR Dataset/train/X_train.txt"))[, featuresWanted, with = FALSE]
data.table::setnames(train, colnames(train), measurements) # Step 4
trainActivities <- fread(...)
trainSubjects <- fread(...)
train <- cbind(trainSubjects, trainActivities, train) 
```

  * It reads the main training data (`X_train.txt`), but only selects the columns (`featuresWanted`) identified in the previous step.
  * The descriptive variable names (`measurements`) are applied to the columns (**Step 4**).
  * The **Subject Number** and **Activity ID** columns are added to the left of the main data table.

### 4\. Load, Filter, and Merge Test Data

The same process is repeated for the test data.

```r
# Load test datasets
test <- fread(file.path(path, "UCI HAR Dataset/test/X_test.txt"))[, featuresWanted, with = FALSE]
data.table::setnames(test, colnames(test), measurements) # Step 4
# ... code to load testActivities and testSubjects ...
test <- cbind(testSubjects, testActivities, test) 
```

### 5\. Combine Data Sets and Add Descriptive Activity Names

The training and test sets are combined, fulfilling **Step 1**.

```r
# merge datasets (Step 1)
combined <- rbind(train, test)

# Convert classLabels to activityName (Step 3)
combined[["Activity"]] <- factor(combined[, Activity]
                              , levels = activityLabels[["classLabels"]]
                              , labels = activityLabels[["activityName"]])
combined[["SubjectNum"]] <- as.factor(combined[, SubjectNum])
```

  * **`rbind(train, test)`** stacks the rows of the test data onto the training data.
  * The **Activity ID** column (which contains numbers 1-6) is converted to a **factor** and mapped to the descriptive names (e.g., WALKING, SITTING) using the `activityLabels` data. This fulfills **Step 3 (Uses descriptive activity names)**.
  * The **SubjectNum** is also converted to a factor.

### 6\. Create the Tidy Data Set (Step 5)

This section calculates the average for every variable, grouped by subject and activity.

```r
combined <- reshape2::melt(data = combined, id = c("SubjectNum", "Activity"))
combined <- reshape2::dcast(data = combined, SubjectNum + Activity ~ variable, fun.aggregate = mean)
```

  * **`melt()`**: This function from `reshape2` converts the data from a **wide** format (many measurement columns) to a **long** format. It uses `SubjectNum` and `Activity` as the identifier columns (`id`).
  * **`dcast()`**: This function then aggregates the data and converts it back to a **wide** format. It groups the data by `SubjectNum` and `Activity` and applies the **`mean`** function to every measurement variable (`variable`). This fulfills **Step 5**.

### 7\. Output

The final tidy data set is saved to a file.

```r
data.table::fwrite(x = combined, file = "tidyData.txt", quote = FALSE)
```

  * The final result is written to `tidyData.txt`, ensuring it is an independent tidy data set.
