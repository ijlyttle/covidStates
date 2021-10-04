Analyze data
================
Compiled at 2021-10-04 08:13:45 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "a4069103-4402-4559-ba03-cca3df086442")
```

The purpose of this document is to create some state-based maps that
show the current trajectory of COVID-19 cases. There will be two maps:

  - seven-day average of newly-reported cases
  - change in newly-reported cases vs. previous seven days

<!-- end list -->

``` r
library("conflicted")
library("readr")
library("dplyr")
library("albersusa")
library("ggplot2")
library("glue")

conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Will prefer dplyr::filter over any other package

``` r
conflict_prefer("lag", "dplyr")
```

    ## [conflicted] Will prefer dplyr::lag over any other package

``` r
# create or *empty* the target directory, used to write this file's data: 
projthis::proj_create_dir_target(params$name, clean = TRUE)

# function to get path to target directory: path_target("sample.csv")
path_target <- projthis::proj_path_target(params$name)

# function to get path to previous data: path_source("00-import", "sample.csv")
path_source <- projthis::proj_path_source(params$name)
```

    ## ℹ Reading workflow configuration from '/Users/runner/work/covidStates/covidStates/workflow/_projthis.yml'

## Read data

``` r
population <- 
  read_csv(
    path_source("01-clean", "population.csv"),
    col_types = cols(state = "c", population = "d")
  ) %>%
  print()
```

    ## # A tibble: 51 × 2
    ##    state                population
    ##    <chr>                     <dbl>
    ##  1 Alaska                   731545
    ##  2 Alabama                 4903185
    ##  3 Arkansas                3017804
    ##  4 Arizona                 7278717
    ##  5 California             39512223
    ##  6 Colorado                5758736
    ##  7 Connecticut             3565287
    ##  8 District of Columbia     705749
    ##  9 Delaware                 973764
    ## 10 Florida                21477737
    ## # … with 41 more rows

``` r
covid <- 
  read_csv(
    path_source("01-clean", "covid.csv"),
    col_types = cols(
      date = "D", 
      state = "c", 
      fips = "c", 
      cases = "d", 
      deaths = "d"
    )
  ) %>%
  print()
```

    ## # A tibble: 29,652 × 5
    ##    date       state      fips  cases deaths
    ##    <date>     <chr>      <chr> <dbl>  <dbl>
    ##  1 2020-01-21 Washington 53        1      0
    ##  2 2020-01-22 Washington 53        1      0
    ##  3 2020-01-23 Washington 53        1      0
    ##  4 2020-01-24 Illinois   17        1      0
    ##  5 2020-01-24 Washington 53        1      0
    ##  6 2020-01-25 California 06        1      0
    ##  7 2020-01-25 Illinois   17        1      0
    ##  8 2020-01-25 Washington 53        1      0
    ##  9 2020-01-26 Arizona    04        1      0
    ## 10 2020-01-26 California 06        2      0
    ## # … with 29,642 more rows

## Wrangle data

``` r
growth <- function(x) {
  # week-over-week growth, normalized by previous week
  
  # use "+ 1" in denominator to avoid division by zero
  result <- (x - dplyr::lag(x, 7)) / (dplyr::lag(x, 7) + 1) 
  
  # three digits should be more than enough
  round(result, 3)
}

per_100k <- function(x, pop) {
  result <- x / (pop / 1e5)
  
  round(result, 3)
}

covid_week <-
  covid %>%
  left_join(population, by = "state") %>%
  group_by(state) %>%
  arrange(date) %>%
  transmute(
    date,
    state,
    cases_total = cases, 
    cases_total_per100k = per_100k(cases_total, population),
    cases_avg_week = (cases - lag(cases_total, 7)) / 7,
    cases_avg_week_per100k = per_100k(cases_avg_week, population),
    cases_week_growth = growth(cases_avg_week),
    deaths_total = deaths,
    deaths_total_per100k = per_100k(deaths_total, population),
    deaths_avg_week = (deaths - lag(deaths_total, 7)) / 7,
    deaths_avg_week_per100k = per_100k(deaths_avg_week, population),
    deaths_week_growth = growth(deaths_avg_week)
  ) %>%
  print()
```

    ## # A tibble: 29,652 × 12
    ## # Groups:   state [51]
    ##    date       state      cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    <date>     <chr>            <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 2020-01-21 Washington           1            0.013             NA               NA
    ##  2 2020-01-22 Washington           1            0.013             NA               NA
    ##  3 2020-01-23 Washington           1            0.013             NA               NA
    ##  4 2020-01-24 Illinois             1            0.008             NA               NA
    ##  5 2020-01-24 Washington           1            0.013             NA               NA
    ##  6 2020-01-25 California           1            0.003             NA               NA
    ##  7 2020-01-25 Illinois             1            0.008             NA               NA
    ##  8 2020-01-25 Washington           1            0.013             NA               NA
    ##  9 2020-01-26 Arizona              1            0.014             NA               NA
    ## 10 2020-01-26 California           2            0.005             NA               NA
    ## # … with 29,642 more rows, and 6 more variables: cases_week_growth <dbl>,
    ## #   deaths_total <dbl>, deaths_total_per100k <dbl>, deaths_avg_week <dbl>,
    ## #   deaths_avg_week_per100k <dbl>, deaths_week_growth <dbl>

It might also be useful to have files for the most-recent day, each for
cases and deaths.

