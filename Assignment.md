---
title: "Predicting barbell quality on body accelerometer measurements"
author: "Jordi ORDONEZ"
date: "4 july of 2016"
output: html_document
---


```r
knitr::opts_chunk$set(echo = TRUE)
knitr::opts_chunk$set(cache=TRUE)
library(caret)
```

```
## Warning: package 'caret' was built under R version 3.2.5
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```
## Warning: package 'ggplot2' was built under R version 3.2.4
```

```r
library(rpart)
library(randomForest)
```

```
## randomForest 4.6-12
```

```
## Type rfNews() to see new features/changes/bug fixes.
```

```
## 
## Attaching package: 'randomForest'
```

```
## The following object is masked from 'package:ggplot2':
## 
##     margin
```

```r
training<-read.csv("./pml-training.csv")
testing<-read.csv("./pml-testing.csv")
```

## Summary

This work is about building a prediction model based on body's censor movement measurements when performing barbell for detecting the way it was performed. Informatuion about the way data was collected is avalaible at <http://groupware.les.inf.puc-rio.br/har>.
You can also download data to perform the analysis :

* [The training set](http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv)
* [The testing set](http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv)

After preprocessing the data in order to allow algorithm selection on it, different prediction models will be tested with cross validation before, and when the best one will be selected, we'll give a meausre of the error applying it on the testing set.


## Preprocessing the data

After downloading our data, we make a partition of our training set to obtain a test set (80% training set, 20% for our test set) and we repartition the training set the same way for the later cross-validation :


```r
training<-read.csv("./pml-training.csv",na.strings=c("NA","#DIV/0!", ""))
testing<-read.csv("./pml-testing.csv",na.strings=c("NA","#DIV/0!", ""))
set.seed(666)
inTrain <- createDataPartition(y=training$classe, p=0.8, list=FALSE)
train <- training[inTrain, ]; test <- training[-inTrain, ]
inTrain <- createDataPartition(y=train$classe, p=0.8, list=FALSE)
train <- train[inTrain, ]; testcv <- train[-inTrain, ]
```

After running str on training we see a lot of NA's  and some variables not interesting for predicting classe such as X, user_name, time, window variables.
We try to clean our data, the NA's, and some variables not interesting for predicting the variable classe wich contains the information on how barbell was performed, we get rid of variables that are nearzerovariance variables and those containing more than 50% of NA's:


```r
train<-train[,-c(1:7)]
NZV <- nearZeroVar(train, saveMetrics=TRUE)
train<-train[!NZV$nzv]
nas<-apply(train,2,function(x) sum(is.na(x))/length(x)>.5)
train<-train[!nas]
dim(train)
```

```
## [1] 12562    53
```

So we reduced the number of variables from 160 to 53, we'll apply the same reduction to our test set for cross validation : our classe variable is still here! 


```r
testcv<-testcv[colnames(train)]
```


## Comparing prediction models : Random Forest and Decision Tree.

The random forest method and results on cross-validation:


```r
fit1<-randomForest(classe ~. , data=train)
pred1 <- predict(fit1, testcv)
confusionMatrix(pred1,testcv$classe)$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##      1.0000000      1.0000000      0.9985297      1.0000000      0.2756282 
## AccuracyPValue  McnemarPValue 
##      0.0000000            NaN
```

The Decision Tree method and results on cross-validation:


```r
fit2<-rpart(classe ~ ., data=train, method="class")
pred2 <- predict(fit2, testcv,type="class")
confusionMatrix(pred2,testcv$classe)$overall
```

```
##       Accuracy          Kappa  AccuracyLower  AccuracyUpper   AccuracyNull 
##   7.546869e-01   6.896614e-01   7.373530e-01   7.714238e-01   2.756282e-01 
## AccuracyPValue  McnemarPValue 
##   0.000000e+00   2.933480e-16
```

Trough this cross validation we select the Random Forest method, wich execution time is higher, but accuracy is really good : 0.999.

## Conclusion :

When predicting with the selected model wich class we should have in the testing set, we obtain a rate of false predictions over all the predictions, this we'll be our expected sample error rate of the model :


```r
pred1 <- predict(fit1, test)
1-sum(pred1 == test$classe) / length(pred1)
```

```
## [1] 0.004588325
```

So the sample error rate of 0.3% allows us to think that the random forest algorithm gives a good predictor for the way we perform barbell.

Choosing random forest method was motivated by the fact that we wanted to predict a categorical variable, and that it gave us a very low sample error rate.

## File for the Assignment

Using our selected model we have the predictions for the testing set :


```r
pred1 <- predict(fit1, testing)
print(pred1)
```

```
##  1  2  3  4  5  6  7  8  9 10 11 12 13 14 15 16 17 18 19 20 
##  B  A  B  A  A  E  D  B  A  A  B  C  B  A  E  E  A  B  B  B 
## Levels: A B C D E
```


