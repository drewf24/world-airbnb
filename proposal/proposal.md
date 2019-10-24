PROJECT TITLE
================
WORLD
10/22/2019

### Section 1. Introduction

Our topic is based on an AirBnb dataset in New York City for 2019.
Ultimately, we hope to determine the best AirBnb’s to rent based on
variables such as price, size, reviews and availability. We wanted to
look at Airbnb data because we thought this could be interesting insight
into the economic housing market in NYC. We plan to compare price by
neighborhood and borough which could provide interesting insights into
the cost of living in NYC. Through Kaggle we found this data set
(<https://www.kaggle.com/dgomonov/new-york-city-airbnb-open-data>) that
was collected in 2019 on Airbnb listings in New York City by a 4th year
data science student at Drexel University. The data set includes
variables: listing ID, name of the listing, host ID, name of the host,
location, neighbourhood, latitude and longitude coordinates, room type,
price in dollars, minimum\_nights, number of reviews, latest review,
number of reviews per month, amount of listing per host and
availability.

### Section 2. Exploratory data analysis

### Load packages & data

``` r
library(tidyverse) 
library(broom)
```

``` r
abnb <- read_csv("AB_NYC_2019.csv")
```

### Section 3. Research questions

### Section 4. Data
