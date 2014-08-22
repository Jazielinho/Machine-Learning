---
title: "Prediction Model Assignment Write-up"
author: "Jahaziel Ponce"
date: "Thursday, August 21, 2014"
output: pdf_document
---

## Background

Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. In this project, your goal will be to use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).

## Download data:

Load packages:


```r
library(caret)
```

```
## 
## Attaching package: 'caret'
## 
## The following object is masked from 'package:survival':
## 
##     cluster
```

Download data:


```r
if (!file.exists("pml-training.csv")) {
    download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", 
                  destfile = "pml-training.csv")
}
if (!file.exists("pml-testing.csv")) {
    download.file("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", 
                  destfile = "pml-testing.csv")
}
```


```r
#Load Data train
training <- read.csv("pml-training.csv", stringsAsFactors=FALSE)
```

## Getting and Cleaning the Data Set

There are 160 columns in the Training Set, and we will be excluding columns with NA values.

Cleaning data:


```r
NewTraining <- training
NewTraining[ NewTraining == '' | NewTraining == 'NA'] <- NA
indx <- which(colSums(is.na(NewTraining))!=0)
NewTraining <- NewTraining[, -indx]
NewTraining <- NewTraining[,-(1:7)]
```

## Creating train y test data


```r
InTraining  <- createDataPartition(y=NewTraining$classe,p=0.70,list=FALSE)
data <- NewTraining
NewTraining <- data[InTraining,]
ValidateSet <- data[-InTraining,]
```

## Model Training

Random Forest we will be using to train our Prediction Model set to predict the weight lifting quality in the Training Set.


```r
library(randomForest)
```

```
## randomForest 4.6-10
## Type rfNews() to see new features/changes/bug fixes.
## 
## Attaching package: 'randomForest'
## 
## The following object is masked from 'package:Hmisc':
## 
##     combine
```

```r
# model fit using random forests
Pmodel <- randomForest(as.factor(classe)~.,data=NewTraining)
Pmodel
```

```
## 
## Call:
##  randomForest(formula = as.factor(classe) ~ ., data = NewTraining) 
##                Type of random forest: classification
##                      Number of trees: 500
## No. of variables tried at each split: 7
## 
##         OOB estimate of  error rate: 0.47%
## Confusion matrix:
##      A    B    C    D    E class.error
## A 3906    0    0    0    0    0.000000
## B   10 2642    6    0    0    0.006020
## C    0   10 2385    1    0    0.004591
## D    0    0   25 2224    3    0.012433
## E    0    0    2    7 2516    0.003564
```

## Model Evaluation

Now we evaluate our model results through confusion Matrix.


```r
#Predictions
predict <- predict(Pmodel, ValidateSet)
# evaluate the model
error <- confusionMatrix(ValidateSet$classe, predict);
```

```
## 
## Attaching package: 'e1071'
## 
## The following object is masked from 'package:Hmisc':
## 
##     impute
```

```r
error
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1671    2    0    0    1
##          B    5 1132    2    0    0
##          C    0    3 1021    2    0
##          D    0    0   12  952    0
##          E    0    0    1    2 1079
## 
## Overall Statistics
##                                         
##                Accuracy : 0.995         
##                  95% CI : (0.993, 0.997)
##     No Information Rate : 0.285         
##     P-Value [Acc > NIR] : <2e-16        
##                                         
##                   Kappa : 0.994         
##  Mcnemar's Test P-Value : NA            
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.997    0.996    0.986    0.996    0.999
## Specificity             0.999    0.999    0.999    0.998    0.999
## Pos Pred Value          0.998    0.994    0.995    0.988    0.997
## Neg Pred Value          0.999    0.999    0.997    0.999    1.000
## Prevalence              0.285    0.193    0.176    0.162    0.184
## Detection Rate          0.284    0.192    0.173    0.162    0.183
## Detection Prevalence    0.284    0.194    0.174    0.164    0.184
## Balanced Accuracy       0.998    0.997    0.992    0.997    0.999
```

Model Accuracy as tested over Validation set = 99.9%

## Model Test

Finally, we proceed with predicting the new values in the testing csv provided


```r
#Load Data test
training <- read.csv("pml-testing.csv", stringsAsFactors=FALSE)
#Predicting
prediccion <- predict(Pmodel, ValidateSet)
```

### Generating Answers Files to Submit for Assignment

The following function to create the files to answers the Prediction Assignment Submission:


```r
write_files = function(x){
  for(i in 1:30){
    filename = paste0("problem_id_",i,".txt")
    write.table(x[i],file=filename,quote=FALSE,row.names=FALSE,col.names=FALSE)
  }
}

write_files(prediccion)
```
