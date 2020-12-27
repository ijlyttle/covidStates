Publish
================
Compiled at 2020-12-27 01:08:13 UTC

``` r
here::i_am(paste0(params$name, ".Rmd"), uuid = "ec845588-783a-4d74-9389-81c54875c3c3")
```

    ## here() starts at /Users/sesa19001/Documents/repos/public/covidStates/workflow

``` r
library("conflicted")
library("projthis")
library("here")
library("fs")
```

The purpose of this document is to pick-and-choose from the data-files
written earlier in this workflow in order to make them more-widely
available. In other circumstances, this may involve uploading data to an
external service. Here, we will:

-   put CSV files into a directory where we will feel confident they can
    be found going forward.
-   put PNG files into a directory where the parent-project’s README
    file can expect to find them.

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

## Transfer files

``` r
file_copy(path_data("02-analyze", "covid_recent_cases.csv"), path_target())
file_copy(path_data("02-analyze", "covid_recent_deaths.csv"), path_target())
file_copy(path_data("02-analyze", "covid_week.csv"), path_target())
```

## Files written

These files have been written to `data/99-publish`:

``` r
proj_dir_info(path_target())
```

    ## # A tibble: 3 x 4
    ##   path                    type         size modification_time  
    ##   <fs::path>              <fct> <fs::bytes> <dttm>             
    ## 1 covid_recent_cases.csv  file        3.42K 2020-12-27 01:08:13
    ## 2 covid_recent_deaths.csv file        3.19K 2020-12-27 01:08:13
    ## 3 covid_week.csv          file        1.46M 2020-12-27 01:08:13
