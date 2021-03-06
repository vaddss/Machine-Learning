

###Practical Machine Learning Course Project (February, 2015)

####Summary
Six participants were asked to perform barbell lifts correctly and incorrectly in 5 different ways. Data from accelerometers on the belt, forearm, arm, and dumbell was recorded and structured into a dataset. More information is avaliable from [this webpage](http://groupware.les.inf.puc-rio.br/har).  
The purpose of this project is to predict the manner in which participants did the exercise (the `classe` variable in the training set). Two data sets are available for our exploration: [the training data](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv) and [the test data](https://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv).  
We want to build three prediction models and find out which one shows better perfomance (more accurate). 

####Loading and cleaning data
```{r}
train <- read.csv("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-training.csv", na.strings=c("NA","#DIV/0!", ","))
test <- read.csv("http://d396qusza40orc.cloudfront.net/predmachlearn/pml-testing.csv", na.strings=c("NA","#DIV/0!", ","))
```
First, seven variables are irrelevant for our exploration as they are dimensional:
```{r}
train <- train[,-c(1:7)]
test <- test[, -c(1:7)]
dim(train)
```
Next, we decide to remove all variables which contain "NA","#DIV/0!" and "," because these variables could distort our results.  
```{r}
#First, we create a vector which contains variables without any Na`s:
var_without_NA <- apply(train,2, function(x) !any(is.na(x)))
train <- train[,var_without_NA]
test <- test[,var_without_NA]
#Let`s check if we have removed all Na`s:
sum(apply(train,2, function(x) any(is.na(x))))
dim(train)
dim(test)
```

####Building models
We want to build three prediction models using two methods: Decision Tree, KNN and Random Forest. The expected out of sample error equals: one minus algorithm`s accuracy on a test data. In order to perfom cross-validation, we want to separate our training set into two subsets:
```{r, warning=FALSE, message=FALSE}
library(caret)
library(rpart)
set.seed(12345)
inTrain = createDataPartition(train$classe, p = .7, list=F)
train_train = train[ inTrain,]
train_test = train[-inTrain,]
```
```{r, results='hide'}
#Desicion tree model:
dt_model <- train(classe~., method="rpart", data=train_train, trControl = trainControl(method = "cv", number = 4, allowParallel = TRUE, verboseIter = TRUE))
```
```{r}
dt_pred <- predict(dt_model, newdata=train_test)
(dt_matrix<-confusionMatrix(dt_pred, train_test$classe))
```
```{r, results='hide'}
#KNN model:
knn_model <- train(classe~., method="knn", data=train_train, trControl = trainControl(method = "cv", number = 4, allowParallel = TRUE, verboseIter = TRUE))
```
```{r}
knn_pred <- predict(knn_model, newdata=train_test)
(knn_matrix <- confusionMatrix(knn_pred, train_test$classe))
```
In order to build a model based on the random forest algorithm we will use the `randomForest` package:  
```{r, results='hide', warning=FALSE, message=FALSE}
library(randomForest)
#Random forest model:
rf_model <- randomForest(classe~., data=train_train) 
```
```{r}
rf_pred <- predict(rf_model, newdata=train_test)
(rf_matrix <- confusionMatrix(rf_pred, train_test$classe))
```
Now we compare the three models using accuracy of each one:
```{r}
#Decision tree accuracy:
round(dt_matrix$overall[1],3)
#KNN accuracy:
round(knn_matrix$overall[1],3)
#Random Forest accuracy:
round(rf_matrix$overall[1],3)
```
####Conclusion
The model with the highest accuracy (0.993) was built upon the random forest algorithm. The expected out of sample error is 0.007.  We will use this model for further predictions. The KNN model works rather well (accuracy 0.902) but has a slighly lower accuracy. And, at last, the decision tree model perfoms very poorly on your data with accuracy only 0.496.  

####Apply model to test data
Now we will apply the chosen model to the `test` dataset:
```{r}
(answers <- predict(rf_model, newdata=test))
```