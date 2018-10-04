---
title: "Quantified Self-movement Data Analysis"
output: 
  html_document:
    keep_md: true
---

#### *Fernaldo R. Winnerdy*
#### *28-Sep-2018*

The project is about predicting the forms of exercise based on the measurements of accelerometer on 6 different subjects. The test subjects were to performed belt, forearm, arm, and dumbell exrecises in a correct form and four different commonly used wrong forms. Base on the data, we try to predict the forms that a person do.

### Data import
First, we import the data to R (data were downloaded beforehand) and load the required library.


```r
library(caret)
```

```
## Loading required package: lattice
```

```
## Loading required package: ggplot2
```

```r
library(rpart)
setwd('/home/fernaldo/Projects/Others/Programming/R-specialization/08-Practical_machine_learning/Week-04')
training <- read.csv('pml-training.csv')
testing <- read.csv('pml-testing.csv')
```

### Cleaning data

Here, we clean the data (both training and testing) such that it includes only the variables from the accelerometer, as well as removing the variables with NA.


```r
accel_index <- grep(pattern = 'accel', names(training))
set_training <- training[, c(accel_index, 160)]
set_training <- set_training[, colSums(is.na(set_training)) == 0]
set_testing <- testing[, c(accel_index, 160)]
set_testing <- set_testing[, colSums(is.na(set_testing)) == 0]
```

We end up with only 17 variables in both training and testing set

### Cross validation separation

Since we need out of sample error estimate using cross-validation, we need to further split the training set into 2, 70% into a training set and 30% into a validation set.


```r
inTrain <- createDataPartition(y = set_training$classe, p=0.7, list=FALSE)
tra <- set_training[inTrain, ]
val <- set_training[-inTrain, ]
```

For cross validation, we separate the data into k=5 folds.


```r
Control <- trainControl(method = "cv", number = 5)
```

### Model fitting

Random forest model is chosen. Accuracy of this model is computed based on the k-fold cross validation method.


```r
fit <- train(classe ~ ., method="rf", data = tra, trControl = Control)
fit$resample
```

```
##    Accuracy     Kappa Resample
## 1 0.9410266 0.9253371    Fold1
## 2 0.9337218 0.9160886    Fold2
## 3 0.9417758 0.9263027    Fold5
## 4 0.9363173 0.9194586    Fold4
## 5 0.9461426 0.9318568    Fold3
```

```r
confusionMatrix.train(fit)
```

```
## Cross-Validated (5 fold) Confusion Matrix 
## 
## (entries are percentual average cell counts across resamples)
##  
##           Reference
## Prediction    A    B    C    D    E
##          A 27.4  0.9  0.3  0.4  0.1
##          B  0.2 17.4  0.5  0.1  0.2
##          C  0.4  0.7 16.5  0.8  0.2
##          D  0.5  0.1  0.1 15.0  0.2
##          E  0.0  0.2  0.0  0.1 17.7
##                             
##  Accuracy (average) : 0.9398
```

The average accuracy is ~94% based on the validation technique that we used.  

### Cross Validation accuracy

The out of sample accuracy (using validation data):


```r
confusionMatrix(predict(fit, val), val$classe)
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1659   14    4    7    1
##          B    3 1115    8    3    5
##          C    4    6 1009    9    1
##          D    8    1    3  943    2
##          E    0    3    2    2 1073
## 
## Overall Statistics
##                                          
##                Accuracy : 0.9854         
##                  95% CI : (0.982, 0.9883)
##     No Information Rate : 0.2845         
##     P-Value [Acc > NIR] : <2e-16         
##                                          
##                   Kappa : 0.9815         
##  Mcnemar's Test P-Value : 0.2072         
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity            0.9910   0.9789   0.9834   0.9782   0.9917
## Specificity            0.9938   0.9960   0.9959   0.9972   0.9985
## Pos Pred Value         0.9846   0.9832   0.9806   0.9854   0.9935
## Neg Pred Value         0.9964   0.9949   0.9965   0.9957   0.9981
## Prevalence             0.2845   0.1935   0.1743   0.1638   0.1839
## Detection Rate         0.2819   0.1895   0.1715   0.1602   0.1823
## Detection Prevalence   0.2863   0.1927   0.1749   0.1626   0.1835
## Balanced Accuracy      0.9924   0.9875   0.9897   0.9877   0.9951
```

Given 95% accuracy of the prediction using the validation set, the expected out of sample error is 

### Test result
The test prediction results are as follows


```r
result <- predict(fit, set_testing)
result
```

```
##  [1] B A C A A E D B A A B C B A E E A B B B
## Levels: A B C D E
```

The above results were used to predict the classe variables of the testing set in the following project quiz. 
