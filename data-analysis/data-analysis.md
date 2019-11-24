NYC Airbnnb Popularity Factors
================
WORLD
12/15/2019

### Load packages & data

``` r
library(tidyverse) 
library(infer)
library(openintro)
```

Uploaded the Airbnb data set from Kaggle via a csv file:

``` r
abnb <- read_csv("AB_NYC_2019.csv")
```

To start, the following research question will be examined: How does
location (borough and neighborhood, for example) influence the price of
a listing?

### Part A

Constructing a bootstrap distribution for the median price of Airbnbs in
NYC:

\`\`{r diff\_med\_income\_boot\_dist} boot\_dist\_median\_price \<- abnb
%\>% specify(response = price) %\>% generate(reps = 1000, type =
“bootstrap”) %\>% calculate(stat = “median”)

``` 



Creating a 95% bootstrap confidence interval for the difference in median incomes of employed Americans born in the first half of the year vs. those born in the second half:

``{r diff_med_income_boot_dist-ci}
set.seed(111319)
(ci_bounds <- get_ci(diff_med_income_boot_dist, level= 0.95))
```

We are 95% confident that the difference in median incomes of employed
Americans born in the first half of the year vs. those born in the
second half is between -6001.25 and 6500.

Second research question: How does the way in which a property is listed
(type of room, for example) influence the availability of a listing?
