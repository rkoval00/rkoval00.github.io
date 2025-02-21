## Predicting Life Expectancy - March-May 2022

**Project description:** During this project, I used data from the World Health Organization (WHO) to attempt to predict a given country's life expectancy. There are many reasons this could be useful, but life expectancy is generally seen as a good indicator of a population's overall well-being, so finding factors that increase this would undoubtedly be useful to policymakers looking to improve the well-being of their constituents.

### Setup


```{r}
library(tidyverse)
library(glmnet)
library(randomForest)
library(gbm)
library(neuralnet)
set.seed(42)
```

### Initial Cleaning

This is the inital data cleaning step. I made the decision to simply remove all rows with NA values. The reason for this is because of a lot of factors that cannot be estimated that differ from country to country. Each country has a plethora of institutions that may make neighboring countries vastly different in some of the predictors used here. An example of this is North and South Korea, whose respective governments are so wildly different, which would undoubtedly cause differences in things measured in this dataset such as education and vaccination rates. Because of this, I don't feel comfortable replacing NA values with the median, for example.

After this, I changed the "Status" column into a factor, allowing it to be used as a predictor. I also removed "Country" as this should not be used to predict life expectancy given the number of unique countries. Once this is all done, the data is randomly split into training (70%) and testing (30%). 

```{r}
life_whole <- read.csv("Life Expectancy Data.csv")

life <- life_whole %>%
  drop_na()
  
life$Status <- as.factor(life$Status)

life <- life[,-c(1)]

split <- sample(nrow(life), nrow(life)*0.7)
train <- life[split,]
test <- life[-split,]

```

### Modeling
#### Naive Model

I first made a completely naive model to estimate life expectancy for each country. I fully expect this model to be a poor estimator of life expectancy, however it is still useful as a baseline to see if the other models are an improvement on using no data to make our prediction. For the naive model, I used the sample mean of the training data as the predicted value for the test data.

```{r}
samp_mean <- mean(train$Life.expectancy)
naive.mspe <- mean((samp_mean-test$Life.expectancy)^2)
```
#### Random Forest

Here, we tune the hyperparameter "mtry", which represents the number of predictors randomly sampled as candidates for each split.

```{r}
mtry <- tuneRF(train[,-3], train[,3], ntreeTry = 500, stepFactor = 0.5, trace=F, plot=F)
best_mtry <- mtry[mtry[,2] == min(mtry[,2]),1]

fit_rf <- randomForest(Life.expectancy ~., data=train, mtry=best_mtry)

rf_pred <- predict(fit_rf, test)
rf_mspe <- mean((rf_pred-test$Life.expectancy)^2)
```
#### Gradient Boost

Here, we tune the hyperparameter "n.trees", which represents the number of trees used to fit the model.

```{r}
fit_gbm <- gbm(Life.expectancy ~., data=train, distribution="gaussian", n.trees=10000, cv.folds=5)
best <- gbm.perf(fit_gbm, method='cv')
fit_gbm2 <- gbm(Life.expectancy ~., data=train, distribution="gaussian", n.trees=best, cv.folds=5)

gb_pred <- predict(fit_gbm2, test)
gb_mpse <- mean((gb_pred-test$Life.expectancy)^2)
```
#### Ordinary Least Squares
```{r}
ols1 <- lm(Life.expectancy ~ ., data=train)

ols1.pred <- predict(ols1, test)
ols1.mpse <- mean((ols1.pred - test$Life.expectancy)^2)
```
#### Backward Selection
```{r}
back <- step(ols1, direction="backward", trace=T)
back.pred <- predict(back, test)
back.mpse <- mean((back.pred - test$Life.expectancy)^2)
```
#### Ridge Regression

I opted for Ridge over Lasso because Backward Selection didn't remove that many variables, indicating to me that Lasso's advantage of shrinking some coefficients to 0 would not be very helpful.

```{r}
X <- as.matrix(train[,-c(1:3)])
y <- as.matrix(train[,3])
X.new <- data.matrix(test[,-c(1:3)])
y.new <- data.matrix(test[,3])
cv.ridge <- cv.glmnet(X, y, alpha = 0, lambda = exp(seq(-6, 2, length.out = 100)))
best.lambda <- cv.ridge$lambda.min
fit.ridge <- glmnet(X, y, alpha=0, lambda = best.lambda)
ridge.pred <- predict(fit.ridge, X.new)
ridge.mpse <- mean((ridge.pred-y.new)^2)
```
### Comparison

To compare each model, I opted to measure the mean squared prediction error (MSPE) for each model. This is calculated by using the trained models to predict the life expectancy for each unseen data point in the test data, squaring the difference between this prediction and the actual life expectancy, and averaging this across all data in the test set. Below is a table containing the MSPE for each of the above models:

<img src="images/life_table.png?raw=true"/>

The main takeaway here is the fact that the random forest model performs the best by far, almost halving the error of the next best model.

### Conclusion

Based on the random forest model's superiority, I made a variable importance plot to determine which factors contributed to predicting life expectancy. Below is the variable importance plot:

<img src="images/rf.png?raw=true"/>

What this illustrates is that four predictors have by far the greatest effect on life expectancy: income composition of resources (a measure of income inequality), HIV/AIDS rates, adult mortality, and years of schooling. In other words, countries' efforts to increase their population's life expectancy when they focus on decreasing income inequality, decreasing the prevalence of HIV/AIDS, lowering adult mortality, and encouraging more education. This insight is important, as it shows how countries with lower life expectancy can close the gap between themselves and richer countries while also working with limited time and resources.

### Data Source

[Life Expectancy Data](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who).
