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

For cross validation, we separate the data into k=5 folds.


```r
Control <- trainControl(method = "cv", number = 5)
```

### Model fitting

Random forest model is chosen. Accuracy of this model is computed based on the k-fold cross validation method.


```r
fit <- train(classe ~ ., method="rf", data = set_training, trControl = Control)
fit$resample
```

```
##    Accuracy     Kappa Resample
## 1 0.9541167 0.9419606    Fold1
## 2 0.9457187 0.9313541    Fold2
## 3 0.9487768 0.9352214    Fold5
## 4 0.9518717 0.9391007    Fold4
## 5 0.9576962 0.9464659    Fold3
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
##          A 27.4  0.7  0.2  0.3  0.0
##          B  0.2 17.9  0.3  0.1  0.2
##          C  0.4  0.6 16.7  0.6  0.1
##          D  0.4  0.1  0.1 15.4  0.1
##          E  0.0  0.1  0.0  0.1 17.9
##                             
##  Accuracy (average) : 0.9516
```

The average accuracy is 95% based on the validation technique that we used. 

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
