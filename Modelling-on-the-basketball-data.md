---
title: "Modelling on the basketball data"
author: "Xucheng XIE"
date: "23/04/2022"
output:
  html_document:
    keep_md: yes
    theme: cerulean
    highlight: haddock
---



## 1. Introduction

### a) Relevant background information of basketball 

Basketball is the third most popular sport in the world, with 2.2 billion fans worldwide.[1] There are two popular types of basketball games, 3x3 and 5x5. Unlike 5x5, which is played on a full-scale basketball court, 3x3 is played on a half-court.[2] The National Basketball Association (NBA) is a men's professional Basketball league consisting of 30 professional teams in North America. It is one of the four major professional sports leagues in the United States. [3]

#### Position requirements

According to the NBA's rulebook, each team consists of five players, and each player will be assigned a specific position, namely, center, power forward, small forward, point guard and shooting guard.[4] Shane (2018) suggests that the center is the heart of formation, with the point guard ranked second.[5] The center is the tallest player in a team, being responsible for rim protection, while the point guard is the shortest player in a team, playing a vital role in running the offence, and that is the reason why the point guard is sometimes called the "floor general" by the masses.

<img src="images/Basketball_Positions.png" width="50%" style="display: block; margin: auto;" />

#### Key metrics

Basketball statistics have been kept to evaluate players' and teams' performance. Some typical examples of basketball statistics include `points (PTS)`, `efficiency (EFF)`, `steals (STL)`, `rebounds (REB)` and `field goal attempt (FGA)`. Haefner (2022) highlights the four most important stats that should be paid attention to are `field Goal attempts (FGA)`, `effective field goal percentage (EFG%)`, `free throw attempts (FTA)` and `free throw percentage (FT%)`. All these four ratios indicate the chance of winning; the higher the ratios, the higher the possibility of winning the game.[6] Sampaio et al.(2006) use discriminant analysis to identify factors driving a team's long-term success.[7] The result shows that `successful free throws (FT)`, `rebounds (REB)`, `assists (AST)`, `steals (STL)` and `blocks (BLK)` are key differences between the best and worst teams. Fayad (2020) scrapes data from [basketball-reference.com](https://www.basketball-reference.com/) and builds a model using the random forest algorithm. He suggests that the team's `net rating (NRtg)` and `home-court advantage` also greatly influence wins.[8]

### b) Description of the scenario

Chicago Bulls are an American professional basketball team headquartered in Chicago. In the most recent season (2018-19), the team was ranked 27th for overall performance and 26th for payroll budget out of thirty teams. In the new season (2019-20), Chicago Bulls will be assigned $118 million for players' contracts.

### c) The aim of the project

The project aims to help Chicago Bulls make the best use of the payroll budget and find the best starting player for each position.

### d) Justification and importance

Players are decisive to a team's long-term success, so it is crucial for managers to find suitable players for their teams. A team that owns players with higher ratings are more likely to win the game. However, some players are unaffordable due to the limited payroll budget, so managers need to balance the team's strength and funding, ensuring every buck is well spent.

## 2. Reading and cleaning the raw data

#### Description of datasets and variables

Five datasets have been used for data analysis, which are  `“2018-19_nba_player-salaries”`, `“2018-19_nba_player-statistics”`, `“2018-19_nba_team-statistics_1”`, `“2018-19_nba_team-statistics_2”` and `“2018-19_nba_team-payroll”`.
<p>&nbsp;</p>

For a detailed description of datasets, click [here](README.txt).
<p>&nbsp;</p>

**<span style="color:red">2018-19_nba_player-salaries</span>**

