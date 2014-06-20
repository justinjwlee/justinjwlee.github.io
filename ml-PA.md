Classification of barbell lifts on 5 different quality levels 
========================================================

# Synopsis

The goal of the report is to predict how well a group of 6 participants had performed a barbell lift. The data were collected from accelerometers on the belt, forearm, arm and dumbell of 6 participants. They were asked to perform barbell lifts correctly and incorrectly in 5 different ways. ('A' 'B' 'C' 'D' 'E' indicated under the 'classe' variable)

The training and test data for this project was downloaded from [here](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv) and [here](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv) respectively on 18 June 2014, 04:46am (UTC).

# Preprocessing

## Feature Selection
- All empty values and irrelevant values are replaced with NA.
- All variables with more than about 95% of samples (18600) being NA are removed.
- All descriptive variables (X, user name, new window, num window) with no bearing on the outcome of prediction are removed.
- Final processed data is named 'training'


```r
rawData <- read.csv("pml-training.csv", na.strings = c("NA", "", "#DIV/0!"))
totalNAs <- apply(rawData, 2, function(x) {
    sum(is.na(x))
})
reducedTraining <- rawData[, -which(totalNAs > 18600)]
training <- reducedTraining[, -grep("timestamp|X|user_name|new_window|num_window", 
    names(reducedTraining))]
```


## Data Splitting
- the training data is split into 70% (train) and 30% (crossValidation)


```r
library(caret)
```

```
## Loading required package: lattice
## Loading required package: ggplot2
```

```r
inTrain <- createDataPartition(y = training$classe, p = 0.7, list = FALSE)
train <- training[inTrain, ]
crossValidation <- training[-inTrain, ]
```


The final processed training set (train) contains 13737 observations and  53 variables

# Training
- The training data set is trained using Random Forest, with training control method set as 4-fold cross validation instead of default bootstrap sampling  
_decision was made to reduce computation time: current computation time is ~20min_

```r
set.seed(1234)
library(caret)
modFit <- train(classe ~ ., data = train, method = "rf", prox = TRUE, trControl = trainControl(method = "cv", 
    number = 4, allowParallel = T))
```


# Out of sample error
- The out of sample error was calculated using the cross validation (crossValidation) sample size of 5885 observations
- The out of sample error of the model is about 0.8% (1 - 0.992)
- Class A (exactly according to specification) was understandably most commonly misclassified as Class B (throwing elbows to the front)

```r
set.seed(1234)
library(caret)
confusionMatrix(crossValidation$classe, predict(modFit, crossValidation))
```

```
## Confusion Matrix and Statistics
## 
##           Reference
## Prediction    A    B    C    D    E
##          A 1671    1    1    0    1
##          B   16 1120    3    0    0
##          C    0    2 1023    1    0
##          D    0    0   11  951    2
##          E    0    0    1    6 1075
## 
## Overall Statistics
##                                        
##                Accuracy : 0.992        
##                  95% CI : (0.99, 0.994)
##     No Information Rate : 0.287        
##     P-Value [Acc > NIR] : <2e-16       
##                                        
##                   Kappa : 0.99         
##  Mcnemar's Test P-Value : NA           
## 
## Statistics by Class:
## 
##                      Class: A Class: B Class: C Class: D Class: E
## Sensitivity             0.991    0.997    0.985    0.993    0.997
## Specificity             0.999    0.996    0.999    0.997    0.999
## Pos Pred Value          0.998    0.983    0.997    0.987    0.994
## Neg Pred Value          0.996    0.999    0.997    0.999    0.999
## Prevalence              0.287    0.191    0.177    0.163    0.183
## Detection Rate          0.284    0.190    0.174    0.162    0.183
## Detection Prevalence    0.284    0.194    0.174    0.164    0.184
## Balanced Accuracy       0.995    0.997    0.992    0.995    0.998
```

