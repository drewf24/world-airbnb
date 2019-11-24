NYC Airbnnb Popularity Factors
================
WORLD
12/15/2019

### Load packages & data

``` r
library(tidyverse) 
library(infer)
library(openintro)
library(dplyr)
library(knitr)
library(broom)
```

Uploaded the Airbnb data set from Kaggle via a csv file:

``` r
abnb <- read_csv("AB_NYC_2019.csv")
```

Assuming that the data we have for Airbnbs in New York City represents
the population. In order to perform the following analysis we used
sample\_n to randomly select 31 sample observations.

In part I, the following research question will be examined: How does
location (borough and neighborhood, for example) influence the price of
a listing?

In part II, the following research question will be examined: How does
the way in which a property is listed (type of room, for example)
influence the availability of a listing?

### Part A

Creating the sample that we will perform the following analysis on:

``` r
set.seed(111519)
abnb_sample <- sample_n(abnb, 50)
```

Constructing a bootstrap distribution for the median price of Airbnbs in
NYC:

``` r
set.seed(111519)
boot_dist_median_price <- abnb_sample %>%
  specify(response = price) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "median")
```

Creating a 95% bootstrap confidence interval for the the median price of
Airbnbs in NYC:

``` r
(ci_bounds <- get_ci(boot_dist_median_price, level= 0.95))
```

    ## # A tibble: 1 x 2
    ##   `2.5%` `97.5%`
    ##    <dbl>   <dbl>
    ## 1     95    148.

We are 95% confident that the population median price per night of
Airbnbs in NYC is between $97.50 and $147.51.

Creating a visualizaing of the bootstrap distribution for median price:

``` r
visualize(boot_dist_median_price) + 
  labs(title = "Bootstrap Dist of Median Price") +
  shade_ci(ci_bounds)
```

![](data-analysis_files/figure-gfm/boot_dist_median_vis-1.png)<!-- -->

### Part B

Using fct\_relevel to make Manhattan the baseline level for
neighbourhood\_group:

``` r
abnb_sample <- abnb_sample %>%
  mutate(neighbourhood_group = fct_relevel(neighbourhood_group, 
                                           "Manhattan", 
                                           "Brooklyn",
                                           "Staten Island",
                                           "Queens",
                                           "Bronx"))
```

Creating a linear model to predict Airbnb price by neighbourhood\_group:

``` r
lm_price_borough <- lm(price ~ neighbourhood_group, data = abnb_sample)

lm_price_borough %>% 
  tidy() %>% 
  select(term, estimate) %>%
  kable(format = "markdown", digits =3)
```

| term                              |  estimate |
| :-------------------------------- | --------: |
| (Intercept)                       |   180.333 |
| neighbourhood\_groupBrooklyn      |  \-63.000 |
| neighbourhood\_groupStaten Island |  \-80.333 |
| neighbourhood\_groupQueens        |  \-71.667 |
| neighbourhood\_groupBronx         | \-125.333 |

The linear model is:

`price-hat = 180.333 -63.000*(neighbourhood_groupBrooklyn)
-80.333*(neighbourhood_groupStaten Island)
-71.667*(neighbourhood_groupQueens) -125.333(neighbourhood_groupBronx)`

Intepreting the intercept:

Given that the Airbnb is in Manhattan the expected price, on average, is
$180.33. In this case, the intercept does have a meaningful
interpretation because an Airbnb could be $180.33 per night.

Interpreting the slopes using lm\_price\_borough linear model:

For an Airbnb in Brooklyn, the average price is expected, on average, to
be $63.00 less than an Airbnb in Manhattan, holding all else constant.

For an Airbnb in Staten Island, the average price is expected, on
average, to be $80.33 less than an Airbnb in Manhattan, holding all else
constant.

For an Airbnb in Queens, the average price is expected, on average, to
be $71.66 less than an Airbnb in Manhattan, holding all else constant.

For an Airbnb in Bronx, the average price is expected, on average, to be
$125.33 less than an Airbnb in Manhattan, holding all else constant.

### Part C

We are suspicious that there might be a relationship between median
price in Manhattan and Brooklyn. Therefore we will attempt to answer the
following: Is there is a difference in the true median prices between
Manhattan and Brooklyn?

``` r
abnb_sample_filtered <- abnb_sample %>%
  filter(neighbourhood_group == "Manhattan" | neighbourhood_group == "Brooklyn")
```

``` r
abnb_sample_filtered %>%
  group_by(neighbourhood_group) %>%
  summarise(med_price = median(price))
```

    ## # A tibble: 2 x 2
    ##   neighbourhood_group med_price
    ##   <fct>                   <dbl>
    ## 1 Manhattan               148  
    ## 2 Brooklyn                 92.5

The observed median prices for Manhattan and Brooklyn are $148.00 and
$92.50, respectively. Therefore, the observed difference in median
prices is $55.50.

H0: There is no difference in median price between Manhattan and
Brooklyn Airbnb per night. HA: There is a difference in median price
between Manhattan and Brooklyn Airbnb per night.

``` r
set.seed(111519)
null_dist_man_brook_med_price <- abnb_sample_filtered %>%
  specify(response = price, explanatory = neighbourhood_group) %>%
  hypothesize(null = "independence") %>%
  generate(reps = 1000, type = "permute") %>%
  calculate(stat = "diff in medians", order = c("Manhattan", "Brooklyn"))
```

Visualizing the null distribution.

``` r
visualize(null_dist_man_brook_med_price) + 
  labs(title = "Null Dist of Difference in Median Price between Manhattan and Brooklyn") +
  shade_p_value(obs_stat = 55.5, direction = "two_sided")
```

    ## Warning: F usually corresponds to right-tailed tests. Proceed with caution.

![](data-analysis_files/figure-gfm/null_dist_diff_median_vis-1.png)<!-- -->

Findng the
p-value.

``` r
get_p_value(null_dist_man_brook_med_price, 55.5, direction = "two_sided")
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.116

We reject the null hypothesis in favor of the alternative hypothesis.
The data does provide convincing evidence of a difference in median
price of Airbnbs for listings in Manhattan and Brooklyn.

Creating a variable (median\_price) to determine whether price is above
or below the median.

``` r
abnb_sample <- abnb_sample %>%
   mutate(price_median = median(price), 
          price = case_when(
      price >= price_median ~ "Median or Above",
      price < price_median ~ "Below Median", 
    ))
```

### Part D

Second research question: How does the way in which a property is listed
(type of room, for example) influence the availability of a listing?
