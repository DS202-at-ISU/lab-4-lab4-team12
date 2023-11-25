
<!-- README.md is generated from README.Rmd. Please edit the README.Rmd file -->

# Lab report \#4 - instructions

Follow the instructions posted at
<https://ds202-at-isu.github.io/labs.html> for the lab assignment. The
work is meant to be finished during the lab time, but you have time
until Monday (after Thanksgiving) to polish things.

All submissions to the github repo will be automatically uploaded for
grading once the due date is passed. Submit a link to your repository on
Canvas (only one submission per team) to signal to the instructors that
you are done with your submission.

# Lab 4: Scraping (into) the Hall of Fame

``` r
# Scraping pulled from the lab manual

library(rvest)
```

    ## Warning: package 'rvest' was built under R version 4.3.2

    ## 
    ## Attaching package: 'rvest'

    ## The following object is masked from 'package:readr':
    ## 
    ##     guess_encoding

``` r
url <- "https://www.baseball-reference.com/awards/hof_2023.shtml"
html <- read_html(url)
tables <- html_table(html)

head(tables[[1]], 3)
```

    ## # A tibble: 3 × 39
    ##   ``    ``          ``    ``    ``    ``    ``    ``    ``    ``    ``    ``   
    ##   <chr> <chr>       <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr> <chr>
    ## 1 Rk    Name        YoB   Votes %vote HOFm  HOFs  Yrs   WAR   WAR7  JAWS  Jpos 
    ## 2 1     Scott Rolen 6th   297   76.3% 99    40    17    70.1  43.6  56.9  55.8 
    ## 3 2     Todd Helton 5th   281   72.2% 175   59    17    61.8  46.6  54.2  53.4 
    ## # ℹ 27 more variables: `Batting Stats` <chr>, `Batting Stats` <chr>,
    ## #   `Batting Stats` <chr>, `Batting Stats` <chr>, `Batting Stats` <chr>,
    ## #   `Batting Stats` <chr>, `Batting Stats` <chr>, `Batting Stats` <chr>,
    ## #   `Batting Stats` <chr>, `Batting Stats` <chr>, `Batting Stats` <chr>,
    ## #   `Batting Stats` <chr>, `Batting Stats` <chr>, `Pitching Stats` <chr>,
    ## #   `Pitching Stats` <chr>, `Pitching Stats` <chr>, `Pitching Stats` <chr>,
    ## #   `Pitching Stats` <chr>, `Pitching Stats` <chr>, `Pitching Stats` <chr>, …

``` r
# Creating column names pulled from the lab manual
write.csv(tables[[1]], "temp.csv", row.names=FALSE)
add_hof <- readr::read_csv("temp.csv", skip = 1, show_col_types =FALSE)
```

    ## New names:
    ## • `G` -> `G...13`
    ## • `H` -> `H...16`
    ## • `HR` -> `HR...17`
    ## • `BB` -> `BB...20`
    ## • `G` -> `G...31`
    ## • `H` -> `H...35`
    ## • `HR` -> `HR...36`
    ## • `BB` -> `BB...37`

``` r
# Making table into an identical dataframe to work with
add_hof <- data.frame(rbind(add_hof))
colnames(add_hof)[5] <- "%.vote"

head(add_hof, 3)
```

    ##   Rk         Name YoB Votes %.vote HOFm HOFs Yrs  WAR WAR7 JAWS Jpos G...13
    ## 1  1  Scott Rolen 6th   297  76.3%   99   40  17 70.1 43.6 56.9 55.8   2038
    ## 2  2  Todd Helton 5th   281  72.2%  175   59  17 61.8 46.6 54.2 53.4   2247
    ## 3  3 Billy Wagner 8th   265  68.1%  107   24  16 27.7 19.8 23.7 32.5    810
    ##     AB    R H...16 HR...17  RBI  SB BB...20    BA   OBP   SLG   OPS OPS.  W  L
    ## 1 7398 1211   2077     316 1287 118     899 0.281 0.364 0.490 0.855  122 NA NA
    ## 2 7962 1401   2519     369 1406  37    1335 0.316 0.414 0.539 0.953  133 NA NA
    ## 3   20    1      2       0    1   0       1 0.100 0.143 0.100 0.243  -35 47 40
    ##    ERA ERA.  WHIP G...31 GS  SV  IP H...35 HR...36 BB...37   SO Pos.Summary
    ## 1   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA        *5/H
    ## 2   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA     *3H/7D9
    ## 3 2.31  187 0.998    853  0 422 903    601      82     300 1196          *1

