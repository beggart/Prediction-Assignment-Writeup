---
title: " Practical Machine Learning - Prediction Assignment Writeup"
author: "Benjamin Eggart"
date: "19th of July 2021"
output: html_document
---

### 1. What it is about

This is the peer assesement solution for the Practical Machine Learning course . It predicts how six participants performed their exercises. This is also the "classe" variable of the training data. The algorithm is used on the 20 test cases available in the test data and the predictions are submitted.  it in the training set, is applied to the 20 test cases available in the test data. 

### 2. R Libraries that are used

```{r results='hide', message=FALSE}
library(knitr)
library(caret)
library(rpart)
library(rpart.plot)
library(rattle)
library(randomForest)
library(corrplot)
library(e1071)
library(gbm)
set.seed(123)
```

### 3. Getting, Loading and Cleaning the Data

Get the data.
```{r results='hide', message=FALSE}
# Download the dataset
Train <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv"
Test  <- "http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv"


```

Load the data and split it in training and test data.
```{r results='hide', message=FALSE}
# Read thedata
training <- read.csv(url(Train))
testing  <- read.csv(url(Test))

# create training dataset with 70% of the data
train_part  <- createDataPartition(training$classe, p=0.7, list=FALSE)
TrainSet <- training[train_part, ]
TestSet  <- training[-train_part, ]
dim(TrainSet)
```
The training data set is made of 19622 observations on 160 columns. The first seven columns give information about the people who did the test. They also provide a timestamps but this is ignored.
Remove NAs.
```{r results='hide', message=FALSE}
# Get the indexes of the columns having at least 90% of NA or blank values on the training data
indColToRemove <- which(colSums(is.na(TrainSet) |TrainSet=="")>0.9*dim(TrainSet)[1]) 
TrainSetClean <- TrainSet[,-indColToRemove]
TrainSetClean <- TrainSetClean[,-c(1:7)]
dim(TrainSetClean)
```
The same is performed on the test data.
```{r results='hid', message=FALSE}
# Get the indexes of the columns having at least 90% of NA or blank values on the test data
indColToRemove <- which(colSums(is.na(TestSet) |TestSet=="")>0.9*dim(TestSet)[1]) 
TestSetClean <- TestSet[,-indColToRemove]
TestSetClean <- TestSetClean[,-1]
dim(TestSetClean)
```

### 4. Where are the correlations?

Before having a modeling procedure a correlation among variables is performed.

```{r results='hid', message=FALSE}
corMatrix <- cor(TrainSetClean[, -53])
corrplot(corMatrix, order = "FPC", method = "color", type = "lower", 
         tl.cex = 0.5, tl.offset =1,  tl.col = rgb(0, 0, 0))

```

### 5. Prediction Models

Three methods will be applied to model regressions with the Train dataset. The best one (highest accuracy for the Test dataset) will be used for the project predictions. (1) Random Forests, (2) Decision Tree and (3) Generalized Boosted Model are investigated. A Confusion Matrix is plotted for each analysis to better visualize the accuracy of the models.

### 5.1 Random Forest

```{r results='hid', message=FALSE}
set.seed(123)
controlRF <- trainControl(method="cv", number=3, verboseIter=FALSE)
modFitRandForest <- train(classe ~ ., data=TrainSetClean, method="rf",
                          trControl=controlRF)
modFitRandForest$finalModel
```

```{r results='hid', message=FALSE}
# prediction and confusion matrix on Test dataset
predictRandForest <- predict(modFitRandForest, newdata=TestSetClean)
confMatRandForest <- confusionMatrix(table(predictRandForest, TestSetClean$classe))
confMatRandForest
```


```{r results='hid', message=FALSE}
# plot the results of the matrix
plot(confMatRandForest$table, col = confMatRandForest$byClass, 
     main = paste("Random Forest - Accuracy =",
                  round(confMatRandForest$overall['Accuracy'], 4)))
```

### 5.2 Decision Tree

```{r results='hid', message=FALSE}
set.seed(123)
modFitDecTree <- rpart(classe ~ ., data=TrainSetClean, method="class")

# prediction on Test dataset
predictDecTree <- predict(modFitDecTree, newdata=TestSetClean, type="class")
confMatDecTree <- confusionMatrix(table(predictDecTree, TestSetClean$classe))
confMatDecTree
```


```{r results='hid', message=FALSE}
# plot matrix results
plot(confMatDecTree$table, col = confMatDecTree$byClass, 
     main = paste("Decision Tree - Accuracy =",
                  round(confMatDecTree$overall['Accuracy'], 4)))
```

### 5.3 Generalized Boosted Model

```{r results='hid', message=FALSE}
set.seed(123)
controlGBM <- trainControl(method = "repeatedcv", number = 5, repeats = 1)
modFitGBM  <- train(classe ~ ., data=TrainSetClean, method = "gbm",
                    trControl = controlGBM, verbose = FALSE)

# prediction on Test dataset
predictGBM <- predict(modFitGBM, newdata=TestSetClean)
confMatGBM <- confusionMatrix(table(predictGBM, TestSetClean$classe))
confMatGBM
```

```{r results='hid', message=FALSE}
# plot matrix results
plot(confMatGBM$table, col = confMatGBM$byClass, 
     main = paste("GBM - Accuracy =", round(confMatGBM$overall['Accuracy'], 4)))
```

### 6. The Selected Model to the Test Data
The accuracy of the three regression models shown above are:

Random Forest = 0.9934, Decision Tree = 0.7573 Generalize Boosted Model = 0.9621.  Of course, Random Forest model will be applied to predict the 20 quiz results on testing dataset as shown below.

```{r results='hid', message=FALSE}
# plot matrix results
predictTEST <- predict(modFitRandForest, newdata=testing)
predictTEST
```
