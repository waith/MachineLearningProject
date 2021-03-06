Machine Learning Project - PML Study
========================================================
Background
----------
Using devices such as Jawbone Up, Nike FuelBand, and Fitbit it is now possible to collect a large amount of data about personal activity relatively inexpensively. These type of devices are part of the quantified self movement - a group of enthusiasts who take measurements about themselves regularly to improve their health, to find patterns in their behavior, or because they are tech geeks. One thing that people regularly do is quantify how much of a particular activity they do, but they rarely quantify how well they do it. 

Objective
---------
In this project, we will use data from accelerometers on the belt, forearm, arm, and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. More information is available from the website here: http://groupware.les.inf.puc-rio.br/har (see the section on the Weight Lifting Exercise Dataset).  

The goal of the project is to predict the manner in which they did the exercise. This is the "classe" variable in the training set. There are 5 different types of Classes : A,B,C,D,E

Analysis
--------

#### Predictive Method

The prediction is on a variable (classe) which is categorical.  Based on this categorical nature, CART (tree) and Random Forest will be used to build the model. The best model will be used for predicting the 20 cases in testing set.

#### Data preparation
There are 160 variables in the training set. Variables not related to motion are removed. This reduces number of variables to 39 variables (38 predictors and the target: classe).  This data cleaning is done in both training and testing data sets


```r
install.packages("caret")
```

```
## Installing package into 'C:/Users/Helena/Documents/R/win-library/3.1'
## (as 'lib' is unspecified)
```

```
## Error: trying to use CRAN without setting a mirror
```

```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
install.packages("e1071")
```

```
## Installing package into 'C:/Users/Helena/Documents/R/win-library/3.1'
## (as 'lib' is unspecified)
```

```
## Error: trying to use CRAN without setting a mirror
```

```r
library(e1071)
#load and clean training set
train<-read.csv("pml-training.csv")
exclude<-c("*min*","*std*","*max*","var","*avg*","*skew*","*kurtosis*","*amplitude*","*total*","*name*","X")
exclusion<-grep(paste(exclude,collapse="|"),colnames(train),ignore.case=F, value=F)
training <- train[,-exclusion]
#load and clean testing set
test<-read.csv("pml-testing.csv")
exclusion<-grep(paste(exclude,collapse="|"),colnames(test),ignore.case=F, value=F)
testing <- test[,-exclusion]
```

#### Sample size
Limited by the capacity of the computer, only 3000 samples are randamly selected from the original training file (19622 samples) to perform the training. If the accuracy of prediction is too low, a larger sample size will be added It is noted later that the accuracy justisfied this decision.


```r
set.seed(123)
trainInds <- sample(nrow(training), 3000)
trainset <- training[trainInds,]
```

#### CART (Tree) Model
Given the limited capacity of the computer,  bootstrap (default) is not used and cross validation is set to 4-fold.  It is noted that accuracy from CART model is only 50% which is not satisfactory.



```r
set.seed(123)
modFit_rpart<-train(classe~.,data=trainset,method="rpart", trControl = trainControl(method = "cv", number = 4))
```

```
## Loading required package: rpart
```

```r
modFit_rpart
```

```
## CART 
## 
## 3000 samples
##   38 predictors
##    5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (4 fold) 
## 
## Summary of sample sizes: 2250, 2249, 2251, 2250 
## 
## Resampling results across tuning parameters:
## 
##   cp    Accuracy  Kappa  Accuracy SD  Kappa SD
##   0.04  0.5       0.4    0.02         0.03    
##   0.05  0.5       0.3    0.04         0.06    
##   0.1   0.3       0.03   0.03         0.05    
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was cp = 0.04.
```

#### Random Forest Model

Given the limited capacity of the computer,  bootstrap (default) is not used and cross validation is set to 4-fold.  It is noted that accuracy from Random Forest model is over 95% which is very satisfactory.


```r
set.seed(123)
modFit_rf<-train(classe~.,data=trainset,method="rf", trControl = trainControl(method = "cv", number = 4))
```

```
## Loading required package: randomForest
## randomForest 4.6-7
## Type rfNews() to see new features/changes/bug fixes.
```

```r
modFit_rf
```

```
## Random Forest 
## 
## 3000 samples
##   38 predictors
##    5 classes: 'A', 'B', 'C', 'D', 'E' 
## 
## No pre-processing
## Resampling: Cross-Validated (4 fold) 
## 
## Summary of sample sizes: 2250, 2249, 2251, 2250 
## 
## Resampling results across tuning parameters:
## 
##   mtry  Accuracy  Kappa  Accuracy SD  Kappa SD
##   2     1         0.9    0.009        0.01    
##   20    1         1      0.005        0.006   
##   40    1         1      0.005        0.007   
## 
## Accuracy was used to select the optimal model using  the largest value.
## The final value used for the model was mtry = 38.
```

###### In sample Error Rate
The in sample error rate is recorded 0%

```r
#perform prediction
pred_rf_sample<-predict(modFit_rf,trainset)
#Confusion Matrix - Compare results
table(trainset$classe)
```

```
## 
##   A   B   C   D   E 
## 859 575 526 517 523
```

```r
table(pred_rf_sample)
```

```
## pred_rf_sample
##   A   B   C   D   E 
## 859 575 526 517 523
```

```r
table(pred_rf_sample, trainset$classe)
```

```
##               
## pred_rf_sample   A   B   C   D   E
##              A 859   0   0   0   0
##              B   0 575   0   0   0
##              C   0   0 526   0   0
##              D   0   0   0 517   0
##              E   0   0   0   0 523
```

```r
InSampleError<- round(sum(!pred_rf_sample==trainset$classe)/nrow(trainset),2)
InSampleError
```

```
## [1] 0
```

###### Out of sample Error Rate
As only 3000 samples are used in the training set, another 3000 samples are randamly selected from the remaining cases to be the out of sample set. It is noted that the out of sample error rate is around 2 %


```r
#3000 out of sample cases are randomly selected from the remaining of the training set
set.seed(123)
outsample<- training[-trainInds,]
outsampleInds <- sample(nrow(outsample), 3000)
outsampleset <- outsample[outsampleInds,]
#perform prediction
pred_rf_outsample<-predict(modFit_rf,outsampleset)
#Confusion Matrix - Compare results
table(outsampleset$classe)
```

```
## 
##   A   B   C   D   E 
## 873 572 530 506 519
```

```r
table(pred_rf_outsample)
```

```
## pred_rf_outsample
##   A   B   C   D   E 
## 886 574 519 516 505
```

```r
table(pred_rf_outsample,outsampleset$classe)
```

```
##                  
## pred_rf_outsample   A   B   C   D   E
##                 A 870  11   0   0   5
##                 B   3 557  13   0   1
##                 C   0   1 513   4   1
##                 D   0   3   4 502   7
##                 E   0   0   0   0 505
```

```r
OutSampleError<- round(sum(!pred_rf_outsample==outsampleset$classe)/nrow(outsampleset),2)
OutSampleError
```

```
## [1] 0.02
```

Testing Results on 20 Testing Cases
---------------
#### Apply Random Forest Model to 20 Test Cases

The Random Forest method is selected to perform the testing. It has performed an excellent prediction with all 20 test cases are predicted correctly of their respective classes. The prediction accuracy is 100%.


```r
pred_rf<-predict(modFit_rf,testing)
pred_rf
```

```
##  [1] B A B A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```