``` r
# Cleaning %.vote column into numerical variable 
add_hof <- add_hof %>% 
  mutate(`%.vote` = parse_number(`%.vote`))

# Cleaning X from player names
add_hof <- add_hof %>% 
  mutate(Name = gsub("X-", "", Name))

# Adding a yearID column since it is not already there from scraping (prepping for eventual join)
add_hof <- add_hof %>% 
  mutate(yearID = 2023)
```

``` r
# Preparing add_hof for joining with People from Lahman set
add_hof <- add_hof %>% 
  separate(col=Name, into=c('nameFirst', 'nameLast'), sep=' ') 

head(add_hof, 3)
```

    ##   Rk nameFirst nameLast YoB Votes %.vote HOFm HOFs Yrs  WAR WAR7 JAWS Jpos
    ## 1  1     Scott    Rolen 6th   297   76.3   99   40  17 70.1 43.6 56.9 55.8
    ## 2  2      Todd   Helton 5th   281   72.2  175   59  17 61.8 46.6 54.2 53.4
    ## 3  3     Billy   Wagner 8th   265   68.1  107   24  16 27.7 19.8 23.7 32.5
    ##   G...13   AB    R H...16 HR...17  RBI  SB BB...20    BA   OBP   SLG   OPS OPS.
    ## 1   2038 7398 1211   2077     316 1287 118     899 0.281 0.364 0.490 0.855  122
    ## 2   2247 7962 1401   2519     369 1406  37    1335 0.316 0.414 0.539 0.953  133
    ## 3    810   20    1      2       0    1   0       1 0.100 0.143 0.100 0.243  -35
    ##    W  L  ERA ERA.  WHIP G...31 GS  SV  IP H...35 HR...36 BB...37   SO
    ## 1 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 2 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 3 47 40 2.31  187 0.998    853  0 422 903    601      82     300 1196
    ##   Pos.Summary yearID
    ## 1        *5/H   2023
    ## 2     *3H/7D9   2023
    ## 3          *1   2023

``` r
# Adding PlayerID to scraped data
modPeop <- People %>% 
  select(nameFirst, nameLast, playerID)

# This join automatically eliminates any players without playerIDs in the Lahman database (removing 4 players)
add_hof <- inner_join(add_hof, modPeop, by = c("nameFirst", "nameLast"))

head(add_hof)
```

    ##   Rk nameFirst  nameLast  YoB Votes %.vote HOFm HOFs Yrs  WAR WAR7 JAWS Jpos
    ## 1  1     Scott     Rolen  6th   297   76.3   99   40  17 70.1 43.6 56.9 55.8
    ## 2  2      Todd    Helton  5th   281   72.2  175   59  17 61.8 46.6 54.2 53.4
    ## 3  3     Billy    Wagner  8th   265   68.1  107   24  16 27.7 19.8 23.7 32.5
    ## 4  4    Andruw     Jones  6th   226   58.1  109   34  17 62.7 46.4 54.6 58.1
    ## 5  5      Gary Sheffield  9th   214   55.0  158   61  22 60.5 38.0 49.3 56.7
    ## 6  7      Jeff      Kent 10th   181   46.5  123   51  17 55.5 35.8 45.6 57.0
    ##   G...13   AB    R H...16 HR...17  RBI  SB BB...20    BA   OBP   SLG   OPS OPS.
    ## 1   2038 7398 1211   2077     316 1287 118     899 0.281 0.364 0.490 0.855  122
    ## 2   2247 7962 1401   2519     369 1406  37    1335 0.316 0.414 0.539 0.953  133
    ## 3    810   20    1      2       0    1   0       1 0.100 0.143 0.100 0.243  -35
    ## 4   2196 7599 1204   1933     434 1289 152     891 0.254 0.337 0.486 0.823  111
    ## 5   2576 9217 1636   2689     509 1676 253    1475 0.292 0.393 0.514 0.907  140
    ## 6   2298 8498 1320   2461     377 1518  94     801 0.290 0.356 0.500 0.855  123
    ##    W  L  ERA ERA.  WHIP G...31 GS  SV  IP H...35 HR...36 BB...37   SO
    ## 1 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 2 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 3 47 40 2.31  187 0.998    853  0 422 903    601      82     300 1196
    ## 4 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 5 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 6 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ##    Pos.Summary yearID  playerID
    ## 1         *5/H   2023 rolensc01
    ## 2      *3H/7D9   2023 heltoto01
    ## 3           *1   2023 wagnebi02
    ## 4     *89H7D/3   2023 jonesan01
    ## 5 *9*7*5*D6H/3   2023 sheffga01
    ## 6     *453H/D6   2023  kentje01

