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

### 2. Initial Cleaning

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

### 3. Support the selection of appropriate statistical tools and techniques

<img src="images/dummy_thumbnail.jpg?raw=true"/>

### 4. Data Source

[Life Expectancy Data](https://www.kaggle.com/datasets/kumarajarshi/life-expectancy-who).
