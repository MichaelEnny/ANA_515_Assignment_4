# ANA_515_Assignment_4

---
title: "ANA_515_Assignment_4"
author: "Michael Eniolade"
date: "4/26/2022"
output: 
  html_document:
    theme: 
      bootswatch: cerulean
---

##### 1.	Business problem/goal
This project is about Credit Card Fraud Detection using Machine Learning and R concepts. 
In this R Project, I will be illustrating how to perform detection of credit cards. 
I will also go through the various algorithms like Decision Trees, Logistic Regression, 
Artificial Neural Networks and finally, Gradient Boosting Classifier. 
For the purpose of this project, I will make use of the Card Transactions dataset that 
contains a mix of fraud as well as non-fraudulent transactions

##### 2.	Dataset source identification 
This dataset was retrieved from the data-flair website fro credit card fraud detection

##### 3. Code that imported and saved credit card dataset in R (creditcard_data <- read.csv("creditcard.csv"))

```{r setup, echo=TRUE}
knitr::opts_chunk$set(echo = TRUE)
library(ranger)
library(caret)
library(data.table)
# 3. Code that imported and saved credit card dataset in R 
getwd()
creditcard_data <- read.csv("creditcard.csv")
```
##### 4. Dataset description using below code
(summary(creditcard_data$Amount))

```{r dataExplore, echo=TRUE}
dim(creditcard_data)
head(creditcard_data,6)
tail(creditcard_data,6)
table(creditcard_data$Class)
summary(creditcard_data$Amount)
names(creditcard_data)
var(creditcard_data$Amount)
```

##### 5. Data Preparation : 
This is more of data manipulation to prepare the credit card data set for modelling. 
I applied the scale function on the amount figure variable and pass the whole data 
set to a new data set. Since raw data are not usually/always in the format we need 
to efficiently carry out our analysis. Raw data may also sometimes need to be rid 
of PII and PHI which can lead to legal/security issues.

```{r dataManipulation, echo=TRUE}
head(creditcard_data)
creditcard_data$Amount=scale(creditcard_data$Amount)
NewData=creditcard_data[,-c(1)]
head(NewData)
```
##### 6. Model Discussion:
At this point, I will be fitting the first model using logistic regression model.This is 
used to model the outcome probability of a classification related to credit card transaction
which in this case is the response (fraud/not fraud), The code below indicates how this is achieved

```{r Model, echo=TRUE}
#install.packages('caTools')
library(caTools)
set.seed(123)
data_sample = sample.split(NewData$Class,SplitRatio=0.80)
train_data = subset(NewData,data_sample==TRUE)
test_data = subset(NewData,data_sample==FALSE)
dim(train_data)
dim(test_data)
logistic_model = glm(Class~.,test_data, family=binomial())
summary(logistic_model)
```
##### 7. Output Discussion:

ROC : In order to assess the performance of the model, I will delineate the ROC curve. 
ROC is also known as Receiver Optimistic Characteristics. For this, I will first import
the ROC package and then plot the ROC curve to analyze its performance.

```{r Plot, echo=TRUE}
library(gbm, quietly=TRUE)
plot(logistic_model)
#fitting logistic regression to model
library(pROC)
lr.predict <- predict(logistic_model,train_data, probability = TRUE)
#auc.gbm = roc(test_data$Class, lr.predict, plot = TRUE, col = "blue")
```

Decision Tree :

In this section, I will implement a decision tree algorithm. Decision Trees 
to plot the outcomes of a decision. These outcomes are basically a consequence
through which we can conclude as to what class the object belongs to. I will 
now implement our decision tree model and will plot it using the rpart.plot() 
function. I will specifically use the recursive parting to plot the decision tree.

```{r DecTree, echo=TRUE}
library(rpart)
library(rpart.plot)
decisionTree_model <- rpart(Class ~ . , creditcard_data, method = 'class')
predicted_val <- predict(decisionTree_model, creditcard_data, type = 'class')
probability <- predict(decisionTree_model, creditcard_data, type = 'prob')
rpart.plot(decisionTree_model)
```

##### 8. Visualizations :

Artificial Neural Networks are a type of machine learning algorithm that are modeled after the human nervous system. The ANN models are able to learn the patterns using the historical data and are able to perform classification on the input data. I will import the neuralnet package that would allow me to implement the ANNs. Then I will proceeded to plot it using the plot() function. Now, in the case of Artificial Neural Networks, there is a range of values that is between 1 and 0. I will set a threshold as 0.5, that is, values above 0.5 will correspond to 1 and the rest will be 0. I will implement this using the code below

```{r ANC, echo=TRUE}
library(neuralnet)
ANN_model =neuralnet (Class~.,train_data,linear.output=FALSE)
plot(ANN_model)

predANN=compute(ANN_model,test_data)
resultANN=predANN$net.result
resultANN=ifelse(resultANN>0.5,1,0)
```

##### 9. GBM

Gradient Boosting is a popular machine learning algorithm that is used to perform classification and regression tasks. This model comprises of several underlying ensemble models like weak decision trees. These decision trees combine together to form a strong model of gradient boosting. I will implement gradient descent algorithm in the model using the code below

```{r GradienBoosting, echo=TRUE}
library(gbm, quietly=TRUE)

# Get the time to train the GBM model
system.time(
       model_gbm <- gbm(Class ~ .
               , distribution = "bernoulli"
               , data = rbind(train_data, test_data)
               , n.trees = 500
               , interaction.depth = 3
               , n.minobsinnode = 100
               , shrinkage = 0.01
               , bag.fraction = 0.5
               , train.fraction = nrow(train_data) / (nrow(train_data) + nrow(test_data))
)
)
# Determine best iteration based on test data
gbm.iter = gbm.perf(model_gbm, method = "test")
model.influence = relative.influence(model_gbm, n.trees = gbm.iter, sort. = TRUE)
#Plot the gbm model
plot(model_gbm)
```

##### Conclusion : 

Summary : Concluding the R Data Science project, This adequately presents how to develop a credit card fraud detection model using machine learning. I used a variety of ML algorithms to implement this model and also plotted the respective performance curves for the models. I show how data can be analyzed and visualized to discern fraudulent transactions from other types of data.