``` r
# adding new data to old HallOfFame from Lahman
new_hof <- full_join(add_hof, HallOfFame, by = c("playerID", "yearID"))

# ordering the yearID from newest (added 2023 players) to oldest
new_hof <- new_hof[order(new_hof$yearID, decreasing = TRUE),]

head(new_hof)
```

    ##   Rk nameFirst  nameLast  YoB Votes %.vote HOFm HOFs Yrs  WAR WAR7 JAWS Jpos
    ## 1  1     Scott     Rolen  6th   297   76.3   99   40  17 70.1 43.6 56.9 55.8
    ## 2  2      Todd    Helton  5th   281   72.2  175   59  17 61.8 46.6 54.2 53.4
    ## 3  3     Billy    Wagner  8th   265   68.1  107   24  16 27.7 19.8 23.7 32.5
    ## 4  4    Andruw     Jones  6th   226   58.1  109   34  17 62.7 46.4 54.6 58.1
    ## 5  5      Gary Sheffield  9th   214   55.0  158   61  22 60.5 38.0 49.3 56.7
    ## 6  7      Jeff      Kent 10th   181   46.5  123   51  17 55.5 35.8 45.6 57.0
    ##   G...13   AB    R H...16 HR...17  RBI  SB BB...20    BA   OBP   SLG   OPS OPS.
    ## 1   2038 7398 1211   2077     316 1287 118     899 0.281 0.364 0.490 0.855  122
    ## 2   2247 7962 1401   2519     369 1406  37    1335 0.316 0.414 0.539 0.953  133
    ## 3    810   20    1      2       0    1   0       1 0.100 0.143 0.100 0.243  -35
    ## 4   2196 7599 1204   1933     434 1289 152     891 0.254 0.337 0.486 0.823  111
    ## 5   2576 9217 1636   2689     509 1676 253    1475 0.292 0.393 0.514 0.907  140
    ## 6   2298 8498 1320   2461     377 1518  94     801 0.290 0.356 0.500 0.855  123
    ##    W  L  ERA ERA.  WHIP G...31 GS  SV  IP H...35 HR...36 BB...37   SO
    ## 1 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 2 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 3 47 40 2.31  187 0.998    853  0 422 903    601      82     300 1196
    ## 4 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 5 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ## 6 NA NA   NA   NA    NA     NA NA  NA  NA     NA      NA      NA   NA
    ##    Pos.Summary yearID  playerID votedBy ballots needed votes inducted category
    ## 1         *5/H   2023 rolensc01    <NA>      NA     NA    NA     <NA>     <NA>
    ## 2      *3H/7D9   2023 heltoto01    <NA>      NA     NA    NA     <NA>     <NA>
    ## 3           *1   2023 wagnebi02    <NA>      NA     NA    NA     <NA>     <NA>
    ## 4     *89H7D/3   2023 jonesan01    <NA>      NA     NA    NA     <NA>     <NA>
    ## 5 *9*7*5*D6H/3   2023 sheffga01    <NA>      NA     NA    NA     <NA>     <NA>
    ## 6     *453H/D6   2023  kentje01    <NA>      NA     NA    NA     <NA>     <NA>
    ##   needed_note
    ## 1        <NA>
    ## 2        <NA>
    ## 3        <NA>
    ## 4        <NA>
    ## 5        <NA>
    ## 6        <NA>

``` r
# creating csv
write.csv(new_hof, file="HallOfFame.csv", row.names = FALSE)
```
