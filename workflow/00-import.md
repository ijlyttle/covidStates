Import Data
================
Compiled at 2020-12-29 21:28:24 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "f8c9b430-542e-4eaa-b315-bad86866aa06")
```

    ## here() starts at /Users/sesa19001/Documents/repos/public/covidStates/workflow

``` r
library("conflicted")
```

The purpose of this document is to import the data we’ll need to make
some COVID-19 maps for the US:

-   [NYT daily state-level
    data](https://github.com/nytimes/covid-19-data/blob/master/us-states.csv)
-   [US state population
    estimates](https://github.com/JoshData/historical-state-population-csv/blob/primary/historical_state_population_by_year.csv)
    from [Josh Tauberer](https://github.com/JoshData).

``` r
# create target directory to write *this* file's data: 
#  - all data written by this file should be written here
proj_create_dir_target(params$name)

# create accessor functions for data directories:
#  - get path to target directory: path_target("sample.csv")
#  - get path to previous data: path_data("00-import", "sample.csv")
path_target <- proj_path_target(params$name)
path_data <- proj_path_data(params$name)
```

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
proj_dir_info(path_target())
```

    ## # A tibble: 2 x 4
    ##   path                  type         size modification_time  
    ##   <fs::path>            <fct> <fs::bytes> <dttm>             
    ## 1 covid-states.csv      file       546.3K 2020-12-29 21:28:29
    ## 2 population-states.csv file        98.8K 2020-12-29 21:28:32
