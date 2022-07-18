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

## Example Workflow

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
repo_path <- create_repo(repo_name)

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

### GitHub Clinic - Generate Markdown Files for Each Participant

Clone the Cohort Repo to RStudio, then run the following code. Detailed instructions of what this looks like:

1. Open RStudio, and create a new script (temporary, you'll deleted it but it's a nicer place to work)
1. Copy the following into the script, then delete the example "first, last, Erin, Julie".
2. Go to the ParticipantsList, and copy the 2 first and last columns
3. Back in RStudio, put your cursor inside the "tribble" parentheses, then, in the Addin menu in the top of RStudio, select "Paste as Tribble"!
4. Then, double-check the column headers - they are likely not `first` and `last` as is written in the `kyber::short_names` call. The easiest thing is to update the column names in the `kyber::short_names` code before running (for example: `kyber::short_names(cohort$First.Name, cohort$Last.Name)`


```
library(stringr)
library(datapasta) # install.packages("datapasta")
library(kyber) ## remotes::install_github("openscapes/kyber")
library(here)

## use `datapasta` addin to vector_tribble these names formatted from the spreadsheet!
cohort <- c(tibble::tribble(
                      ~first,             ~last,
                      "Erin",        "Robinson",
                      "Julie",         "Lowndes",
               )
)

kyber::short_names(cohort$first, cohort$last) |>
   create_github_clinic(here())
```

You'll now have .md files for each participant in the cohort! Any duplicate names with have a `_LastInitial`. Now commit and push the Markdown files to to GitHub.com. Don't push the .gitignore or .rproj since they're not relevant for the Clinic. (You can do Command-A to select all files and then unclick those 2 you don't want).


### Create GitHub team, add usernames

1. Open RStudio, and create a new script (temporary, you'll deleted it but it's a nicer place to work)
1. Paste the following in it and review the code. You may already have a GitHub PAT set; there is more information at the top of the README about it. 
1. Run this code first as-is with the example usernames in the `members` variable to check -
1. Check that the example usernames were added in the Cohort GitHub: go to github.com/openscapes/cohort-name > Settings > Collaborators and Teams 
1. If the team was created with the username and appears in the repo, woohoo! 
1. Open the ParticipantsList and copy the GitHub username column, including the header. 
3. In RStudio, put your cursor after `members <-` and use the `datapasta` Addin > Paste As Tribble to paste the usernames into the `members` variable, deleting the previous example user. 
2. After pasting in the R script, rename the header as "username" (no spaces or asterices)
5. Run the following code again and check that everyone was added!
6. Finally, in the ParticipantList, highlight all usernames we've added in green. This helps us know who else to add during the live clinic session.

``` r
## First make sure you have a GitHub PAT set. If you need one, here's what you'd do:
# usethis::create_github_token() ## use their defaults plus `admin:org`
# Sys.setenv(GITHUB_PAT = "ghp_0id4zkO4GqSuEsC6Zs22wf34Y0u3270") 

library(kyber) 
library(rmarkdown)
library(tibble)
library(fs)
library(datapasta)

## create naming for GitHub - do this once!
repo_name <- "2022-nasa-champions"
team_name <- paste0(repo_name, "-cohort")
create_team(team_name, maintainers = "jules32", org = "openscapes")

## create member variable - do this twice (first as test!)
## this is where you'll use datapasta and run everything below
members <- tibble::tribble(
     ~username,
     "eeholmes",
  )

add_team_members(team_name, members = members$username, org = "openscapes")
add_repo_to_team(repo_name, team_name, org = "openscapes")
```

Yay! Now all to do is to highlight the usernames in green in the ParticpantList for our bookkeeping!


### Agendas

    kyber::call_agenda(
        registry_url = "https://docs.google.com/spreadsheets/d/1Oej46BMX_SLIc1cwoyLHzNWwGlT3szez8FDKc3b418w/edit#gid=942365997", 
        cohort_id = "2022-nasa-champions", 
        call_number = 3)

Then, to move this to a Google Doc and fine-tune formatting, follow these notes (as of July 14, 2022): 
- In RStudio, Knit (or PreviewHTML) the resulting `agenda.md` and copy-paste the result into a Google Doc (example: 03_CallAgenda [ 2022-noaa-afsc ]. You might need to to expand the knitted preview into the browser to get it to copy/paste correctly into Google Docs
- Move Google Doc to Openscapes Workspace folder Openscapes_CohortCalls [ year-cohort-name ].
- Select all (cmd-A) and change font to Open Sans
- (specific to Call 3 Agenda:) Make "_opening.Rmd" part 9 point font 
- Made Header 1 font 18, bold; update heading 1 to match (see screenshot below)
- Made Header 2 font 14, bold; update heading 2 to match
- Select all (cmd-A), then: 
  - "add space" then "remove space **after** paragraph" throughout to make spacing a little more cozy (yes seems odd to do and undo, but it works)
  - "add space" then "remove space **before** paragraph" throughout to make spacing a little more cozy
- Review doc and fix any further font weirdness
- Add page numbers 

**How to "update heading 1 to match"**: In Google Doc, to update a text style (headings, normal text, with font type, size etc), highlight a section with the style you want, click the styles dropdown shown in the screenshot, and select e.g. "Update Heading 1 to match". Double check the doc because we've noticed it missed some in an agenda.

![Screen Shot 2022-07-14 at 5 50 28 PM](https://user-images.githubusercontent.com/11927811/179125336-ec2fc1e5-792e-495d-8a29-c1ed3ec3cdc4.png)
