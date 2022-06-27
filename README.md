# kyber <a href="https://openscapes.github.io/kyber/"><img src="man/figures/logo.png" align="right" height="138" /></a>

<!-- badges: start -->

[![R-CMD-check](https://github.com/Openscapes/kyber/workflows/R-CMD-check/badge.svg)](https://github.com/Openscapes/kyber/actions)

<!-- badges: end -->

## Installation

You can install Kyber using the `remotes` package:

``` r
remotes::install_github("openscapes/kyber@main")
```

## Overview

Kyber contains tools for setting up learning cohorts on GitHub, purpose-built 
for the [Openscapes Champions Program](https://www.openscapes.org/champions/).

- `init_repo()` initializes a local git repository and new GitHub repository
with a README.Rmd, a code of conduct, and a gitignore.
- `create_github_clinic()` creates all of the files you need for teaching a
GitHub clinic.
- `create_team()` creates a new team on GitHub.
- `add_team_members()` adds members to a team on GitHub based on their GitHub
usernames.
- `add_repo_to_team()` adds a GitHub repository to a team on GitHub.
- `call_agenda()` creates agenda documents for each individual Cohort Call.

## Quick Cohort Setup

### Configuration

Using Kyber requires more configuration than most R packages since Kyber
functions automate processes on GitHub that you would normally do by hand.

First, make sure that you have `googlesheets4` installed and that you have
authorized your computer to read from Google Sheets. Run the following to 
test your configuration settings:

``` r
library(googlesheets4)

cohort_registry_url <- "https://docs.google.com/spreadsheets/d/1Oej46BMX_SLIc1cwoyLHzNWwGlT3szez8FDKc3b418w/"

read_sheet(cohort_registry_url, sheet = "test-sheet")
```

``` r
library(usethis)
library(gitcreds)

create_github_token(
  scopes = c("repo", "user", "gist", "workflow", "admin:org")
)

gitcreds_set()
```

<!---

This workflow often happens in 4 separate stages:

1.  create the repo and readme (pre-cohort)
2.  create `github-clinic` files (days before GitHub Clinic in Call 2)
3.  create github team and add usernames (day before Clinic, when we
    have all usernames)
4.  create agenda documents before each Cohort Call

For creating the GitHub Team and adding usernames, Kyber requires you to
set up a GitHub Personal Access Token with scopes for **repo** and
**admin:org**. See the [GitHub PAT
documentation](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)
for more information about how to generate your PAT. You can create your PAT with `usethis::create_github_token()` with their defaults, plus `admin:org`.
Please make sure
that you do not share your PAT or commit it to a Git repository, since
anyone with your PAT can act as you on GitHub.

### Create GitHub repo

``` r
library(kyber) 
library(rmarkdown)
library(tibble)
library(fs)

repo_name <- "2021-ilm-rotj"

# This will open a README.Rmd for you to edit
repo_path <- ky_create_repo(repo_name)

# Then render the README.Rmd to README.md
render(path(repo_path, "README.Rmd"))

# We still need to work out the next part of the workflow and the extent to
# which it should be automated, but I imagine something like:
#
# 1. Move README.Rmd out of this repository to another repository that perhaps 
# only contains README.Rmds.
# 2. Git add, commit, and push in the repo that only contains README.Rmds.
# 3. Git add, commit, and push in this repository.
```

### GitHub Clinic

Clone then run this in the cohort's repo:

    library(stringr)
    library(datapasta) # install.packages("datapasta")
    library(kyber) ## remotes::install_github("openscapes/kyber")

    ## use `datapasta` addin to vector_paste these names formatted from the spreadsheet!
    cohort <-c(tibble::tribble(
                         ~first,             ~last,
                         "Erin",        "Robinson",
                        "Julie",         "Lowndes",
                  )
    )

    ky_short_names(cohort$first, cohort$last) |>
      ky_create_github_clinic(here())

### Create GitHub team, add usernames

``` r
# First make sure to set your GitHub PAT
usethis::create_github_token()
## use their defaults plus `admin:org`
Sys.setenv(GITHUB_PAT = "ghp_0id4zkO4GqSuEsC6Zs22wf34Y0u3270") # must do this each R session

library(kyber) 
library(rmarkdown)
library(tibble)
library(fs)

members <- tibble::tribble(
          ~username,
     "jules32",
       "erinmr"
  )


repo_name <- "2022-nasa-champions"
team_name <- paste0(repo_name, "-cohort")

ky_create_team(team_name, maintainers = "jules32", org = "nasa-openscapes")
ky_add_team_members(team_name, members = members$username, org = "nasa-openscapes")
ky_add_repo_to_team(repo_name, team_name, org = "nasa-openscapes")
```

### Agendas

    kyber::call_agenda(
        registry_url = "https://docs.google.com/spreadsheets/d/1Oej46BMX_SLIc1cwoyLHzNWwGlT3szez8FDKc3b418w/edit#gid=942365997", 
        cohort_id = "2022-nasa-champions", 
        call_number = 3)

### With Erin, April 25

```{r}
devtools::load_all()
library(kyber)
```

--->
