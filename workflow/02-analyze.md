Analyze data
================
Compiled at 2021-08-08 08:13:21 UTC

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

    ## # A tibble: 26,745 × 5
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
    ## # … with 26,735 more rows

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

    ## # A tibble: 26,745 × 12
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
    ## # … with 26,735 more rows, and 6 more variables: cases_week_growth <dbl>,
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
    ##  1 2021-08-07 Louisiana           573903           12345.          4603.             99.0
    ##  2 2021-08-07 Florida            2725450           12690.         19250.             89.6
    ##  3 2021-08-07 Arkansas            402908           13351.          2351.             77.9
    ##  4 2021-08-07 Mississippi         358149           12034.          2092              70.3
    ##  5 2021-08-07 Alabama             607209           12384.          3086              62.9
    ##  6 2021-08-07 Oklahoma            493983           12484.          1907.             48.2
    ##  7 2021-08-07 Texas              3227480           11131.         13883.             47.9
    ##  8 2021-08-07 South Carolina      634310           12320.          2452.             47.6
    ##  9 2021-08-07 Missouri            715882           11664.          2879.             46.9
    ## 10 2021-08-07 Georgia            1184614           11157.          4446.             41.9
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
    ##    date       state       deaths_total deaths_total_per100k deaths_avg_week
    ##    <date>     <chr>              <dbl>                <dbl>           <dbl>
    ##  1 2021-08-07 Arkansas            6290                 208.           21.3 
    ##  2 2021-08-07 Louisiana          11210                 241.           30.1 
    ##  3 2021-08-07 Nevada              6005                 195.           13.3 
    ##  4 2021-08-07 Missouri           10490                 171.           25.7 
    ##  5 2021-08-07 Florida            39695                 185.           88   
    ##  6 2021-08-07 Mississippi         7621                 256.           11.1 
    ##  7 2021-08-07 Alabama            11624                 237.           12.6 
    ##  8 2021-08-07 Arizona            18376                 252.           18.6 
    ##  9 2021-08-07 Wyoming              786                 136.            1.43
    ## 10 2021-08-07 Montana             1722                 161.            2.57
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
    ## 1 cases.png               file      343.29K 2021-08-08 08:13:28
    ## 2 change.png              file      340.82K 2021-08-08 08:13:28
    ## 3 covid_recent_cases.csv  file        3.43K 2021-08-08 08:13:27
    ## 4 covid_recent_deaths.csv file        3.18K 2021-08-08 08:13:27
    ## 5 covid_week.csv          file        2.64M 2021-08-08 08:13:27
