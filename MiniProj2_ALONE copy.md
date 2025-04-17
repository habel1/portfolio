STAT 228: Mini Project 2
================
Zoe P Habel

# Data Cleaning + Wrangling in R

## An example using Alone survivalists

Alone is a survival show which started in the US and has now expanded to
Australia. The premise of the show is for several survival experts to
try to outlast each other in the wilderness with limited survival
supplies and in complete isolation. See the [Wikipedia
page](https://en.wikipedia.org/wiki/Alone_(TV_series)) for further
details on the series.

Statistician Daniel Oehm created 4 excellent data sets containing
various statistics from the TV series. More info about the data sets can
be found on his
[blog](https://gradientdescending.com/alone-r-package-datasets-from-the-survival-tv-series/)
and on the [github repository](https://github.com/doehm/alone)
containing the source code.

Data ‘cleaning’ and ‘wrangling’ refers to the process of preparing data
for analysis. Basically, we want to make it easier to read and analyze !
Today we will be using the `survivalists` data set from the “alone”
package to give a brief overview of the process.

### Exploring the data

Let’s start by loading our packages:

``` r
library(tidyverse)
library(alone)
```

According to the package documentation, the `survivalist` package is *“a
data frame of survivalists across all 9 seasons detailing name and
demographics, location and profession, result, days lasted, reasons for
tapping out (detailed and categorized), page URL.”*

**Let’s take a quick look!**

``` r
head(survivalists)
```

    ## # A tibble: 6 × 20
    ##   version season id    name        first_name last_name   age gender city  state
    ##   <chr>    <dbl> <chr> <chr>       <chr>      <chr>     <dbl> <chr>  <chr> <chr>
    ## 1 AU           1 AU004 Gina Chick  Gina       Chick        52 Female <NA>  New …
    ## 2 AU           1 AU008 Mike Atkin… Mike       Atkinson     45 Male   <NA>  New …
    ## 3 AU           1 AU007 Michael Wa… Michael    Wallace      43 Male   <NA>  New …
    ## 4 AU           1 AU006 Kate Graro… Kate       Grarock      41 Female <NA>  Aust…
    ## 5 AU           1 AU002 Chris Bakon Chris      Bakon        39 Male   <NA>  Tasm…
    ## 6 AU           1 AU003 Duane Byrn… Duane      Byrnes       35 Male   <NA>  New …
    ## # ℹ 10 more variables: country <chr>, result <dbl>, days_lasted <dbl>,
    ## #   medically_evacuated <lgl>, reason_tapped_out <chr>, reason_category <chr>,
    ## #   episode_tapped <dbl>, team <chr>, day_linked_up <dbl>, profession <chr>

We won’t use every variable for our analysis, so here’s info on just a
few:

**Pertinent Variable Descriptions:**

| Variable Name | Description |
|----|----|
| version | Country code for the version of the show |
| season | The season number |
| id | Survivalist ID |
| age | Age of survivalist |
| gender | Gender |
| result | Place the survivalist finished in the season |
| days_lasted | The number of days lasted in the game before tapping out or winning |
| medically_evacuated | Logical. If the survivalist was medically evacuated from the game |
| reason_category | A simplified category of the reason for tapping out |

### Tidying the data for our specific use

Since we’re not interested in analyzing every single variable, **let’s
start by selecting the ones we want to keep using the function
“select()”.**

``` r
survivalist_data <- survivalists |>
  select(version,season,id,age,gender,result,days_lasted,medically_evacuated,reason_category)
head(survivalist_data)
```

    ## # A tibble: 6 × 9
    ##   version season id      age gender result days_lasted medically_evacuated
    ##   <chr>    <dbl> <chr> <dbl> <chr>   <dbl>       <dbl> <lgl>              
    ## 1 AU           1 AU004    52 Female      1          67 FALSE              
    ## 2 AU           1 AU008    45 Male        2          64 TRUE               
    ## 3 AU           1 AU007    43 Male        3          30 FALSE              
    ## 4 AU           1 AU006    41 Female      4          23 FALSE              
    ## 5 AU           1 AU002    39 Male        5          12 FALSE              
    ## 6 AU           1 AU003    35 Male        6          10 FALSE              
    ## # ℹ 1 more variable: reason_category <chr>

Let’s say we only want to look at the US seasons of the show since
there’s more data. **We can use a function called “filter()” to filter
by US seasons.**

``` r
survivalists_US <- survivalist_data |>
  filter(version=="US") |>
  #we don't need the version column anymore, so we're removing it here
  select(!version)
```

The documentation explains that participants who have “NA” in certain
categories relating to elimination are missing values because they were
not eliminated, meaning that “NA” actually means “winner.” This is kind
of confusing! But we can make this a bit clearer by adding a logical
“winner” column. **We’ll use a function called “mutate()” to create a
new column.**

``` r
survivalists_US <- survivalists_US |>
  mutate(winner = if_else(result=="1"&is.na(reason_category),true ="YES",false = "NO")) 
```

*Let’s check that our transformation worked by selecting all
participants in first place & the lowest place*

``` r
first_place <- survivalists_US|>
  slice_min(result)
table(first_place$winner)
```

    ## 
    ## YES 
    ##  12

``` r
#compare to the amount of "YES" winners in the whole set
table(survivalists_US$winner)
```

    ## 
    ##  NO YES 
    ## 102  12

There’s 12 “YES” among those in first place and the same amount in the
whole data set – **looking good!**

### Ready to start analyzing !

Now that we’ve got a nice looking data set, let’s try to find some
patterns!

First, make some plots which show us how many days participants lasted
(`days_lasted`) by whether they won (`winner`) and and their final
standing (`result`).

``` r
ggplot(survivalists_US, aes(y=days_lasted, fill=winner)) +
  geom_boxplot() +
  labs(y="Days Lasted",fill="Season Winner?") +
  theme_classic()
```

![](MiniProj2_ALONE_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
ggplot(survivalists_US, aes(y=result, x=days_lasted,color=winner)) +
  geom_point() +
  labs(x="Days Lasted",y="Final Standing",color="Season Winner?") +
  theme_classic()
```

![](MiniProj2_ALONE_files/figure-gfm/unnamed-chunk-7-2.png)<!-- --> The
boxplots show that every winner in this set lasted over 50 days, which
is just under the third quartile \# of days that those who did not win
lasted. The scatter plot adds additional information about those
outliers, showing that many of those participants who lasted longer made
it further along in the competition!

There’s a lot more that could be explored, but we’ll leave it here for
today! Thank you for following along with my tutorial, I hope you found
this helpful!
