Import Data
================
Compiled at 2021-03-23 08:13:09 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "f8c9b430-542e-4eaa-b315-bad86866aa06")
```

The purpose of this document is to import the data we’ll need to make
some COVID-19 maps for the US:

  - [NYT daily state-level
    data](https://github.com/nytimes/covid-19-data/blob/master/us-states.csv)
  - [US state population
    estimates](https://github.com/JoshData/historical-state-population-csv/blob/primary/historical_state_population_by_year.csv)
    from [Josh Tauberer](https://github.com/JoshData).

<!-- end list -->

``` r
library("conflicted")
```

``` r
# create or *empty* the target directory, used to write this file's data: 
projthis::proj_create_dir_target(params$name, clean = TRUE)

# function to get path to target directory: path_target("sample.csv")
path_target <- projthis::proj_path_target(params$name)

# function to get path to previous data: path_source("00-import", "sample.csv")
path_source <- projthis::proj_path_source(params$name)
```

## Download

We call `download.file()` to put the files directly into our target
directory, using `path_target()` to specify the path within the target
directory. First, the COVID data:

``` r
download.file(
  "https://raw.githubusercontent.com/nytimes/covid-19-data/master/us-states.csv",
  destfile = path_target("covid-states.csv")
)
```

Next, the US states population data:

``` r
download.file(
  "https://raw.githubusercontent.com/JoshData/historical-state-population-csv/primary/historical_state_population_by_year.csv",
  destfile = path_target("population-states.csv")
)
```

## Files written

These files have been written to `data/00-import`:

``` r
projthis::proj_dir_info(path_target())
```

    ## # A tibble: 2 x 4
    ##   path                  type         size modification_time  
    ##   <fs::path>            <fct> <fs::bytes> <dttm>             
    ## 1 covid-states.csv      file       707.9K 2021-03-23 08:13:10
    ## 2 population-states.csv file        98.8K 2021-03-23 08:13:10