``` r
covid_recent_cases <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("cases")) %>%
  arrange(desc(cases_avg_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 × 7
    ## # Groups:   state [51]
    ##    date       state          cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    <date>     <chr>                <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 2021-10-03 Alaska              114867           15702.          1140.            156. 
    ##  2 2021-10-03 Wyoming              91169           15752.           472.             81.5
    ##  3 2021-10-03 North Dakota        133819           17560.           616.             80.9
    ##  4 2021-10-03 Montana             152159           14237.           854.             79.9
    ##  5 2021-10-03 West Virginia       244698           13654.          1414.             78.9
    ##  6 2021-10-03 Idaho               260395           14571.          1341.             75.0
    ##  7 2021-10-03 Kentucky            701264           15696.          3122.             69.9
    ##  8 2021-10-03 South Carolina      860369           16710.          2681.             52.1
    ##  9 2021-10-03 Wisconsin           814364           13987.          2912.             50.0
    ## 10 2021-10-03 Ohio               1429745           12231.          5676.             48.6
    ## # … with 41 more rows, and 1 more variable: cases_week_growth <dbl>

``` r
covid_recent_deaths <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("deaths")) %>%
  arrange(desc(deaths_avg_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 × 7
    ## # Groups:   state [51]
    ##    date       state          deaths_total deaths_total_per100k deaths_avg_week
    ##    <date>     <chr>                 <dbl>                <dbl>           <dbl>
    ##  1 2021-10-03 Idaho                  2965                 166.           30.1 
    ##  2 2021-10-03 West Virginia          3722                 208.           28.4 
    ##  3 2021-10-03 South Carolina        12570                 244.           70   
    ##  4 2021-10-03 Alabama               14470                 295.           64   
    ##  5 2021-10-03 Florida               55300                 257.          246.  
    ##  6 2021-10-03 Mississippi            9646                 324.           31.6 
    ##  7 2021-10-03 Kentucky               8905                 199.           45.3 
    ##  8 2021-10-03 Wyoming                 996                 172.            5.86
    ##  9 2021-10-03 Georgia               25371                 239.          106.  
    ## 10 2021-10-03 Montana                2017                 189.            9.86
    ## # … with 41 more rows, and 2 more variables: deaths_avg_week_per100k <dbl>,
    ## #   deaths_week_growth <dbl>

## Plot data

Let’s make some choropleth maps using
[ggplot2](https://ggplot2.tidyverse.org/).

``` r
map_recent_cases <- 
  usa_sf("laea") %>%
  left_join(covid_recent_cases, by = c(name = "state"))

date <- max(covid_recent_cases$date)

gg_cases <-
  ggplot(map_recent_cases, aes(fill = cases_avg_week_per100k)) +
  geom_sf(size = 0.25, color = "white") + 
  scale_fill_distiller(
    palette = "Oranges", 
    direction = 1,
    limits = c(0, NA)
  ) +
  labs(
    title = glue("Newly-reported COVID-19 cases, seven-day average as of {date}"),
    subtitle = "Data from the New York Times",
    fill = "cases\nper 100k"
  ) +
  theme_void() + 
  theme(
    legend.text.align = 1, # right-justify
    plot.background = element_rect(fill = "#ffffff")
  )

gg_cases
```

![](02-analyze_files/figure-gfm/unnamed-chunk-1-1.png)<!-- -->

``` r
# see https://github.com/tidyverse/ggplot2/issues/3738#issuecomment-583750802
mid_rescaler <- function(mid = 0) {
  function(x, to = c(0, 1), from = range(x, na.rm = TRUE)) {
    scales::rescale_mid(x, to, from, mid)
  }
}

gg_change <-
  ggplot(map_recent_cases, aes(fill = cases_week_growth)) +
  geom_sf(size = 0.25, color = "white") + 
  scale_fill_distiller(
    palette = "PuOr", 
    rescaler = mid_rescaler(),
    labels = scales::label_percent()
  ) +
  labs(
    title = glue("Week-over-week change in reported COVID-19 cases, as of {date}"),
    subtitle = "Data from the New York Times",
    fill = "change"
  ) +
  theme_void() +
  theme(
    legend.text.align = 1, # right-justify
    plot.background = element_rect(fill = "#ffffff")
  )

gg_change
```

![](02-analyze_files/figure-gfm/unnamed-chunk-2-1.png)<!-- -->

## Write data

``` r
write_csv(covid_week, path_target("covid_week.csv"))
write_csv(covid_recent_cases, path_target("covid_recent_cases.csv"))
write_csv(covid_recent_deaths, path_target("covid_recent_deaths.csv"))

ggsave(path_target("cases.png"), plot = gg_cases, width = 7, height = 5)
ggsave(path_target("change.png"), plot = gg_change, width = 7, height = 5)
```

## Files written

These files have been written to `data/02-analyze`:

``` r
projthis::proj_dir_info(path_target())
```

    ## # A tibble: 5 × 4
    ##   path                    type         size modification_time  
    ##   <fs::path>              <fct> <fs::bytes> <dttm>             
    ## 1 cases.png               file      342.57K 2021-10-04 08:13:54
    ## 2 change.png              file      333.32K 2021-10-04 08:13:55
    ## 3 covid_recent_cases.csv  file        3.45K 2021-10-04 08:13:53
    ## 4 covid_recent_deaths.csv file        3.22K 2021-10-04 08:13:53
    ## 5 covid_week.csv          file        2.94M 2021-10-04 08:13:53