This dataset contains statistics of individual NBA players, including `Field Goals (FG)`, [Effective Field Goal Percentage (eFG%)](https://en.wikipedia.org/wiki/Effective_field_goal_percentage) and `Points (PTS)`.

(708 observations, 29 variables)
<p>&nbsp;</p>

**<span style="color:red">2018-19_nba_player-statistics</span>**

This dataset contains the salary of individual players during the 2018-19 NBA season, including `unique player identification number (player_id)`, `player name (player_name)` and `year salary in $USD (salary)`.

(576 observations, 3 variables)
<p>&nbsp;</p>

**<span style="color:red">2018-19_nba_team-statistics_1</span>**

This dataset contains miscellaneous team statistics for the 2018-19 season, including `True Shooting Percentage (TS%)`, `Effective Field Goal Percentage (eFG%)` and `Turnover Percentage (TOV%)`.

(30 observations, 22 variables)
<p>&nbsp;</p>

**<span style="color:red">2018-19_nba_team-statistics_2</span>**

This dataset contains total team statistics for the 2018-19 NBA season, including `Games (G)`, `Free Throws (FT)`, `Steals (STL)` and `Points (PTS)`.

(30 observations, 25 variables)
<p>&nbsp;</p>

**<span style="color:red">2018-19_nba_team-payroll</span>**

This dataset contains the team payroll budget for the 2019-20 NBA season, including `unique team identification number (team_id)`, `team name (team)` and `team payroll budget in 2019-20 in $USD (salary)`.

(30 observations, 3 variables)
<p>&nbsp;</p>

#### Load required packages

Two packages will be used in the data wrangling process.


```r
# Packages required
pkgs <- c("tidyverse","naniar")

# Load the packages using a loop
for (i in 1:length(pkgs)){
  if(!(pkgs[i] %in% rownames(installed.packages()))){
    install.packages(pkgs[i])}
  library(pkgs[i],character.only=TRUE)
  }
```

#### Reading the data

Read the files using the `read_csv()` function from the `readr` package.


```r
# CSV files in the "data" folder
files <- list.files("data", pattern = "csv$") 

# Names of the datasets
file_name <- gsub(pattern = ".csv", "",files)

# Empty list
emty_list <- list()

# Read the data using a loop
for (i in 1:length(files)){
  emty_list[[i]] <- read_csv(paste0("data/",files[i]))
  assign(file_name[i],emty_list[[i]]) 
}

# Keep only five datasets
rm(list=setdiff(ls(),file_name))
```

#### Cleaning the data

Display the internal structure of datasets using the `str()` function from `Base-R` and check the existence of missing values with the `any_na()` function from the `naniar` package.

**1. dataset "2018-19_nba_player-salaries" **

```r
# Structure
str(`2018-19_nba_player-salaries`)
```

```
## spec_tbl_df [576 x 3] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ player_id  : num [1:576] 1 2 3 4 5 6 7 8 9 10 ...
##  $ player_name: chr [1:576] "Alex Abrines" "Quincy Acy" "Steven Adams" "Jaylen Adams" ...
##  $ salary     : num [1:576] 3667645 213948 24157304 236854 2955840 ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   player_id = col_double(),
##   ..   player_name = col_character(),
##   ..   salary = col_double()
##   .. )
##  - attr(*, "problems")=<externalptr>
```

```r
# Missing values (no missing values have been found)
any_na(`2018-19_nba_player-salaries`)
```

```
## [1] FALSE
```

**2. dataset "2018-19_nba_player-statistics"**

```r
# Structure
str(`2018-19_nba_player-statistics`)
```

```
## spec_tbl_df [708 x 29] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ player_name: chr [1:708] "Alex Abrines" "Quincy Acy" "Jaylen Adams" "Steven Adams" ...
##  $ Pos        : chr [1:708] "SG" "PF" "PG" "C" ...
##  $ Age        : num [1:708] 25 28 22 25 21 21 25 33 21 23 ...
##  $ Tm         : chr [1:708] "OKC" "PHO" "ATL" "OKC" ...
##  $ G          : num [1:708] 31 10 34 80 82 19 7 81 10 38 ...
##  $ GS         : num [1:708] 2 0 1 80 28 3 0 81 1 2 ...
##  $ MP         : num [1:708] 588 123 428 2669 1913 ...
##  $ FG         : num [1:708] 56 4 38 481 280 11 3 684 13 67 ...
##  $ FGA        : num [1:708] 157 18 110 809 486 ...
##  $ FG%        : num [1:708] 0.357 0.222 0.345 0.595 0.576 0.306 0.3 0.519 0.333 0.376 ...
##  $ 3P         : num [1:708] 41 2 25 0 3 6 0 10 3 32 ...
##  $ 3PA        : num [1:708] 127 15 74 2 15 23 4 42 12 99 ...
##  $ 3P%        : num [1:708] 0.323 0.133 0.338 0 0.2 0.261 0 0.238 0.25 0.323 ...
##  $ 2P         : num [1:708] 15 2 13 481 277 5 3 674 10 35 ...
##  $ 2PA        : num [1:708] 30 3 36 807 471 ...
##  $ 2P%        : num [1:708] 0.5 0.667 0.361 0.596 0.588 0.385 0.5 0.528 0.37 0.443 ...
##  $ eFG%       : num [1:708] 0.487 0.278 0.459 0.595 0.579 0.389 0.3 0.522 0.372 0.466 ...
##  $ FT         : num [1:708] 12 7 7 146 166 4 1 349 8 45 ...
##  $ FTA        : num [1:708] 13 10 9 292 226 4 2 412 12 60 ...
##  $ FT%        : num [1:708] 0.923 0.7 0.778 0.5 0.735 1 0.5 0.847 0.667 0.75 ...
##  $ ORB        : num [1:708] 5 3 11 391 165 3 1 251 11 3 ...
##  $ DRB        : num [1:708] 43 22 49 369 432 16 3 493 15 20 ...
##  $ TRB        : num [1:708] 48 25 60 760 597 19 4 744 26 23 ...
##  $ AST        : num [1:708] 20 8 65 124 184 5 6 194 13 25 ...
##  $ STL        : num [1:708] 17 1 14 117 71 1 2 43 1 6 ...
##  $ BLK        : num [1:708] 6 4 5 76 65 4 0 107 0 6 ...
##  $ TOV        : num [1:708] 14 4 28 135 121 6 2 144 8 33 ...
##  $ PF         : num [1:708] 53 24 45 204 203 13 4 179 7 47 ...
##  $ PTS        : num [1:708] 165 17 108 1108 729 ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   player_name = col_character(),
##   ..   Pos = col_character(),
##   ..   Age = col_double(),
##   ..   Tm = col_character(),
##   ..   G = col_double(),
##   ..   GS = col_double(),
##   ..   MP = col_double(),
##   ..   FG = col_double(),
##   ..   FGA = col_double(),
##   ..   `FG%` = col_double(),
##   ..   `3P` = col_double(),
##   ..   `3PA` = col_double(),
##   ..   `3P%` = col_double(),
##   ..   `2P` = col_double(),
##   ..   `2PA` = col_double(),
##   ..   `2P%` = col_double(),
##   ..   `eFG%` = col_double(),
##   ..   FT = col_double(),
##   ..   FTA = col_double(),
##   ..   `FT%` = col_double(),
##   ..   ORB = col_double(),
##   ..   DRB = col_double(),
##   ..   TRB = col_double(),
##   ..   AST = col_double(),
##   ..   STL = col_double(),
##   ..   BLK = col_double(),
##   ..   TOV = col_double(),
##   ..   PF = col_double(),
##   ..   PTS = col_double()
##   .. )
##  - attr(*, "problems")=<externalptr>
```

```r
# Missing values (missing values have been found, and further action is required)
any_na(`2018-19_nba_player-statistics`)
```

```
## [1] TRUE
```

**3. dataset "2018-19_nba_team-statistics_1"**

```r
# Structure
str(`2018-19_nba_team-statistics_1`)
```

```
## spec_tbl_df [30 x 25] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ Rk    : num [1:30] 1 2 3 4 5 6 7 8 9 10 ...
##  $ Team  : chr [1:30] "Milwaukee Bucks" "Golden State Warriors" "Toronto Raptors" "Utah Jazz" ...
##  $ Age   : num [1:30] 26.9 28.4 27.3 27.3 29.2 26.2 24.9 25.7 25.7 27 ...
##  $ W     : num [1:30] 60 57 58 50 53 53 54 49 49 48 ...
##  $ L     : num [1:30] 22 25 24 32 29 29 28 33 33 34 ...
##  $ PW    : num [1:30] 61 56 56 54 53 51 51 52 50 50 ...
##  $ PL    : num [1:30] 21 26 26 28 29 31 31 30 32 32 ...
##  $ MOV   : num [1:30] 8.87 6.46 6.09 5.26 4.77 4.2 3.95 4.44 3.4 3.33 ...
##  $ SOS   : num [1:30] -0.82 -0.04 -0.6 0.03 0.19 0.24 0.24 -0.54 0.15 -0.57 ...
##  $ SRS   : num [1:30] 8.04 6.42 5.49 5.28 4.96 4.43 4.19 3.9 3.56 2.76 ...
##  $ ORtg  : num [1:30] 114 116 113 111 116 ...
##  $ DRtg  : num [1:30] 105 110 107 106 111 ...
##  $ NRtg  : num [1:30] 8.6 6.4 6 5.2 4.8 4.2 4.1 4.4 3.3 3.4 ...
##  $ Pace  : num [1:30] 103.3 100.9 100.2 100.3 97.9 ...
##  $ FTr   : num [1:30] 0.255 0.227 0.247 0.295 0.279 0.258 0.232 0.215 0.266 0.242 ...
##  $ 3PAr  : num [1:30] 0.419 0.384 0.379 0.394 0.519 0.339 0.348 0.381 0.347 0.292 ...
##  $ TS%   : num [1:30] 0.583 0.596 0.579 0.572 0.581 0.568 0.558 0.567 0.545 0.561 ...
##  $ eFG%  : num [1:30] 0.55 0.565 0.543 0.538 0.542 0.528 0.527 0.534 0.514 0.53 ...
##  $ TOV%  : num [1:30] 12 12.6 12.4 13.4 12 12.1 11.9 11.5 11.7 12.4 ...
##  $ ORB%  : num [1:30] 20.8 22.5 21.9 22.9 22.8 26.6 26.6 21.6 26 21.9 ...
##  $ FT/FGA: num [1:30] 0.197 0.182 0.198 0.217 0.221 0.21 0.175 0.173 0.19 0.182 ...
##  $ DRB%  : num [1:30] 80.3 77.1 77.1 80.3 74.4 77.9 78 77 78.2 76.2 ...
##  $ ...23 : logi [1:30] NA NA NA NA NA NA ...
##  $ ...24 : logi [1:30] NA NA NA NA NA NA ...
##  $ ...25 : logi [1:30] NA NA NA NA NA NA ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   Rk = col_double(),
##   ..   Team = col_character(),
##   ..   Age = col_double(),
##   ..   W = col_double(),
##   ..   L = col_double(),
##   ..   PW = col_double(),
##   ..   PL = col_double(),
##   ..   MOV = col_double(),
##   ..   SOS = col_double(),
##   ..   SRS = col_double(),
##   ..   ORtg = col_double(),
##   ..   DRtg = col_double(),
##   ..   NRtg = col_double(),
##   ..   Pace = col_double(),
##   ..   FTr = col_double(),
##   ..   `3PAr` = col_double(),
##   ..   `TS%` = col_double(),
##   ..   `eFG%` = col_double(),
##   ..   `TOV%` = col_double(),
##   ..   `ORB%` = col_double(),
##   ..   `FT/FGA` = col_double(),
##   ..   `DRB%` = col_double(),
##   ..   ...23 = col_logical(),
##   ..   ...24 = col_logical(),
##   ..   ...25 = col_logical()
##   .. )
##  - attr(*, "problems")=<externalptr>
```

```r
# Missing values (missing values have been found, and further action is required)
any_na(`2018-19_nba_team-statistics_1`)
```

```
## [1] TRUE
```

**4. dataset "2018-19_nba_team-statistics_2"**

```r
# Structure
str(`2018-19_nba_team-statistics_2`)
```

```
## spec_tbl_df [30 x 25] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ Rk  : num [1:30] 1 2 3 4 5 6 7 8 9 10 ...
##  $ Team: chr [1:30] "Milwaukee Bucks" "Golden State Warriors" "New Orleans Pelicans" "Philadelphia 76ers" ...
##  $ G   : num [1:30] 82 82 82 82 82 82 82 82 82 82 ...
##  $ MP  : num [1:30] 19780 19805 19755 19805 19830 ...
##  $ FG  : num [1:30] 3555 3612 3581 3407 3384 ...
##  $ FGA : num [1:30] 7471 7361 7563 7233 7178 ...
##  $ FG% : num [1:30] 0.476 0.491 0.473 0.471 0.471 0.467 0.454 0.474 0.464 0.468 ...
##  $ 3P  : num [1:30] 1105 1087 842 889 821 ...
##  $ 3PA : num [1:30] 3134 2824 2449 2474 2118 ...
##  $ 3P% : num [1:30] 0.353 0.385 0.344 0.359 0.388 0.359 0.348 0.366 0.378 0.341 ...
##  $ 2P  : num [1:30] 2450 2525 2739 2518 2563 ...
##  $ 2PA : num [1:30] 4337 4537 5114 4759 5060 ...
##  $ 2P% : num [1:30] 0.565 0.557 0.536 0.529 0.507 0.523 0.51 0.539 0.504 0.543 ...
##  $ FT  : num [1:30] 1471 1339 1462 1742 1853 ...
##  $ FTA : num [1:30] 1904 1672 1921 2258 2340 ...
##  $ FT% : num [1:30] 0.773 0.801 0.761 0.771 0.792 0.814 0.713 0.804 0.726 0.768 ...
##  $ ORB : num [1:30] 762 797 909 892 796 ...
##  $ DRB : num [1:30] 3316 2990 2969 3025 2936 ...
##  $ TRB : num [1:30] 4078 3787 3878 3917 3732 ...
##  $ AST : num [1:30] 2136 2413 2216 2207 1970 ...
##  $ STL : num [1:30] 615 625 610 606 561 546 766 680 679 683 ...
##  $ BLK : num [1:30] 486 525 441 432 385 413 425 437 363 379 ...
##  $ TOV : num [1:30] 1137 1169 1215 1223 1193 ...
##  $ PF  : num [1:30] 1608 1757 1732 1745 1913 ...
##  $ PTS : num [1:30] 9686 9650 9466 9445 9442 ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   Rk = col_double(),
##   ..   Team = col_character(),
##   ..   G = col_double(),
##   ..   MP = col_double(),
##   ..   FG = col_double(),
##   ..   FGA = col_double(),
##   ..   `FG%` = col_double(),
##   ..   `3P` = col_double(),
##   ..   `3PA` = col_double(),
##   ..   `3P%` = col_double(),
##   ..   `2P` = col_double(),
##   ..   `2PA` = col_double(),
##   ..   `2P%` = col_double(),
##   ..   FT = col_double(),
##   ..   FTA = col_double(),
##   ..   `FT%` = col_double(),
##   ..   ORB = col_double(),
##   ..   DRB = col_double(),
##   ..   TRB = col_double(),
##   ..   AST = col_double(),
##   ..   STL = col_double(),
##   ..   BLK = col_double(),
##   ..   TOV = col_double(),
##   ..   PF = col_double(),
##   ..   PTS = col_double()
##   .. )
##  - attr(*, "problems")=<externalptr>
```

```r
# Missing values (no missing values have been found)
any_na(`2018-19_nba_team-statistics_2`)
```

```
## [1] FALSE
```

**5. dataset "2019-20_nba_team-payroll"**

```r
# Structure
str(`2019-20_nba_team-payroll`)
```

```
## spec_tbl_df [30 x 3] (S3: spec_tbl_df/tbl_df/tbl/data.frame)
##  $ team_id: num [1:30] 1 2 3 4 5 6 7 8 9 10 ...
##  $ team   : chr [1:30] "Miami" "Golden State" "Oklahoma City" "Toronto" ...
##  $ salary : chr [1:30] "$153,171,497" "$146,291,276" "$144,916,427" "$137,793,831" ...
##  - attr(*, "spec")=
##   .. cols(
##   ..   team_id = col_double(),
##   ..   team = col_character(),
##   ..   salary = col_character()
##   .. )
##  - attr(*, "problems")=<externalptr>
```

```r
# Convert the class of the variable "salary" from character to number
`2019-20_nba_team-payroll`$salary <- parse_number(`2019-20_nba_team-payroll`[['salary']])

# Missing values (no missing values have been found)
any_na(`2019-20_nba_team-payroll`)
```

```
## [1] FALSE
```

## 3. Exploratory analysis

### a) Checking for errors and missing values within the datasets

Two datasets were found to have missing values.

- **`2018-19_nba_player-statistics`**

```r
gg_miss_var(`2018-19_nba_player-statistics`,show_pct = TRUE)
```

![](figs/2018-19_nba_player-statistics-1.png)<!-- -->

Missing values in the dataset are the results of dividing zero y zero. For example, `FG% on 3-Pt FGAs (3P%)` is the quotient of `3-Point Field Goals (3P)` and `3-Point Field Goal Attempts (3PA)`. When a player has zero attempts, the ratio will become undefined. Hence, missing values will be replaced with zero.


```r
# Replace missing values
for (i in 1:ncol(`2018-19_nba_player-statistics`)){
  `2018-19_nba_player-statistics`[,i][is.na(`2018-19_nba_player-statistics`[,i])] <- 0
}
```

- **`2018-19_nba_team-statistics_1`**

```r
gg_miss_var(`2018-19_nba_team-statistics_1`,show_pct = TRUE)
```

![](figs/2018-19_nba_team-statistics_1-1.png)<!-- -->

Three empty columns will be removed.

```r
empty_columns <- sapply(`2018-19_nba_team-statistics_1`, function(x) all(is.na(x) | x == ""))
`2018-19_nba_team-statistics_1` <- `2018-19_nba_team-statistics_1`[, !empty_columns]
```

### b) Checking for the distribution of variables

1. Age

Muhr (2020) analyses NBA player development and has found that players in the 24–28 range have slightly higher expected performance than players in other age groups. 

```r
# Distribution of players' age
ggplot(`2018-19_nba_player-statistics`,aes(x=Age))+
  geom_histogram(mapping = aes(y = ..density..), colour = "black", fill = "grey",bins=7)+
  geom_density(alpha = 0.3, fill = "dodgerblue")+
  ggtitle("Distribution of players' age")+
  xlab("Players' age")+
  theme_classic()+
  theme(axis.ticks.x = element_blank(),
        axis.ticks.y = element_blank())
```

![](figs/age-1.png)<!-- -->
Most NBA players are in their twenties. It is estimated that players' age directly impacts how many points they can get per game.

2. Salary


```r
# Distribution of NBA player salaries
ggplot(`2018-19_nba_player-salaries`,aes(x=salary/1000))+
  geom_histogram(mapping = aes(y = ..density..), colour = "black", fill = "grey",bins=8)+
  geom_density(alpha = 0.3, fill = "dodgerblue")+
  ggtitle("Distribution of NBA player salaries")+
  xlab("Salaries (in thousands)")+
  scale_x_continuous(labels = scales::comma,
                     breaks = seq(0,50000,by=5000))+
  theme_classic()+
  theme(axis.ticks.x = element_blank(),
        axis.ticks.y = element_blank())
```

![](figs/salary-1.png)<!-- -->
Most NBA players' salaries are less than 5,000 thousand dollars. The distribution of NBA player salaries is unimodal and right-skewed, with several large outliers.

### c) Checking for relationships between variables, or differences between groups

Use joining functions from the "dplyr" package to create a new data frame `team_stat`. 

```r
team_stat <- `2018-19_nba_team-statistics_1` %>% left_join(select(`2018-19_nba_team-statistics_2`,-Rk), by = 'Team')
```

Then, create two new variables representing points per game and the percentage of wins, respectively.


```r
team_stat <- team_stat %>% mutate(PTSG = PTS/G,
                                  WinP = W/G)
```

1. Relationships between variables (teams)

The correlation heatmap using teams' statistics suggests that all variables except `Defensive Rating (DRtg)` and `Turnover Percentage (TOV%)` positively influence `the percentage of wins (WinP)`. There is a correlation of over 98% between `Net Rating (NRtg)` and `the percentage of wins (WinP)`, verifying Fayad's statements.

Some variables, for example, `Net Rating (NRtg)` and `Effective Field Goal Percentage (eFG%)`, are highly correlated. To avoid the issue of multicollinearity, we can select only one of them in the data modelling process.

```r
# Install the package "ellipse" if not installed
if(!require(ellipse)){
  install.packages("ellipse")
}

# Load the package "ellipse"
library(ellipse)

# Create a correlation heatmap using teams' statistics
corr <- team_stat[,c("Age","ORtg","DRtg","NRtg","eFG%","TOV%","ORB%","FT/FGA","DRB%","3P%","FT%","AST","STL","BLK","PTSG","WinP")] %>% cor() 

colorfun <- colorRamp(c("#CC0000","white","#3366CC"), space="Lab")

plotcorr(corr, col=rgb(colorfun((corr+1)/2), maxColorValue=255),
         mar = c(0.1, 0.1, 0.1, 0.1))
```

![](figs/heatmap_teams-1.png)<!-- -->

2. Relationships between variables (players)

The correlation heatmap using players' statistics suggests that the variable `Turnovers (TOV)` has the highest impact on the variable `Points per game (PTSG)`. In addition, `Players' age (Age)` and `Effective Field Goal Percentage (eFG%)` are not the determinant factors of points received at each game.

```r
# Install the package "corrplot" if not installed
if(!require(corrplot)){
  install.packages("corrplot")
}

# Load the package "corrplot"
library(corrplot)

# Create new variables representing players' performance per game
`2018-19_nba_player-statistics` <- within(`2018-19_nba_player-statistics`,{
  ORBG <- ORB/G
  DRBG <- DRB/G
  TRBG <- TRB/G
  ASTG <- AST/G
  STLG <- STL/G
  BLKG <- BLK/G
  TOVG <- TOV/G
  PFG <- PF/G
  PTSG <- PTS/G
  })

# Potential variables used for data modelling
varibls <- c("Age","FG%","3P%","2P%","eFG%","FT%","ORBG","DRBG","TRBG","ASTG","STLG","BLKG","TOVG","PFG","PTSG")
  
# Create a correlation heatmap using players' statistics
colo <- colorRampPalette(c("#7F0000", "red", "#FF7F00", "yellow", "#7FFF7F", 
                           "cyan", "#007FFF", "blue", "#00007F"))

`2018-19_nba_player-statistics`[names(`2018-19_nba_player-statistics`) %in% varibls] %>% relocate('PTSG',.after = last_col()) %>% cor() %>% 
  corrplot(order = "hclust", addrect = 2, col = colo(10)) 
```

![](figs/heatmap_players-1.png)<!-- -->

3. Difference between groups

The boxplot shows that centers have higher `Effective Field Goal Percentage (eFG%)` than other players, so our estimation is that more funding should be apportioned to centers.

```r
# Boxplot
`2018-19_nba_player-statistics` %>% ggplot(aes(x=reorder(Pos,`eFG%`, FUN = median),y=`eFG%`))+
  geom_boxplot(aes(col=Pos))+
  geom_jitter(width=0.3,alpha=0.1,aes(col=Pos))+
  xlab("Position") +
  ylab("Effective Field Goal Percentage (eFG%)") +
  ggtitle("Field Goal Percentage by Position")+
  theme_classic()+
  theme(axis.ticks.x = element_blank(),
        axis.ticks.y = element_blank(),
        legend.position = "right")
```

![](figs/position-1.png)<!-- -->

### d) Justification for decisions made about data modelling.

To help teams find the most suitable players, we need to understand the relationship between the dependent variable `the percentage of wins (WinP)` and other explanatory variables. For example, when `Net Rating (NRtg)` increases by one per cent, to what extent will `the percentage of wins (WinP)` increase. After that, we build a second regression model based on players' statistics. However, there are only 30 samples in the `team_stat` dataset, which is quite small. Deziel(2018) argues that a small sample size will increase the margin of error and decrease the statistical power.[9] As a result, we will build a regression model using `Points per game (PTSG)` as the dependent variable. We don't use `salary` as the dependent variable because some players play at different positions in the 2018-19 season, and salaries differ significantly among positions. Still, we don't know how much have these players earned from each position. Wolf (2019) did a survey and found that centers have the highest average salaries, 46 per cent higher than that of power forwards.[10]

<img src="images/Average Player's Salary by Position.png" width="50%" style="display: block; margin: auto;" />

# 4. Data modelling and results

The second correlation heatmap shows that many factors positively impact `Points per game (PTSG)`. However, including too many features will lead to the problem of overfitting. Estevez (2018) suggests fitting a Random Forest model to identify variable importance.[11]

The variable importance plot shows that `Turnovers per game (TOVG)`, `Defensive rebounds per game (DRBG)`,  `Steals per game (STLG)`, `Total rebounds per game (TRBG)` and `Assists per game (ASTG)` are the five most important factors on `Points per game (PTSG)`. `Defensive rebounds per game (DRBG)` and `Total rebounds per game (TRBG)` are highly correlated, so `Total rebounds per game (TRBG)` will be discarded. `Assists per game (ASTG)` and `Turnovers per game (TOVG)` also show a strong positive relationship, so `Assists per game (ASTG)` will be discarded as well. Thus, the model has three explanatory variables, which are `Turnovers per game (TOVG)`, `Defensive rebounds per game (DRBG)` and `Steals per game (STLG)`.


```r
# Install the package "caret" if not installed
if(!require(caret)){
  install.packages("caret")
}

# Load the package "caret"
library(caret)

# A dataset containing variables used for modelling
datst <- `2018-19_nba_player-statistics` %>% select(all_of(varibls))

colnames(datst) <- make.names(colnames(datst))

rPartMod <-  train(PTSG ~ ., data=datst, method="rpart")
rpartImp <-  varImp(rPartMod,10)
plot(rpartImp, 
     top = 6, 
     main="Variable Importance based on players' statistics")
```

![](figs/variable_importance-1.png)<!-- -->

### a) Data modelling (e.g. creating a linear regression).

A multiple linear regression model with three explanatory variables is built.


```r
# Install the package "broom" if not installed
if(!require(broom)){
  install.packages("broom")
}

# Load the package "broom"
library(broom)

# A function to extract coefficients 
source("funcs/Equation.R")

# multiple linear regression model
model <- lm(PTSG ~ TOVG + DRBG + STLG, data = `2018-19_nba_player-statistics`)
```

The equation of the model:

```r
regEq(model,3)
```

```
## [1] "PTSG = 0.307 + 4.377*TOVG + 0.802*DRBG + 2.245*STLG"
```

### b) Assumption checking.

1. Response variable

The response variable `Points per game (PTSG)` is continuous.

2. Explanatory variables

All three variables, `Turnovers per game (TOVG)`, `Defensive rebounds per game (DRBG)` and `Steals per game (STLG)`, are continuous.

3. Independence of observations

We can tell from our study design that we do not have independence of observations as we are just analysing basketball players from one season, meaning there are no repeated measures.

Nonetheless, we can also test that our residuals are not autocorrelated with the DW test. Our results are 1.6, which is close to the recommended 2 to ensure independence. 

Thus we have not failed this assumption. 

```r
# Install the package "car" if not installed
if(!require(car)){
  install.packages("car")
}

# Load the package "car"
library(car)

# DW test
durbinWatsonTest(model)
```

```
##  lag Autocorrelation D-W Statistic p-value
##    1       0.1997355      1.600416       0
##  Alternative hypothesis: rho != 0
```

4. Linear relationship

Partial regression plots suggest that there is a linear relationship between the response variable `Points per game (PTSG)` and each explanatory variable.

```r
# partial regression plots
avPlots(model)
```

![](figs/linear relationship-1.png)<!-- -->

5-1. Outliers

There are six outliers with a z-score higher than 3, so the assumption has not been met.

```r
std_res <- rstandard(model)
points <- 1:length(std_res)
labls <- if_else(std_res >= 3, paste(points), "")

ggplot(data = NULL, aes(x = points, y = std_res, label = labls)) +
  geom_point(color = "dodgerblue") +
  geom_text(nudge_x = 0.1, cex =3)+
  ylim(c(-4,4)) +
  geom_hline(yintercept = c(-3, 3), colour = "red", linetype = "dashed")+
  theme_classic()
```

![](figs/outlier-1.png)<!-- -->

5-2. Leverage points

No high leverage points have been found, so the assumption has been met.

```r
hats <- hatvalues(model)

hat_labels <- if_else(hats >= 0.10, paste(points), "")

ggplot(data = NULL, aes(x = points, y = hats)) +
  geom_point(color = "dodgerblue") +
  geom_text(aes(label = hat_labels), nudge_y = 0.005)+
  theme_classic()
```

![](figs/leverage points-1.png)<!-- -->

5-3. Influential points

Five influential points have been found, so the assumption has not been met.

```r
cook <- cooks.distance(model)

cook_labels <- if_else(cook >= 0.05, paste(points), "")

ggplot(data = NULL, aes(x = points, y = cook)) +
  geom_point(color = "dodgerblue") +
  geom_text(aes(label = cook_labels), nudge_x = 1)+
  theme_classic()
```

![](figs/influence-1.png)<!-- -->

6. Homoscedasticity

The assumption of homoscedasticity has not been met. The residual plot shows a triangle pattern, and the Breusch–Pagan test also suggests the issue of heteroscedasticity. Hence, a WLS model is needed.

```r
res <- residuals(model)
fitted <- predict(model)

ggplot(data = NULL, aes(x = fitted, y = res)) +
  geom_point(colour = "dodgerblue") + 
  geom_hline(yintercept = 0, colour = "red", linetype = "dashed")+
  theme_classic()
```

![](figs/homoscedasticity-1.png)<!-- -->

```r
# Install the package "lmtest" if not installed
if(!require(lmtest)){
  install.packages("lmtest")
}

# Load the package "lmtest"
library(lmtest)

bptest(model)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  model
## BP = 92.112, df = 3, p-value < 2.2e-16
```

7. Residuals

The assumption of residuals following the normal distribution has not been fully met, which is one of the results of heteroscedasticity.

```r
ggplot(data = NULL, aes(x = res)) +
  geom_histogram(colour = "black", fill = "dodgerblue", bins = 8)+
  theme_classic()
```

![](figs/residuals-1.png)<!-- -->

```r
ggplot(data = NULL, aes(sample = res)) +
  stat_qq() + 
  stat_qq_line()+
  theme_classic()
```

![](figs/residuals-2.png)<!-- -->

8. Multicollinearity

The plot suggests that there is a positive relationship between variables, and further investigation is needed. VIF is lower than 5, so the issue of multicollinearity does not exist.

```r
# Plot
pairs(formula = ~ PTSG + TOVG + DRBG + STLG, data = datst)
```

![](figs/multicollinearity-1.png)<!-- -->

```r
# Variance Inflation Factor (VIF)
vif(model)
```

```
##     TOVG     DRBG     STLG 
## 2.299734 1.775661 1.694033
```

### c) New model:


```r
outliers <- which(std_res >= 3)

high_influential_points <- which(cook >= 0.05)

removed_points <- unique(c(outliers,high_influential_points))

`2018-19_nba_player-statistics` <- `2018-19_nba_player-statistics`[-removed_points,]

`2018-19_nba_player-statistics` <- `2018-19_nba_player-statistics` %>% mutate(PTSGn = PTSG/(TOVG^0.5), 
TOVGn=TOVG/(TOVG^0.5), 
DRBGn=DRBG/(TOVG^0.5),
STLGn=STLG/(TOVG^0.5),
interceptn=1/(TOVG^0.5))

# Install the package "IDPmisc" if not installed
if(!require(IDPmisc)){
  install.packages("IDPmisc")
}

# Load the package "IDPmisc"
library(IDPmisc)

`2018-19_nba_player-statistics` <- NaRV.omit(`2018-19_nba_player-statistics`)

model_NEW <- lm(PTSGn ~ + 0 + TOVGn + DRBGn + STLGn, data = `2018-19_nba_player-statistics`)
```

Residual plot and the Breusch–Pagan test:

Both the residual plot and the bp test show that the issue of heteroscedasticity no longer exists.

```r
plot(model_NEW,1)
```

![](figs/wls1-1.png)<!-- -->

```r
bptest(model_NEW)
```

```
## 
## 	studentized Breusch-Pagan test
## 
## data:  model_NEW
## BP = 3.1811, df = 2, p-value = 0.2038
```

QQ plot:

QQ plot suggests that the residuals of new model follows the normal distribution.

```r
plot(model_NEW,2)
```

![](figs/wls2-1.png)<!-- -->

### d) Model output and interpretation of your model.

1. Interpretation:

Intercept: WLS model does not have an intercept

Slope estimate of `Turnovers per game (TOVGn)` = 4.28, meaning that for every 1 unit that `Turnovers per game (TOVGn)` is incresaed, a player's points per game increases by 4.28.

Slope estimate of `Defensive rebounds per game (DRBGn)` = 0.961, meaning that for every 1 unit that `Defensive rebounds per game (DRBGn)` is incresaed, a player's points per game increases by 0.961.

Slope estimate of `Steals per game (STLGn)` = 2.05, meaning that for every 1 unit that `Steals per game (STLGn)` is incresaed, a player's points per game increases by 2.05.

2. The summary of the model:

Based on the p-values and an alpha = 5%, we can conclude that slope estimates are statistically significant.

```r
tidy(model_NEW)
```

```
## # A tibble: 3 x 5
##   term  estimate std.error statistic  p.value
##   <chr>    <dbl>     <dbl>     <dbl>    <dbl>
## 1 TOVGn    4.28     0.246      17.4  2.75e-56
## 2 DRBGn    0.961    0.0754     12.8  1.85e-33
## 3 STLGn    2.05     0.306       6.69 4.83e-11
```

3. $R^{2}$:

R-squared = 0.911, meaning that 91.3% of the variance in `Points per game (PTSGn)` is explained by the variances in explanatory variables.

4. F distribution:

The p-value of F distribution is 2.2e-16, which is less than 5 per cent, so we reject the null hypothesis and conclude that at least one slope is not equal 
to 0 (i.e., the regression is significant).

```r
glance(model_NEW)
```

```
## # A tibble: 1 x 12
##   r.squared adj.r.squared sigma statistic p.value    df logLik   AIC   BIC
##       <dbl>         <dbl> <dbl>     <dbl>   <dbl> <dbl>  <dbl> <dbl> <dbl>
## 1     0.911         0.911  2.65     2250.       0     3 -1585. 3179. 3197.
## # ... with 3 more variables: deviance <dbl>, df.residual <int>, nobs <int>
```

# 5. Player recommendations

NBA teams are allowed to carry 15 players on the active roster for regular seasons. We recommend that about half of the payroll budget be spent on the five starting players (i.e., salaries paid to five starting players are two times higher than those of the other ten players). Thus, the payroll budget for these five players is $56299100.5.

Since Center and Point Guard are the two most important positions, more funds have to be spent on these two players. Based on each position's average salary (Wolf,2019)[10], the rough allocation of funds is:

`Center`: $13511784.12 (24% of 56299100.5 dollars)

`Point guard`: $12948793.115 (23% of 56299100.5 dollars)

`Small forward`: $11259820.1 (20% of 56299100.5 dollars)

`Shooting guard`: $9570847.085 (17% of 56299100.5 dollars)

`Power forward`: $9007856.08 (16% of 56299100.5 dollars)

The next step is selecting players whose actual points per game are higher than predicted.

```r
`2018-19_nba_player-statistics`$fit <- as.vector(fitted.values(model_NEW))

Recommendation <- `2018-19_nba_player-statistics`%>% filter(fit >=PTSGn) %>% left_join(subset(`2018-19_nba_player-salaries`,select = -player_id), by = "player_name")

# Center
Recommendation %>% filter(between(salary,0.9*13511784.12,1.1*13511784.12) & Pos == "C") %>% arrange(desc(TOVGn))
```

```
## # A tibble: 6 x 45
##   player_name  Pos     Age Tm        G    GS    MP    FG   FGA `FG%`  `3P` `3PA`
##   <chr>        <chr> <dbl> <chr> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1 Mason Pluml~ C        28 DEN      82    17  1731   262   442 0.593     2    10
## 2 Cody Zeller  C        26 CHO      49    47  1243   190   345 0.551     6    22
## 3 Marcin Gort~ C        34 LAC      47    43   751    99   186 0.532     0     0
## 4 Tyson Chand~ C        36 PHO       7     0    89     8    12 0.667     0     0
## 5 Tyson Chand~ C        36 TOT      55     6   875    61    99 0.616     0     1
## 6 Tyson Chand~ C        36 LAL      48     6   786    53    87 0.609     0     1
## # ... with 33 more variables: 3P% <dbl>, 2P <dbl>, 2PA <dbl>, 2P% <dbl>,
## #   eFG% <dbl>, FT <dbl>, FTA <dbl>, FT% <dbl>, ORB <dbl>, DRB <dbl>,
## #   TRB <dbl>, AST <dbl>, STL <dbl>, BLK <dbl>, TOV <dbl>, PF <dbl>, PTS <dbl>,
## #   PTSG <dbl>, PFG <dbl>, TOVG <dbl>, BLKG <dbl>, STLG <dbl>, ASTG <dbl>,
## #   TRBG <dbl>, DRBG <dbl>, ORBG <dbl>, PTSGn <dbl>, TOVGn <dbl>, DRBGn <dbl>,
## #   STLGn <dbl>, interceptn <dbl>, fit <dbl>, salary <dbl>
```

```r
# Point guard
Recommendation %>% filter(between(salary,0.9*12948793.115,1.1*12948793.115) & Pos == "PG") %>% arrange(desc(TOVGn))
```

```
## # A tibble: 3 x 45
##   player_name Pos     Age Tm        G    GS    MP    FG   FGA `FG%`  `3P` `3PA`
##   <chr>       <chr> <dbl> <chr> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1 Jeremy Lin  PG       30 ATL      51     1  1003   180   386 0.466    44   132
## 2 Jeremy Lin  PG       30 TOT      74     4  1436   238   541 0.44     55   187
## 3 Jeremy Lin  PG       30 TOR      23     3   433    58   155 0.374    11    55
## # ... with 33 more variables: 3P% <dbl>, 2P <dbl>, 2PA <dbl>, 2P% <dbl>,
## #   eFG% <dbl>, FT <dbl>, FTA <dbl>, FT% <dbl>, ORB <dbl>, DRB <dbl>,
## #   TRB <dbl>, AST <dbl>, STL <dbl>, BLK <dbl>, TOV <dbl>, PF <dbl>, PTS <dbl>,
## #   PTSG <dbl>, PFG <dbl>, TOVG <dbl>, BLKG <dbl>, STLG <dbl>, ASTG <dbl>,
## #   TRBG <dbl>, DRBG <dbl>, ORBG <dbl>, PTSGn <dbl>, TOVGn <dbl>, DRBGn <dbl>,
## #   STLGn <dbl>, interceptn <dbl>, fit <dbl>, salary <dbl>
```

```r
# Small forward
Recommendation %>% filter(between(salary,0.9*11259820.1,1.1*11259820.1) & Pos == "SF") %>% arrange(desc(TOVGn))
```

```
## # A tibble: 2 x 45
##   player_name  Pos     Age Tm        G    GS    MP    FG   FGA `FG%`  `3P` `3PA`
##   <chr>        <chr> <dbl> <chr> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1 Robert Covi~ SF       28 PHI      13    13   440    50   117 0.427    30    77
## 2 Robert Covi~ SF       28 TOT      35    35  1203   156   362 0.431    85   225
## # ... with 33 more variables: 3P% <dbl>, 2P <dbl>, 2PA <dbl>, 2P% <dbl>,
## #   eFG% <dbl>, FT <dbl>, FTA <dbl>, FT% <dbl>, ORB <dbl>, DRB <dbl>,
## #   TRB <dbl>, AST <dbl>, STL <dbl>, BLK <dbl>, TOV <dbl>, PF <dbl>, PTS <dbl>,
## #   PTSG <dbl>, PFG <dbl>, TOVG <dbl>, BLKG <dbl>, STLG <dbl>, ASTG <dbl>,
## #   TRBG <dbl>, DRBG <dbl>, ORBG <dbl>, PTSGn <dbl>, TOVGn <dbl>, DRBGn <dbl>,
## #   STLGn <dbl>, interceptn <dbl>, fit <dbl>, salary <dbl>
```

```r
# Shooting guard 
Recommendation %>% filter(between(salary,0.85*9570847.085,1.15*9570847.085) & Pos == "SG") %>% arrange(desc(TOVGn))
```

```
## # A tibble: 1 x 45
##   player_name  Pos     Age Tm        G    GS    MP    FG   FGA `FG%`  `3P` `3PA`
##   <chr>        <chr> <dbl> <chr> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1 Markelle Fu~ SG       20 PHI      19    15   427    65   155 0.419     4    14
## # ... with 33 more variables: 3P% <dbl>, 2P <dbl>, 2PA <dbl>, 2P% <dbl>,
## #   eFG% <dbl>, FT <dbl>, FTA <dbl>, FT% <dbl>, ORB <dbl>, DRB <dbl>,
## #   TRB <dbl>, AST <dbl>, STL <dbl>, BLK <dbl>, TOV <dbl>, PF <dbl>, PTS <dbl>,
## #   PTSG <dbl>, PFG <dbl>, TOVG <dbl>, BLKG <dbl>, STLG <dbl>, ASTG <dbl>,
## #   TRBG <dbl>, DRBG <dbl>, ORBG <dbl>, PTSGn <dbl>, TOVGn <dbl>, DRBGn <dbl>,
## #   STLGn <dbl>, interceptn <dbl>, fit <dbl>, salary <dbl>
```

```r
# Power forward
Recommendation %>% filter(between(salary,0.9*9007856.08,1.1*9007856.08) & Pos == "PF") %>% arrange(desc(TOVGn))
```

```
## # A tibble: 1 x 45
##   player_name  Pos     Age Tm        G    GS    MP    FG   FGA `FG%`  `3P` `3PA`
##   <chr>        <chr> <dbl> <chr> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl> <dbl>
## 1 Jared Dudley PF       33 BRK      59    25  1220   101   239 0.423    53   151
## # ... with 33 more variables: 3P% <dbl>, 2P <dbl>, 2PA <dbl>, 2P% <dbl>,
## #   eFG% <dbl>, FT <dbl>, FTA <dbl>, FT% <dbl>, ORB <dbl>, DRB <dbl>,
## #   TRB <dbl>, AST <dbl>, STL <dbl>, BLK <dbl>, TOV <dbl>, PF <dbl>, PTS <dbl>,
## #   PTSG <dbl>, PFG <dbl>, TOVG <dbl>, BLKG <dbl>, STLG <dbl>, ASTG <dbl>,
## #   TRBG <dbl>, DRBG <dbl>, ORBG <dbl>, PTSGn <dbl>, TOVGn <dbl>, DRBGn <dbl>,
## #   STLGn <dbl>, interceptn <dbl>, fit <dbl>, salary <dbl>
```
If two or more players are suitable, a player having the highest `Turnovers Per game (TOVGn)` will be selected for the new season.

Center: Mason Plumlee

Point guard: Jeremy Lin

Small forward: Robert Covington

Shooting guard: Markelle Fultz

Power forward: Jared Dudley

# 6. Summary

In conclusion, A player's points per game depend on three factors, turnovers per game, defensive rebounds per game, and steals per game. The correlation heatmap using teams' statistics shows that the more point a team get in a game, the higher probability of winning the game.

The study does not build a model analysing the factors that make the team successful as the sample size is too small. More samples are needed in future studies. In addition, players' salaries usually consist of several parts and differ significantly among positions. To build a model analysing which factors contribute more to salaries, we need to collect more data on the components of players' salaries. Furthermore, home-court advantage is a significant factor influencing wins. In the future, it can be used as a dummy variable in the data modelling.

The study uses multiple linear regression to build a model. Compared with other machine learning algorithms, it is relatively simple and sensitive to outliers. Many factors drive a team or a player's success, but multiple linear regression cannot capture all these important features. In the future, the study will apply more advanced algorithms such as the Random Forest and XGBoost.


# 7. Reference list

1. The 10 most popular sports in the world [Internet]. [place unknown: publisher unknown]; 2022 Mar 10 [cited 2022 Apr 30]. Available from:https://www.econotimes.com/The-10-most-popular-sports-in-the-world-1628908

2. Utathya Nag. How is 3x3 basketball different from traditional basketball? [Internet]. 
Lausanne (Switzerland): Olympics; 2015 [updated 2021; cited 2021 May 01]. Available from: https://olympics.com/en/featured-news/what-how-play-3x3-basketball-rules-scoring-tokyo-olympics-court-size

3. Major professional sports leagues in the United States and Canada [Internet]. [place unknown: publisher unknown]; [date unknown] [cited 2022 May 01]. Available from:https://en.wikipedia.org/wiki/Major_professional_sports_leagues_in_the_United_States_and_Canada

4. Basketball positions [Internet]. [place unknown: publisher unknown]; [date unknown] [cited 2022 May 01]. Available from:https://en.wikipedia.org/wiki/Basketball_positions

5. Shane. The Most Important Positions to Consider in Basketball Betting [Internet]. 
[place unknown: publisher unknown]: GamblingSites; 2018 [cited 2021 May 02]. Available from: https://www.gamblingsites.net/news/the-most-important-positions-to-consider-in-basketball-betting-12343/

6. Jeff Haefner. 9 Stats That Every Serious Basketball Coach Should Track [Internet]. 
[place unknown]: Breakthrough Basketball; [date unknown] [cited 2021 May 02]. Available from: https://www.breakthroughbasketball.com/stats/9_stats_basketball_coach_should_track.html

7. Sampaio J, Janeira M, Ibáñez S, Lorenzo A. Discriminant analysis of game-related statistics between basketball guards, forwards and centres in three professional leagues. European journal of sport science. 2006 Sep 1;6(3):173-8.

8. Alexander Fayad. Building My First Machine Learning Model | NBA Prediction Algorithm [Internet]. [place unknown]: Towards Data Science; 2020 [cited 2021 May 03]. Available from: https://towardsdatascience.com/building-my-first-machine-learning-model-nba-prediction-algorithm-dee5c5bc4cc1

9. Chris Deziel. The Effects of a Small Sample Size Limitation [Internet]. [place unknown]: Sciencing; [date unknown] [updated 2021; cited 2021 May 03]. Available from: https://towardsdatascience.com/building-my-first-machine-learning-model-nba-prediction-algorithm-dee5c5bc4cc1

10. Connor Wolf. An Analysis of NBA Teams’ Spending by Position for the Upcoming Season [Internet]. AL (U.S.): Samford University; 2019 [cited 2021 May 07]. Available from: https://www.samford.edu/sports-analytics/fans/2019/An-Analysis-of-NBA-Teams-Spending-by-Position-for-the-Upcoming-Season

11. Christopher Estevez. Feature Selection Blog4 [Internet]. [place unknown: publisher unknown]; 2018 [cited 2021 May 07]. Available from: https://rstudio-pubs-static.s3.amazonaws.com/411593_24037b8f555c440da5ecb0242444fb2b.html
