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
```

Uploaded the Airbnb data set from Kaggle via a csv file:

``` r
abnb <- read_csv("AB_NYC_2019.csv")
```

To start, the following research question will be examined: How does
location (borough and neighborhood, for example) influence the price of
a listing?

### Part A

For bootstrapping, a sample to perform a bootstrap analysis.

``` r
set.seed(111519)
abnb_sample <- sample_n(abnb, 30)
```

Constructing a bootstrap distribution for the median price of Airbnbs in
NYC:

``` r
boot_dist_median_price <- abnb_sample %>%
  specify(response = price) %>%
  generate(reps = 1000, type = "bootstrap") %>%
  calculate(stat = "median")
```

Creating a 95% bootstrap confidence interval for the median price of
Airbnbs:

``` r
set.seed(111519)
(ci_bounds <- get_ci(boot_dist_median_price, level= 0.95))
```

    ## # A tibble: 1 x 2
    ##   `2.5%` `97.5%`
    ##    <dbl>   <dbl>
    ## 1   92.5    138.

We are 95% confident that the median price of Airbnbs per night in NYC
is between $92.50 and $138.50.

### Part B

Created a linear model predict Airbnb price by borough
(neighbourhood\_group):

``` r
abnb %>%
  mutate(neighbourhood_group = fct_relevel(neighbourhood_group, "Manhattan", "Brooklyn","Staten Island","Queens", "Bronx"))
```

    ## # A tibble: 48,895 x 16
    ##       id name  host_id host_name neighbourhood_g… neighbourhood latitude
    ##    <dbl> <chr>   <dbl> <chr>     <fct>            <chr>            <dbl>
    ##  1  2539 Clea…    2787 John      Brooklyn         Kensington        40.6
    ##  2  2595 Skyl…    2845 Jennifer  Manhattan        Midtown           40.8
    ##  3  3647 THE …    4632 Elisabeth Manhattan        Harlem            40.8
    ##  4  3831 Cozy…    4869 LisaRoxa… Brooklyn         Clinton Hill      40.7
    ##  5  5022 Enti…    7192 Laura     Manhattan        East Harlem       40.8
    ##  6  5099 Larg…    7322 Chris     Manhattan        Murray Hill       40.7
    ##  7  5121 Blis…    7356 Garon     Brooklyn         Bedford-Stuy…     40.7
    ##  8  5178 Larg…    8967 Shunichi  Manhattan        Hell's Kitch…     40.8
    ##  9  5203 Cozy…    7490 MaryEllen Manhattan        Upper West S…     40.8
    ## 10  5238 Cute…    7549 Ben       Manhattan        Chinatown         40.7
    ## # … with 48,885 more rows, and 9 more variables: longitude <dbl>,
    ## #   room_type <chr>, price <dbl>, minimum_nights <dbl>,
    ## #   number_of_reviews <dbl>, last_review <date>, reviews_per_month <dbl>,
    ## #   calculated_host_listings_count <dbl>, availability_365 <dbl>

``` r
lm(price ~ neighbourhood_group, data = abnb)
```

    ## 
    ## Call:
    ## lm(formula = price ~ neighbourhood_group, data = abnb)
    ## 
    ## Coefficients:
    ##                      (Intercept)       neighbourhood_groupBrooklyn  
    ##                            87.50                             36.89  
    ##     neighbourhood_groupManhattan         neighbourhood_groupQueens  
    ##                           109.38                             12.02  
    ## neighbourhood_groupStaten Island  
    ##                            27.32

The linear model is:

`price-hat = 87.50 +36.89*(neighbourhood_groupBrooklyn)
+12.02*(neighbourhood_groupQueens) +27.32*(neighbourhood_groupStaten
Island)`

Manhattan is the baseline level for borough variable.

Intepreting the intercept:

Given that a teaching professor has an average attractiveness of 0,
their expected professor evaluation, on average, is 3.982 out of 5. In
this case, the intercept does not have a meaningful interpretation
because a student could not rate their professor a 0. The least they
could rate their professor is a 1.

Interpreting the slopes using m\_bty\_rank linear model:

For each additional point rating increase in the attractiveness scale
(out of 10), the average professor evaluation is expected to increase,
on average, by 0.068 rating points (out of 5), holding all else
constant.

Professors on the tenure track are expected to have, on average, an
average professor evaluation score which is 0.161 rating points lower
(out of 5) than the average professor rating of teaching professors,
holding all else constant.

Tenured professors are expected to have, on average, an average
professor evaluation score which is 0.126 rating points lower (out of 5)
than the average professor rating of teaching professors, holding all
else constant.

### Part C

``` r
abnb_sample <- abnb_sample %>%
   mutate(price_median = median(price), 
          price = case_when(
      price > price_median ~ "Above",
      price < price_median ~ "Below"
    ))
```

We will now perform a hypothesis test to determine if there is a
difference in the true median prices between Manhattan and Brooklyn.

We will set our alpha level of 5%.

m1 = median price of Manhattan m2 = median price of Brooklyn

Null Hypothesis: There is no difference between the median prices
between Manhattan and Brooklyn. Ho: m1 - m2 = 0

Alternative Hypothesis: There is a difference between the median prices
between Manhattan and Brooklyn. Ha: m1 - m2 \!= 0

If the p-value is greater than our predetermined alpha level of 0.05,
then we fail to reject the null hypothesis and cannot conlude that there
is a statistically significant difference between the median prices of
Manhattan and Brooklyn.

If the p-value is less than our predetermined alpha level of 0.05, then
we reject the null hypothesis and have convincing evidence to conclude
that there is a statistically significant difference bteween the median
prices of Manhattan and Brooklyn.

### Part D

Second research question: How does the way in which a property is listed
(type of room, for example) influence the availability of a listing?
