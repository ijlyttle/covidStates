projthis demo: COVID-19 cases in US
================
Compiled at 2021-01-06 08:21:48 UTC

``` r
here::i_am("README.Rmd", uuid = "11c1d2d6-6424-429e-9312-e14f7b7b1e05")

# function to get path to previous data: path_source("99-publish", "sample.csv")
path_source <- projthis::proj_path_source("README")
```

This purpose of this document is to show how to build a
[projthis](https://ijlyttle.github.io/projthis/) workflow. The
particular case is to create a workflow that automates a daily update of
COVID-related graphics.

``` r
library("conflicted")
```

## Workflow

In the context of the projthis package, a *workflow* consists of a
folder with:

  - an ordered set of RMarkdown files.
  - a `data` directory with a subdirectory dedicated to each RMarkdown
    file, only *that* RMarkdown file can write to its subdirectory.

The function `prothis::proj_workflow_render()` is used to run an entire
workflow, by rendering all the RMarkdown files in alphabetical order.

This particular workflow is composed of five steps:

  - [`00-import`](00-import.md): Import data from external sources.
  - [`01-clean`](01-clean.md): Clean and wrangle data into a consistent
    format.
  - [`02-analyze`](02-analyze.md): Build some maps.
  - [`99-publish`](99-publish.md): Create “externally-available” data.
  - [`README`](README.md): Show the highlights.

There is no fixed rule for how many steps are in a workflow; however, an
“earlier” RMarkdown file shall not read from the data directory of a
“later” RMarkdown file.

I have a personal habit of including `00-import`, `99-publish`, and
`README` in all of the workflows I create. In the following sections,
I’ll walk through the process of building the each of the steps - with
a focus on what the projthis package provides.

## Procedure

This directory was created in the [main procedure](../README.md), by
calling:

``` r
proj_use_workflow("workflow", git_ignore_data = FALSE)
```

1.  This creates a `README.Rmd` from a template, which is then edited
    and knit. At this point, the workflow directory is in [this
    state](https://github.com/ijlyttle/covidStates/tree/create-workflow/workflow).

2.  With *this* file open in the RStudio IDE, create a new RMarkdown
    file from a workflow template using:
    
    ``` r
    projthis::proj_workflow_use_rmd("00-import")
    ```
    
    The newly-created file looks like
    [this](https://github.com/ijlyttle/covidStates/blob/5acbfc5bc1c898c1210455f2c921732e100069a7/workflow/00-import.Rmd).
    
    In the YAML metadata, you’ll see:
    
    ``` yaml
    title: "00-import"
    date: "Compiled at `r format(Sys.time(), '%Y-%m-%d %H:%M:%S', tz = 'UTC')` UTC"
    output: github_document
    params:
      name: "00-import" # change if you rename file
    ```
    
    You’ll almost certainly want to change the `title`. It can be useful
    for the compiled Markdown file to have the `date` when it was
    compiled. We are using a `github_document` because you can share it
    easily and securely on GitHub; it is
    [browseable](https://happygitwithr.com/workflows-browsability.html).
    You can make it private, restricting who sees it if need be. We are
    using the [parmeterized
    reports](https://bookdown.org/yihui/rmarkdown/parameterized-reports.html)
    feature of RMarkdown; it is important that `params$name` be
    **identical** to the sans-extension name of the RMarkdown file. If
    you change the file name, you will need to change this as well.
    
    In the first code-chunk, you’ll see:
    
    ``` r
    here::i_am(paste0(params$name, ".Rmd"), uuid = "f8c9b430-542e-4eaa-b315-bad86866aa06")
    ```
    
    This is the first reason that `params$name` is important -
    `here::i_am()` verifies that it is being called from a directory
    that:
    
      - contains a file named what you claim it is named.
      - that file contains the `uuid` in its text.
    
    These make sure that you are running the code from where you
    *expect* to be running the code. Furthermore, it establishes the
    *root directory* as the directory containing this file. This becomes
    important in a following code chunk:
    
    ``` r
    # create or *empty* the target directory, used to write this file's data: 
    projthis::proj_create_dir_target(params$name)
    
    # function to get path to target directory: path_target("sample.csv")
    path_target <- projthis::proj_path_target(params$name)
    
    # function to get path to previous data: path_source("00-import", "sample.csv")
    path_source <- projthis::proj_path_source(params$name)
    ```
    
    The call to `proj_create_dir_target()` creates a directory called
    `data/00-import`, named for this file. If this directory already
    exists, it is deleted then created anew, empty. Either way, the
    process begins with an empty directory.
    
    All of the data written from `00-import.Rmd` shall be written to
    `data/00-import`. We call this directory the **target directory** -
    no other RMarkdown file shall write to it.
    
    To help enforce this edict, a couple of helper functions are
    created: `path_source()` and `path_target()`. These functions help
    you make sure you’re reading data from the right place, and writing
    data to the right place. For example, `path_target("sample.csv")` to
    return the path needed to write `sample.csv` into the target
    directory. You can use `path_source()` to return the path for data
    written to “earlier” data directories; it makes sure that the path
    you provide is, indeed, “earlier” than the current file.
    
    Using these path-accessor functions helps keep you from “falling off
    the boat”.
    
    The final section in the template prints out a file-listing of the
    target directory, once you have finished writing all your files to
    it:
    
    ``` r
    proj_dir_info(path_target())
    ```
    
    This is a wrapper to `fs::dir_info()`, which:
    
      - shows the files from the perspective of the target directory.
      - returns only a small number of (hopefully useful) columns.
      - shows datetimes using UTC.

3.  Next, we build out the `00-import` file; its purpose is to import
    all of the data this workflow will use.
    
    We get the [daily COVID data for US
    states](https://github.com/nytimes/covid-19-data/blob/master/us-states.csv)
    from the New York Times. We also get an estimate of the [population
    for the US
    states](https://github.com/JoshData/historical-state-population-csv/blob/primary/historical_state_population_by_year.csv)
    from [Josh Tauberer](https://github.com/JoshData).
    
    At this point, I update the project dependencies:
    
    ``` r
    projthis::proj_update_deps()
    ```
    
    The result is that DESCRIPTION file now contains:
    
        Imports: 
         rmarkdown,
         conflicted,
         here,
         projthis
    
    Until such time as the projthis package will be on CRAN, we also
    have to add a `Remotes:` field:
    
        Remotes:
         ijlyttle/projthis
    
    You can see how the
    [`00-import.md`](https://github.com/ijlyttle/covidStates/blob/workflow-import/workflow/00-import.md)
    file looks at this point.

4.  Create and build `01-clean.Rmd`; the goal is to pare down the
    impoerted datasets to what we need. Like before:
    
    ``` r
    projthis::proj_workflow_use_rmd("01-clean")
    ```
    
    Develop some code in the file, then:
    
    ``` r
    projthis::proj_update_deps()
    ```
    
    You can see how the
    [`01-clean.md`](https://github.com/ijlyttle/covidStates/blob/workflow-clean/workflow/01-clean.md)
    file looks at this point.
    
    This procedure is repeated for:
    
      - `02-analyze.Rmd`, where we put together our maps
        ([link](https://github.com/ijlyttle/covidStates/blob/workflow-analyze/workflow/02-analyze.md)
        to compiled file).
      - `99-publish.Rmd`, where the data and materials are made
        available
        ([link](https://github.com/ijlyttle/covidStates/blob/workflow-publish/workflow/99-publish.md)
        to compiled file).
    
    In my other workflows, I use a `99-publish.Rmd` file to:
    
      - export “finished product” to an external file service, like Box.
      - create [external data for
        packages](https://r-pkgs.org/data.html).
      - keep data in a “known” place inside this workflow for other
        files to import. This is how this [project’s README
        file](https://raw.githubusercontent.com/ijlyttle/covidStates/master/README.md)
        accesses the plots.

5.  This entire workflow can be run using:
    
    ``` r
    projthis::proj_workflow_render("workflow")
    ```
