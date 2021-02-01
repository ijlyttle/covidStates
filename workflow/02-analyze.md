Analyze data
================
Compiled at 2021-02-01 08:16:15 UTC

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
```

    ## 
    ## Attaching package: 'glue'

    ## The following object is masked from 'package:dplyr':
    ## 
    ##     collapse

``` r
conflict_prefer("filter", "dplyr")
```

    ## [conflicted] Removing existing preference

    ## [conflicted] Will prefer dplyr::filter over any other package

``` r
conflict_prefer("lag", "dplyr")
```

    ## [conflicted] Will prefer dplyr::lag over any other package

``` r
# create or *empty* the target directory, used to write this file's data: 
projthis::proj_create_dir_target(params$name)

# function to get path to target directory: path_target("sample.csv")
path_target <- projthis::proj_path_target(params$name)

# function to get path to previous data: path_source("00-import", "sample.csv")
path_source <- projthis::proj_path_source(params$name)
```

## Read data

``` r
population <- 
  read_csv(
    path_source("01-clean", "population.csv"),
    col_types = cols(state = "c", population = "d")
  ) %>%
  print()
```

    ## # A tibble: 51 x 2
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

    ## # A tibble: 17,106 x 5
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
    ## # … with 17,096 more rows

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

    ## # A tibble: 17,106 x 12
    ## # Groups:   state [51]
    ##    date       state cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    <date>     <chr>       <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 2020-01-21 Wash…           1            0.013             NA               NA
    ##  2 2020-01-22 Wash…           1            0.013             NA               NA
    ##  3 2020-01-23 Wash…           1            0.013             NA               NA
    ##  4 2020-01-24 Illi…           1            0.008             NA               NA
    ##  5 2020-01-24 Wash…           1            0.013             NA               NA
    ##  6 2020-01-25 Cali…           1            0.003             NA               NA
    ##  7 2020-01-25 Illi…           1            0.008             NA               NA
    ##  8 2020-01-25 Wash…           1            0.013             NA               NA
    ##  9 2020-01-26 Ariz…           1            0.014             NA               NA
    ## 10 2020-01-26 Cali…           2            0.005             NA               NA
    ## # … with 17,096 more rows, and 6 more variables: cases_week_growth <dbl>,
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

    ## # A tibble: 51 x 7
    ## # Groups:   state [51]
    ##    date       state cases_total cases_total_per… cases_avg_week cases_avg_week_…
    ##    <date>     <chr>       <dbl>            <dbl>          <dbl>            <dbl>
    ##  1 2021-01-30 Ariz…      756027           10387.          5491.             75.4
    ##  2 2021-01-30 Sout…      439785            8542.          3714.             72.1
    ##  3 2021-01-30 Rhod…      114438           10803.           672.             63.4
    ##  4 2021-01-30 New …     1408955            7243.         12198              62.7
    ##  5 2021-01-30 Texas     2362817            8149.         17239.             59.5
    ##  6 2021-01-30 Okla…      386590            9770.          2349.             59.4
    ##  7 2021-01-30 Geor…      881740            8305.          6240.             58.8
    ##  8 2021-01-30 New …      692543            7797.          5001              56.3
    ##  9 2021-01-30 Virg…      502221            5884.          4795.             56.2
    ## 10 2021-01-30 Kent…      365016            8170.          2425.             54.3
    ## # … with 41 more rows, and 1 more variable: cases_week_growth <dbl>

``` r
covid_recent_deaths <- 
  covid_week %>%
  filter(date == max(date)) %>%
  select(date, state, starts_with("deaths")) %>%
  arrange(desc(deaths_avg_week_per100k)) %>%
  print()
```

    ## # A tibble: 51 x 7
    ## # Groups:   state [51]
    ##    date       state deaths_total deaths_total_pe… deaths_avg_week
    ##    <date>     <chr>        <dbl>            <dbl>           <dbl>
    ##  1 2021-01-30 Alab…         7566             154.           130. 
    ##  2 2021-01-30 Ariz…        13111             180.           134. 
    ##  3 2021-01-30 Tenn…         9483             139.           107. 
    ##  4 2021-01-30 Sout…         7014             136.            76.4
    ##  5 2021-01-30 Cali…        40694             103.           554. 
    ##  6 2021-01-30 Arka…         4838             160.            39.3
    ##  7 2021-01-30 Miss…         6018             202.            38  
    ##  8 2021-01-30 Sout…         1775             201.            11.3
    ##  9 2021-01-30 Geor…        13776             130.           135. 
    ## 10 2021-01-30 Penn…        21658             169.           162. 
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
    legend.text.align = 1 # right-justify
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
    legend.text.align = 1 # right-justify
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

    ## # A tibble: 5 x 4
    ##   path                    type         size modification_time  
    ##   <fs::path>              <fct> <fs::bytes> <dttm>             
    ## 1 cases.png               file      354.66K 2021-02-01 08:16:21
    ## 2 change.png              file      334.94K 2021-02-01 08:16:21
    ## 3 covid_recent_cases.csv  file        3.44K 2021-02-01 08:16:20
    ## 4 covid_recent_deaths.csv file        3.25K 2021-02-01 08:16:20
    ## 5 covid_week.csv          file        1.65M 2021-02-01 08:16:20
