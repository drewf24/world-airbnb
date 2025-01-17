NYC Airbnnb Popularity Factors
================
Team World
10/24/2019

### Section 1. Introduction

Using the Airbnb dataset, we hope to gain a better understanding of the
gig hospitality market in New York City for 2019. Specifically, we want
to find, explore and understand trends in the market as they relate to
supply (availability) and demand (what people are willing to pay). The
data set we chose allows us to draw conclusions and understand the
factors that contribute to the New York City Airbnb scene.

Through Kaggle, we found this data set
(<https://www.kaggle.com/dgomonov/new-york-city-airbnb-open-data>) that
was collected, and continues to be updated, in 2019 on Airbnb listings
in New York City. This data set continues to be curated by a 4th year
data science student at Drexel University (Dgomonov). We are unable to
reach out to him to get more information on the data set, such as where
he got it from or how he collected it, because our account on Kaggle is
not at the “Contributor tier.”

To analyze this data set it is first important to understand its
variables which include: listing ID, name of the listing, host ID, name
of the host, location, neighbourhood, latitude and longitude
coordinates, room type, price in dollars, minimum\_nights, number of
reviews, latest review, number of reviews per month, amount of listing
per host and availability. A description of each of these is included in
the codebook.

Based on these factors, it seems as though price and availibility would
make the most interesting response variables because they are very
important to understand the economic landscape. For example someone who
is looking to book an Airbnb is going to be very interested in what is
available within their budget. Of course there are going to be other
factors that are important to a consumer, such as location within NYC.
However, we expect availability and price to be the most predictive and
representative of the Airbnbs in New York City.

Some variables from the dataset that would be interesting to explore in
section two that pertain to location include the borough, neighborhood,
latitude, and longitude. We know that in real estate location highly
influences value, especially in a large, wealthy, productive city like
New York. We hope to explore how location influences potential response
variables such as availability and price.

We also hope to explore how the listing is presented on the Airbnb
platform might influence consumer choices. Some variables that could be
insightful include the name of the listing which we may use text
analysis to analyze. Each observation is also classified as one of three
room types: private room, shared room and entire home/apt. Moreover,
minimum nights required to stay and frequency of reviews are also
included. We hope to see how this mix of categorical and numerical
values impact one of the potential response variables.

Although section two is preliminary exploration, various tools from the
course such as the group\_by function, the summarise function, linear
modeling, and possible joining with outside data to draw meaningful
insights. Clear and specific notation will be employed to make sure
analysis is reproducible.

### Section 2. Exploratory data analysis

### Load packages & data

Loaded the tidyverse and broom packages:

``` r
library(tidyverse) 
library(broom)
```

Uploaded the Airbnb data set from Kaggle via a csv file:

``` r
abnb <- read_csv("AB_NYC_2019.csv")
```

Creating a histogram that displays number of listings by neighborhood
borough in New York City:

``` r
ggplot(data = abnb, mapping = aes(x = neighbourhood_group)) +
  geom_histogram(stat = "count") + 
  labs(title = "Listings by Borough", x = "Borough", y = "Count")
```

    ## Warning: Ignoring unknown parameters: binwidth, bins, pad

![](proposal_files/figure-gfm/visualization-univariate-1.png)<!-- -->

Manhattan and Brooklyn dominate the number of listings on Airbnb in New
York City. More specifically, there is roughly 20,000 listings in
Brooklyn and slightly more than 20,000 listings in Manhattan. The Queens
borough follows after with roughly 5,000 listings, followed by Bronx and
Staten Island with fewer than 1,100 listings each.

Creating summary statistics that will count the number of listings in
each neighborhood group:

``` r
abnb %>%
  count(neighbourhood_group) %>%
  arrange(desc(n))
```

    ## # A tibble: 5 x 2
    ##   neighbourhood_group     n
    ##   <chr>               <int>
    ## 1 Manhattan           21661
    ## 2 Brooklyn            20104
    ## 3 Queens               5666
    ## 4 Bronx                1091
    ## 5 Staten Island         373

Manhattan has the most listings at 21,661 followed by Brooklyn at 20,104
listings, followed by Queens at 5,666, Bronx at 1,091, and Staten Island
last at only 373 listings.

Creating a facet histogram that will display the price of listings by
borough:

``` r
abnb %>%
  ggplot(mapping = aes(x = price)) +
  geom_histogram(bins=35) +
  facet_wrap(.~neighbourhood_group) +
  labs(title = "Price of Listings by Borough", x = "Price ($)", y = "Count")
```

![](proposal_files/figure-gfm/price-borough-1.png)<!-- -->

The prices for listings are very skewed to the right. In other words,
there are many outliers that fall out of the IQR for each neighborhood
group. Most listings fall in the 50 to 150 price range, with the
outliers at much higher prices. The graph also shows us the number of
listings at a given price within that borough.

Creating summary statistics that displays the median and IQR price of
each neighborhood group.

``` r
abnb %>%
  group_by(neighbourhood_group) %>%
  summarise(
    med_price = median(price), 
    IQR_price = IQR(price)
    ) %>%
  arrange(desc(med_price)) %>%
  head(5)
```

    ## # A tibble: 5 x 3
    ##   neighbourhood_group med_price IQR_price
    ##   <chr>                   <dbl>     <dbl>
    ## 1 Manhattan                 150       125
    ## 2 Brooklyn                   90        90
    ## 3 Queens                     75        60
    ## 4 Staten Island              75        60
    ## 5 Bronx                      65        54

Manhattan has the highest median price at 150 followed by Brooklyn at
90, Queens and Staten Island at 75, and Bronx last at 65.

Creating summary statistics that finds the neighbourhoods with the
highest median prices and gives their corresponding borough:

``` r
abnb %>%
  group_by(neighbourhood_group, neighbourhood) %>%
  summarise(median_price = median(price)) %>%
  arrange(desc(median_price)) %>%
  head(5)
```

    ## # A tibble: 5 x 3
    ## # Groups:   neighbourhood_group [3]
    ##   neighbourhood_group neighbourhood  median_price
    ##   <chr>               <chr>                 <dbl>
    ## 1 Staten Island       Fort Wadsworth          800
    ## 2 Staten Island       Woodrow                 700
    ## 3 Manhattan           Tribeca                 295
    ## 4 Queens              Neponsit                274
    ## 5 Manhattan           NoHo                    250

Fort Wadsworth in Staten Island has the highest median price at 800
followed by Woodrow at 700, Tribeca at 295, Neponsit at 274, and NoHo at
250.

Creating a scatterplot that finds the prices at different latitude and
longitude co-ordinates:

``` r
abnb %>%
  ggplot(mapping = aes (x = longitude, y = latitude, color = price)) +
  geom_point(alpha = 0.3) +
  labs(title = "Prices at Latitude and Longitude Coordinates", x = "Longitude", y = "Latitude")
```

![](proposal_files/figure-gfm/price-at-longtitude-1.png)<!-- -->

Most listings take place in the (-74, -73.7) longtitude region and
(40.55, 40.9) latitude region. Prices are generally in the 50 to 150
range.

Creating a histogram that describes the number of days a listing is
available in a year:

``` r
abnb %>%
  ggplot(mapping = aes(x = availability_365)) +
  geom_histogram(binwidth=80) + 
  labs(title = "Availability of Listings Distribution",
       x = "Availability (days per year)",
       y = "Count"
  )
```

![](proposal_files/figure-gfm/avail-listings-distr-1.png)<!-- -->

The graph is skewed to the right with most occurrences happening in the
0 to 100 availibily (days per year). More specifically, approximately
more than 25,000 listings occurr in the 0 to 100 availability (days per
year).

Creating a boxplot to show which room type (shared, private or
house/apartment) is available for the most amount of days per year:

``` r
abnb %>%
  ggplot(mapping = aes(x =  room_type, y = availability_365, fill = room_type)) +
  geom_boxplot() + 
  labs(
    title = "Availability by Room Type",
    x = "Room Type", 
    y = "Availability (days per year)")
```

![](proposal_files/figure-gfm/room_type-availability_365-1.png)<!-- -->

The graph above are boxplots that show that the availability based on
room type is also skewed to the right. The median available number of
days days among entire home and private rooms are slightly less than 50
days while shared rooms have a rough median availability of 90 days.
Shared rooms are available for the most number of days in the year.

Creating a table to describe summary statistics for availibility by room
type:

``` r
abnb %>%
  group_by(room_type) %>%
  summarise(
    med_availability_365 = median(availability_365), 
    IQR_availability_365 = IQR(availability_365)
    ) %>%
  arrange(desc(med_availability_365))
```

    ## # A tibble: 3 x 3
    ##   room_type       med_availability_365 IQR_availability_365
    ##   <chr>                          <dbl>                <dbl>
    ## 1 Shared room                       90                  341
    ## 2 Private room                      45                  214
    ## 3 Entire home/apt                   42                  229

The median availability are 90 days for shared room, 45 days for private
room, and 42 days for entire home/apartment. The range for shared room
type is the highest with 341 days.

Creating a new variable mutate to use availibility as a index of
popularity of the listing based on “busy”, “not busy”, “Somewhat Busy”,
and “NA”. Visualizing this data using a segmented barplot:

``` r
abnb %>%
  mutate(avail = case_when(
    availability_365 < 100            ~ "Very Busy",
    availability_365 < 200 & availability_365 > 100  ~ "Somewhat Busy",
    availability_365 > 200    ~ "Not Busy",)
    ) %>%
ggplot(mapping = aes(x = room_type, fill = avail)) +
  geom_bar(position = "fill") +
  labs(title = "Popularity of Listings by Room Type", 
       x = "Room Type", y = "Proportion", 
       fill = "Busyness of Room Type") 
```

![](proposal_files/figure-gfm/prop-of-avail-1.png)<!-- -->

The stacked bar graph above shows the proprotion of listings based on
room type that are either not busy, very busy, somewhat busy, or have no
data (NAs).

According to the graph, all entire home, private rooms, shared room
listing types are generally mostly very busy at roughly 60% while only
roughly 30% across the charts are not busy. Shared rooms are generally
less busy than private rooms and entire home/apt listings.

Creating a facet scatterplot that will display the busyness of different
room types:

``` r
abnb %>%
  mutate(avail = case_when(
    availability_365 < 100            ~ "Very Busy",
    availability_365 < 200 & availability_365 > 100  ~ "Somewhat Busy",
    availability_365 > 200    ~ "Not Busy",)
    ) %>%
ggplot(mapping = aes(x = calculated_host_listings_count, y = reviews_per_month)) +
  geom_point() +
  facet_grid(room_type ~ avail) +
  labs(title = "Host Listings vs. Reviews Frequency by Availability & Room Type", x = "Number of Listings (per host)", y = "Number of Reviews (per month)")
```

    ## Warning: Removed 10052 rows containing missing values (geom_point).

![](proposal_files/figure-gfm/avail-by-room_type-reviews-1.png)<!-- -->

The graphs above show that among entire home/apt and private rooms,
there is a higher amount of reviews per month (roughly less than 20)
with fewer calculated host listings.

### Section 3. Research questions

The first research question that we are interested in exploring is “How
does location (borough and neighborhood, for example) influence the
price of a listing?”

Through our exploratory analysis we were able to see that the vast
majority of Airbnb listings are located in Brooklyn and Manhattan which
is not surprising. We are very interested in the prices of different
Airbnb listings, we can see the distribution of prices for each borough
in the second visualization. The highest median price is located in
Manhattan at $150 as well as the largest IQR ($125). The histograms also
show that for each borough, price is very dramatically skewed right with
many outliers.

We then decided to look further into price distribution across
neighborhoods within each borough which showed the highest median price
per neighborhood was Fort Wadsworth, Staten Island which was surprising
because Staten Island overall had one of the lowest median prices. We
plan to continue this analysis to determine what other factors
contribute to fluctuation in prices (such as using latitude and
longitude coordinates to see how distance from central park may affect
prices as well as which neighborhoods are the highest and lowest cost
relative to its borough).

To complete this research we will use the location variables
(neighbourhood, neighbourhood\_group, and latitude longitude) as
predictors variables and price will be the response variable. It is
hypothesized that Airbnbs closer to Manhattan are expected, on average,
to be listed for a higher price.

In addition to location and price, we wanted to explore which factors
made certain Airbnb listings more popular than others. We decided that
the best way to measure how popular a listing is would be to look at the
availability variable. This variable shows the average number of days in
a year that the listing is available to be booked, those that are most
popular would be available for less days than those that are least
popular. Using these assumptions we came up with our second research
question: “How does the way in which a property is listed (type of room,
for example) influence the availability of a listing?”

When looking at a histogram of the availability variable (the fourth
visualization), we can clearly see that the majority of listings are
only available from 0-50 days out of the year and the distribution was
heavily skewed right. Looking at how the variable “room\_type” affects
the availability of the listing shows that rooms that are considered
“shared rooms” have the largest IQR and highest median availability.
Initially, it is hypothesized that a shared room is likely the least
desired of the three room types and therefore has the most availability.

To complete this analysis we would use the room type, minimum nights and
price as predictor variables and availability as the response variable
to represent popularity.

In addition to this analysis we would be interesting in doing a text
analysis of the description of the listing to determine if there were
any trends in the description that could correlate to availability.

### Section 4. Data

Used the glimpse function to show information about and part of abnb:

``` r
glimpse(abnb)
```

    ## Observations: 48,895
    ## Variables: 16
    ## $ id                             <dbl> 2539, 2595, 3647, 3831, 5022, 509…
    ## $ name                           <chr> "Clean & quiet apt home by the pa…
    ## $ host_id                        <dbl> 2787, 2845, 4632, 4869, 7192, 732…
    ## $ host_name                      <chr> "John", "Jennifer", "Elisabeth", …
    ## $ neighbourhood_group            <chr> "Brooklyn", "Manhattan", "Manhatt…
    ## $ neighbourhood                  <chr> "Kensington", "Midtown", "Harlem"…
    ## $ latitude                       <dbl> 40.64749, 40.75362, 40.80902, 40.…
    ## $ longitude                      <dbl> -73.97237, -73.98377, -73.94190, …
    ## $ room_type                      <chr> "Private room", "Entire home/apt"…
    ## $ price                          <dbl> 149, 225, 150, 89, 80, 200, 60, 7…
    ## $ minimum_nights                 <dbl> 1, 1, 3, 1, 10, 3, 45, 2, 2, 1, 5…
    ## $ number_of_reviews              <dbl> 9, 45, 0, 270, 9, 74, 49, 430, 11…
    ## $ last_review                    <date> 2018-10-19, 2019-05-21, NA, 2019…
    ## $ reviews_per_month              <dbl> 0.21, 0.38, NA, 4.64, 0.10, 0.59,…
    ## $ calculated_host_listings_count <dbl> 6, 2, 1, 1, 1, 1, 1, 1, 1, 4, 1, …
    ## $ availability_365               <dbl> 365, 355, 365, 194, 0, 129, 0, 22…
