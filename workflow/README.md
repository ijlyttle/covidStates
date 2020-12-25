projthis demo: COVID-19 cases in US
================

The purpose of this repository is to show how to use the
[projthis](https://ijlyttle.github.io/projthis/) package to manage a
project-based workflow. The particular case will be to create a workflow
that, ultimately, automates a daily update of COVID-related graphics.

## Procedure

This directory was created in the [main procedure](../README.md).

1.  Edited and knit `README.Rmd`. At this point, the workflow directory
    is in [this
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
    date: "Compiled at 2020-12-25 23:40:30 UTC"
    output: github_document
    params:
      name: "00-import" # change if you rename file
    ```

    You’ll almost certainly want to change the `title`. It can be useful
    for the compiled Markdown file to have the `date` at which it was
    compiled. We are using a `github_document` because you can share it
    easily and securely on GitHub; it is
    [browseable](https://happygitwithr.com/workflows-browsability.html),
    and you can make it private to restrict who sees it, if need be. We
    are using the [parmeterized
    reports](https://bookdown.org/yihui/rmarkdown/parameterized-reports.html)
    feature of RMarkdown; it is important `params$name` be **identical**
    to the sans-extension name of the RMarkdown file. If you change the
    filename, you will need to change this as well.

    In the first code-chunk, you’ll see:

    ``` r
    here::i_am(paste0(params$name, ".Rmd"), uuid = "f8c9b430-542e-4eaa-b315-bad86866aa06")
    ```

    This is the first reason that `params$name` is important -
    `here::i_am()` verifies that it is being called from a directory
    that has:

    -   a file named what you claim it is named.
    -   that file has contains the `uuid` in its text.

    These makes sure that you are running the code from where you
    *expect* to be running the code. Furthermore, it establishes the
    \*root directory\*\* as the directory containing this file. This
    becomes important in the next code chunk:

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

    The call to `proj_create_dir_target()` creates a directory (relative
    to the *root*) called `data/00-import`, named for this file. All of
    the data written from `00-import.Rmd` shall be written to
    `data/00-import`. We call this directory the **target directory** -
    no other RMarkdown file shall write to it.

    To help enforce this edict, a couple of helper functions are
    created: `path_target()` and `path_data()`. For example,
    `path_target("sample.csv")` to return the path needed to write
    `sample.csv` into the target directory. You can use `path_data()` to
    return the path for data written to “earlier” data directories; it
    makes sure that the path you provide is, indeed, “earlier” than the
    current file.

    The final section in the template prints out a file-listing of the
    target directory, once you have finished writing all your files to
    it:

    ``` r
    proj_dir_info(path_target())
    ```

    This is a wrapper to `fs::dir_info()`, which:

    -   shows the files from the perspective of the target directory.
    -   returns only a small number of (hopefully useful) columns.
    -   shows datetimes using UTC.

3.  Next, we build out the `00-import` file. We get the [daily COVID
    data for US
    states](https://github.com/nytimes/covid-19-data/blob/master/us-states.csv)
    from the New York Times. We also get an estimate of the [population
    for the US
    states](https://github.com/JoshData/historical-state-population-csv/blob/primary/historical_state_population_by_year.csv)
    from [Josh Tauberer](https://github.com/JoshData).

    At this point, I update the project dependencies:

    ``` r
    proj_update_deps()
    ```

    The result is that DESCRIPTION file now contains:

        Imports: 
         rmarkdown,
         conflicted,
         here,
         projthis

    Until such time as {projthis} will be on CRAN, we also have to add a
    `Remotes:` field:

        Remotes:
         ijlyttle/projthis

## File structure

    data/
    00-import.md
    00-import.Rmd
    README.md
    README.Rmd

-   `data/` directory to contain data written by RMarkdown files (empty,
    not yet committed to git).
-   `00-import.md` Markdown version, viewable on GitHub.
-   `00-import.Rmd` RMarkdown source for importing files.
-   `REAMDE.md` Markdown version of README.
-   `README.Rmd` RMarkdown source for REAMDE.
