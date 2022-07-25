## Predicting Life Expectancy - March-May 2022

**Project description:** During this project, I used data from the World Health Organization (WHO) to attempt to predict a given country's life expectancy. There are many reasons this could be useful, but life expectancy is generally seen as a good indicator of a population's overall well-being, so finding factors that increase this would undoubtedly be useful to policymakers looking to improve the well-being of their constituents.

### 1. Suggest hypotheses about the causes of observed phenomena


```{r}
#Make 'Status' a factor so data can be handled in regression
life$Status <- as.factor(life$Status)

#Get rid of country to keep it from being used as a categorical variable
life <- life[,-c(1)]
```

### 2. Assess assumptions on which statistical inference will be based

```javascript
if (isAwesome){
  return true
}
```

### 3. Support the selection of appropriate statistical tools and techniques

<img src="images/dummy_thumbnail.jpg?raw=true"/>

### 4. Provide a basis for further data collection through surveys or experiments

Sed ut perspiciatis unde omnis iste natus error sit voluptatem accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore veritatis et quasi architecto beatae vitae dicta sunt explicabo. 

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).
