
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

# ordering the yearID from newest to oldest (to show our additions)
new_hof <- new_hof[order(new_hof$yearID, decreasing = TRUE),]

new_hof
```

    ##      Rk nameFirst  nameLast  YoB Votes %.vote HOFm HOFs Yrs   WAR WAR7 JAWS
    ## 1     1     Scott     Rolen  6th   297   76.3   99   40  17  70.1 43.6 56.9
    ## 2     2      Todd    Helton  5th   281   72.2  175   59  17  61.8 46.6 54.2
    ## 3     3     Billy    Wagner  8th   265   68.1  107   24  16  27.7 19.8 23.7
    ## 4     4    Andruw     Jones  6th   226   58.1  109   34  17  62.7 46.4 54.6
    ## 5     5      Gary Sheffield  9th   214   55.0  158   61  22  60.5 38.0 49.3
    ## 6     7      Jeff      Kent 10th   181   46.5  123   51  17  55.5 35.8 45.6
    ## 7     8      Alex Rodriguez  2nd   139   35.7  390   77  22 117.5 64.3 90.9
    ## 8     9     Manny   Ramirez  7th   129   33.2  226   69  19  69.3 39.9 54.6
    ## 9    10      Omar   Vizquel  6th    76   19.5  120   42  24  45.6 26.8 36.2
    ## 10   11      Andy  Pettitte  5th    66   17.0  128   44  18  60.2 34.1 47.2
    ## 11   12     Bobby     Abreu  4th    60   15.4   95   54  18  60.2 41.6 50.9
    ## 12   13     Jimmy   Rollins  2nd    50   12.9  121   42  17  47.6 32.7 40.1
    ## 13   14      Mark   Buehrle  3rd    42   10.8   52   31  16  59.1 35.8 47.4
    ## 14   16     Torii    Hunter  3rd    27    6.9   58   34  19  50.7 30.8 40.7
    ## 15   17    Huston    Street  1st     1    0.3   57   23  13  14.5 12.2 13.3
    ## 16   18   Bronson    Arroyo  1st     1    0.3   15   15  16  23.4 22.8 23.1
    ## 17   19      Mike    Napoli  1st     1    0.3   17   25  12  26.3 22.0 24.2
    ## 18   20      John    Lackey  1st     1    0.3   48   28  15  37.3 29.2 33.3
    ## 19   22    Jhonny   Peralta  1st     0    0.0   34   24  15  30.4 26.5 28.5
    ## 20   24     Andre    Ethier  1st     0    0.0   21   18  12  21.5 18.8 20.2
    ## 21   25    Jacoby  Ellsbury  1st     0    0.0   36   16  11  31.2 28.0 29.6
    ## 22   26      Matt      Cain  1st     0    0.0   26   14  13  29.1 29.0 29.1
    ## 23   27     Jered    Weaver  1st     0    0.0   47   30  12  34.6 31.2 32.9
    ## 24   28    Jayson     Werth  1st     0    0.0   19   23  15  29.2 27.5 28.4
    ## 4312 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4313 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4314 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4315 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4316 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4317 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4318 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4319 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4320 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4321 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4322 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4323 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4324 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4325 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4326 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4327 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4328 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4329 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4330 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4331 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4332 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4333 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4334 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4335 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4336 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4337 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4338 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4339 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4340 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4341 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4342 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4343 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4344 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4345 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4346 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4347 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4287 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4288 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4289 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4290 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4291 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4292 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4293 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4294 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4295 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4296 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4297 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4298 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4299 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4300 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4301 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4302 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4303 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4304 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4305 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4306 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4307 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4308 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4309 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4310 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4311 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4253 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4254 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4255 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4256 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4257 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4258 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4259 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4260 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4261 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4262 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4263 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4264 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4265 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4266 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4267 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4268 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4269 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4270 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4271 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4272 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4273 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4274 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4275 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4276 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4277 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4278 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4279 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4280 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4281 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4282 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4283 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4284 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4285 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4286 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4216 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4217 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4218 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4219 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4220 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4221 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4222 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4223 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4224 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4225 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4226 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4227 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4228 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4229 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4230 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4231 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4232 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4233 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4234 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4235 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4236 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4237 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4238 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4239 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4240 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4241 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4242 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4243 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4244 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4245 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4246 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4247 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4248 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4249 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4250 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4251 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4252 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4181 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4182 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4183 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4184 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4185 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4186 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4187 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4188 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4189 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4190 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4191 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4192 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4193 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4194 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4195 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4196 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4197 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4198 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4199 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4200 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4201 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4202 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4203 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4204 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4205 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4206 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4207 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4208 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4209 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4210 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4211 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4212 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4213 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4214 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4215 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4145 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4146 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4147 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4148 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4149 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4150 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4151 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4152 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4153 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4154 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4155 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4156 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4157 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4158 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4159 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4160 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4161 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4162 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4163 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4164 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4165 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4166 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4167 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4168 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4169 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4170 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4171 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4172 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4173 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4174 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4175 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4176 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4177 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4178 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4179 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4180 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4113 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4114 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4115 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4116 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4117 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4118 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4119 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4120 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4121 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4122 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4123 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4124 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4125 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4126 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4127 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4128 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4129 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4130 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4131 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4132 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4133 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4134 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4135 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4136 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4137 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4138 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4139 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4140 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4141 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4142 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4143 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4144 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4079 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4080 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4081 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4082 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4083 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4084 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4085 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4086 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4087 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4088 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4089 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4090 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4091 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4092 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4093 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4094 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4095 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4096 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4097 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4098 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4099 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4100 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4101 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4102 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4103 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4104 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4105 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4106 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4107 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4108 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4109 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4110 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4111 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4112 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4040 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4041 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4042 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4043 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4044 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4045 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4046 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4047 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4048 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4049 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4050 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4051 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4052 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4053 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4054 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4055 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4056 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4057 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4058 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4059 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4060 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4061 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4062 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4063 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4064 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4065 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4066 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4067 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4068 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4069 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4070 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4071 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4072 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4073 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4074 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4075 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4076 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4077 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4078 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4000 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4001 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4002 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4003 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4004 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4005 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4006 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4007 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4008 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4009 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4010 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4011 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4012 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4013 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4014 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4015 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4016 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4017 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4018 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4019 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4020 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4021 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4022 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4023 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4024 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4025 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4026 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4027 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4028 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4029 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4030 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4031 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4032 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4033 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4034 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4035 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4036 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4037 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4038 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 4039 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3972 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3973 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3974 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3975 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3976 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3977 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3978 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3979 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3980 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3981 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3982 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3983 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3984 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3985 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3986 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3987 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3988 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3989 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3990 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3991 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3992 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3993 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3994 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3995 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3996 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3997 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3998 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3999 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3938 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3939 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3940 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3941 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3942 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3943 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3944 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3945 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3946 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3947 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3948 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3949 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3950 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3951 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3952 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3953 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3954 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3955 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3956 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3957 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3958 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3959 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3960 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3961 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3962 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3963 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3964 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3965 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3966 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3967 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3968 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3969 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3970 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3971 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3910 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3911 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3912 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3913 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3914 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3915 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3916 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3917 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3918 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3919 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3920 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3921 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3922 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3923 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3924 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3925 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3926 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3927 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3928 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3929 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3930 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3931 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3932 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3933 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3934 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3935 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3936 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3937 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3886 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3887 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3888 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3889 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3890 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3891 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3892 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3893 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3894 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3895 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3896 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3897 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3898 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3899 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3900 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3901 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3902 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3903 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3904 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3905 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3906 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3907 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3908 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3909 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3856 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3857 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3858 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3859 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3860 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3861 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3862 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3863 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3864 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3865 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3866 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3867 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3868 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3869 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3870 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3871 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3872 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3873 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3874 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3875 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3876 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3877 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3878 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3879 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3880 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3881 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3882 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3883 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3884 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3885 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3824 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3825 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3826 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3827 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3828 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3829 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3830 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3831 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3832 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3833 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3834 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3835 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3836 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3837 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3838 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3839 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3840 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3841 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3842 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3843 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3844 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3845 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3846 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3847 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3848 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3849 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3850 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3851 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3852 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3853 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3854 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3855 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3778 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3779 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3780 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3781 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3782 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3783 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3784 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3785 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3786 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3787 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3788 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3789 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3790 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3791 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3792 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3793 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3794 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3795 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3796 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3797 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3798 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3799 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3800 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3801 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3802 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3803 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3804 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3805 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3806 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3807 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3808 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3809 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3810 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3811 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3812 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3813 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3814 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3815 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3816 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3817 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3818 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3819 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3820 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3821 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3822 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3823 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3751 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3752 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3753 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3754 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3755 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3756 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3757 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3758 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3759 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3760 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3761 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3762 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3763 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3764 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3765 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3766 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3767 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3768 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3769 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3770 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3771 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3772 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3773 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3774 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3775 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3776 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3777 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3719 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3720 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3721 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3722 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3723 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3724 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3725 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3726 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3727 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3728 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3729 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3730 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3731 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3732 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3733 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3734 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3735 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3736 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3737 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3738 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3739 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3740 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3741 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3742 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3743 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3744 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3745 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3746 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3747 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3748 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3749 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3750 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3686 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3687 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3688 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3689 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3690 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3691 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3692 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3693 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3694 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3695 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3696 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3697 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3698 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3699 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3700 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3701 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3702 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3703 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3704 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3705 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3706 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3707 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3708 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3709 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3710 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3711 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3712 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3713 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3714 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3715 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3716 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3717 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3718 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3658 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3659 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3660 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3661 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3662 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3663 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3664 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3665 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3666 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3667 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3668 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3669 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3670 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3671 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3672 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3673 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3674 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3675 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3676 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3677 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3678 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3679 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3680 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3681 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3682 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3683 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3684 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3685 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3624 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3625 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3626 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3627 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3628 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3629 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3630 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3631 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3632 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3633 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3634 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3635 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3636 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3637 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3638 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3639 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3640 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3641 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3642 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3643 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3644 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3645 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3646 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3647 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3648 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3649 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3650 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3651 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3652 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3653 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3654 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3655 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3656 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3657 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3591 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3592 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3593 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3594 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3595 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3596 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3597 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3598 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3599 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3600 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3601 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3602 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3603 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3604 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3605 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3606 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3607 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3608 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3609 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3610 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3611 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3612 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3613 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3614 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3615 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3616 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3617 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3618 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3619 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3620 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3621 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3622 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3623 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3559 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3560 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3561 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3562 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3563 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3564 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3565 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3566 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3567 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3568 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3569 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3570 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3571 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3572 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3573 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3574 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3575 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3576 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3577 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3578 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3579 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3580 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3581 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3582 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3583 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3584 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3585 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3586 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3587 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3588 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3589 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3590 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3529 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3530 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3531 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3532 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3533 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3534 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3535 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3536 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3537 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3538 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3539 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3540 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3541 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3542 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3543 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3544 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3545 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3546 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3547 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3548 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3549 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3550 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3551 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3552 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3553 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3554 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3555 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3556 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3557 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3558 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3496 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3497 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3498 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3499 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3500 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3501 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3502 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3503 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3504 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3505 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3506 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3507 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3508 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3509 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3510 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3511 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3512 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3513 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3514 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3515 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3516 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3517 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3518 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3519 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3520 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3521 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3522 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3523 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3524 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3525 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3526 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3527 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3528 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3457 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3458 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3459 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3460 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3461 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3462 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3463 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3464 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3465 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3466 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3467 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3468 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3469 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3470 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3471 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3472 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3473 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3474 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3475 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3476 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3477 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3478 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3479 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3480 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3481 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3482 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3483 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3484 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3485 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3486 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3487 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3488 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3489 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3490 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3491 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3492 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3493 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3494 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3495 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3414 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3415 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3416 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3417 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3418 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3419 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3420 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3421 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3422 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3423 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3424 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3425 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3426 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3427 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3428 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3429 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3430 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3431 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3432 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3433 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3434 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3435 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3436 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3437 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3438 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3439 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3440 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3441 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3442 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3443 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3444 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3445 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3446 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3447 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3448 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3449 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3450 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3451 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3452 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3453 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3454 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3455 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3456 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3373 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3374 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3375 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3376 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3377 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3378 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3379 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3380 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3381 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3382 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3383 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3384 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3385 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3386 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3387 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3388 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3389 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3390 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3391 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3392 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3393 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3394 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3395 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3396 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3397 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3398 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3399 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3400 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3401 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3402 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3403 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3404 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3405 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3406 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3407 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3408 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3409 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3410 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3411 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3412 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3413 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3340 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3341 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3342 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3343 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3344 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3345 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3346 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3347 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3348 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3349 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3350 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3351 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3352 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3353 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3354 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3355 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3356 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3357 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3358 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3359 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3360 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3361 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3362 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3363 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3364 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3365 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3366 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3367 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3368 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3369 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3370 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3371 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3372 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3301 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3302 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3303 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3304 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3305 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3306 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3307 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3308 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3309 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3310 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3311 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3312 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3313 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3314 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3315 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3316 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3317 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3318 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3319 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3320 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3321 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3322 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3323 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3324 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3325 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3326 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3327 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3328 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3329 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3330 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3331 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3332 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3333 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3334 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3335 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3336 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3337 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3338 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3339 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3254 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3255 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3256 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3257 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3258 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3259 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3260 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3261 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3262 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3263 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3264 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3265 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3266 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3267 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3268 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3269 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3270 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3271 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3272 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3273 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3274 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3275 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3276 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3277 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3278 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3279 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3280 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3281 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3282 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3283 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3284 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3285 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3286 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3287 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3288 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3289 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3290 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3291 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3292 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3293 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3294 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3295 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3296 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3297 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3298 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3299 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3300 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3210 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3211 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3212 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3213 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3214 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3215 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3216 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3217 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3218 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3219 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3220 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3221 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3222 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3223 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3224 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3225 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3226 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3227 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3228 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3229 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3230 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3231 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3232 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3233 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3234 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3235 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3236 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3237 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3238 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3239 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3240 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3241 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3242 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3243 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3244 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3245 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3246 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3247 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3248 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3249 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3250 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3251 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3252 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3253 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3167 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3168 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3169 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3170 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3171 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3172 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3173 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3174 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3175 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3176 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3177 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3178 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3179 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3180 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3181 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3182 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3183 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3184 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3185 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3186 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3187 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3188 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3189 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3190 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3191 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3192 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3193 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3194 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3195 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3196 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3197 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3198 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3199 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3200 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3201 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3202 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3203 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3204 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3205 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3206 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3207 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3208 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3209 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3123 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3124 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3125 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3126 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3127 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3128 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3129 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3130 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3131 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3132 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3133 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3134 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3135 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3136 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3137 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3138 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3139 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3140 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3141 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3142 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3143 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3144 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3145 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3146 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3147 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3148 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3149 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3150 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3151 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3152 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3153 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3154 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3155 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3156 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3157 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3158 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3159 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3160 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3161 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3162 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3163 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3164 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3165 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3166 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3094 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3095 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3096 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3097 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3098 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3099 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3100 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3101 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3102 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3103 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3104 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3105 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3106 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3107 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3108 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3109 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3110 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3111 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3112 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3113 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3114 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3115 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3116 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3117 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3118 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3119 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3120 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3121 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3122 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3051 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3052 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3053 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3054 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3055 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3056 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3057 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3058 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3059 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3060 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3061 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3062 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3063 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3064 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3065 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3066 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3067 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3068 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3069 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3070 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3071 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3072 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3073 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3074 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3075 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3076 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3077 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3078 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3079 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3080 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3081 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3082 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3083 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3084 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3085 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3086 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3087 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3088 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3089 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3090 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3091 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3092 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3093 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3008 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3009 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3010 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3011 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3012 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3013 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3014 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3015 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3016 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3017 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3018 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3019 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3020 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3021 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3022 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3023 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3024 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3025 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3026 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3027 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3028 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3029 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3030 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3031 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3032 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3033 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3034 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3035 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3036 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3037 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3038 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3039 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3040 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3041 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3042 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3043 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3044 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3045 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3046 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3047 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3048 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3049 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3050 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2977 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2978 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2979 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2980 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2981 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2982 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2983 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2984 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2985 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2986 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2987 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2988 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2989 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2990 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2991 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2992 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2993 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2994 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2995 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2996 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2997 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2998 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2999 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3000 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3001 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3002 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3003 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3004 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3005 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3006 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 3007 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2929 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2930 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2931 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2932 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2933 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2934 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2935 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2936 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2937 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2938 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2939 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2940 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2941 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2942 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2943 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2944 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2945 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2946 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2947 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2948 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2949 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2950 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2951 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2952 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2953 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2954 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2955 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2956 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2957 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2958 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2959 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2960 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2961 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2962 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2963 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2964 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2965 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2966 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2967 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2968 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2969 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2970 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2971 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2972 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2973 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2974 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2975 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2976 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2885 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2886 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2887 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2888 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2889 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2890 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2891 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2892 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2893 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2894 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2895 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2896 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2897 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2898 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2899 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2900 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2901 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2902 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2903 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2904 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2905 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2906 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2907 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2908 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2909 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2910 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2911 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2912 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2913 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2914 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2915 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2916 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2917 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2918 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2919 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2920 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2921 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2922 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2923 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2924 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2925 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2926 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2927 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2928 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2844 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2845 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2846 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2847 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2848 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2849 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2850 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2851 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2852 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2853 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2854 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2855 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2856 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2857 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2858 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2859 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2860 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2861 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2862 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2863 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2864 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2865 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2866 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2867 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2868 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2869 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2870 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2871 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2872 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2873 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2874 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2875 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2876 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2877 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2878 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2879 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2880 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2881 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2882 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2883 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2884 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2781 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2782 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2783 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2784 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2785 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2786 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2787 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2788 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2789 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2790 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2791 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2792 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2793 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2794 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2795 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2796 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2797 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2798 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2799 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2800 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2801 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2802 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2803 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2804 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2805 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2806 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2807 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2808 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2809 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2810 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2811 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2812 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2813 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2814 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2815 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2816 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2817 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2818 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2819 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2820 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2821 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2822 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2823 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2824 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2825 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2826 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2827 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2828 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2829 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2830 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2831 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2832 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2833 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2834 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2835 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2836 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2837 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2838 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2839 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2840 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2841 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2842 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2843 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2725 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2726 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2727 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2728 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2729 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2730 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2731 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2732 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2733 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2734 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2735 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2736 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2737 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2738 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2739 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2740 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2741 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2742 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2743 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2744 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2745 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2746 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2747 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2748 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2749 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2750 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2751 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2752 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2753 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2754 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2755 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2756 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2757 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2758 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2759 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2760 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2761 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2762 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2763 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2764 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2765 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2766 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2767 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2768 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2769 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2770 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2771 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2772 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2773 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2774 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2775 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2776 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2777 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2778 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2779 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2780 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2686 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2687 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2688 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2689 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2690 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2691 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2692 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2693 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2694 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2695 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2696 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2697 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2698 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2699 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2700 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2701 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2702 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2703 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2704 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2705 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2706 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2707 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2708 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2709 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2710 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2711 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2712 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2713 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2714 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2715 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2716 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2717 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2718 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2719 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2720 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2721 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2722 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2723 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2724 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2647 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2648 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2649 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2650 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2651 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2652 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2653 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2654 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2655 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2656 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2657 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2658 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2659 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2660 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2661 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2662 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2663 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2664 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2665 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2666 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2667 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2668 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2669 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2670 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2671 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2672 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2673 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2674 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2675 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2676 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2677 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2678 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2679 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2680 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2681 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2682 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2683 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2684 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2685 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2611 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2612 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2613 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2614 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2615 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2616 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2617 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2618 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2619 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2620 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2621 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2622 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2623 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2624 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2625 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2626 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2627 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2628 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2629 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2630 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2631 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2632 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2633 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2634 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2635 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2636 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2637 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2638 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2639 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2640 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2641 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2642 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2643 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2644 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2645 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2646 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2570 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2571 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2572 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2573 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2574 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2575 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2576 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2577 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2578 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2579 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2580 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2581 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2582 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2583 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2584 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2585 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2586 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2587 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2588 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2589 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2590 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2591 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2592 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2593 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2594 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2595 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2596 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2597 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2598 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2599 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2600 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2601 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2602 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2603 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2604 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2605 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2606 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2607 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2608 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2609 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2610 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2522 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2523 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2524 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2525 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2526 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2527 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2528 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2529 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2530 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2531 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2532 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2533 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2534 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2535 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2536 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2537 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2538 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2539 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2540 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2541 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2542 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2543 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2544 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2545 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2546 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2547 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2548 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2549 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2550 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2551 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2552 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2553 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2554 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2555 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2556 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2557 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2558 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2559 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2560 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2561 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2562 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2563 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2564 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2565 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2566 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2567 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2568 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2569 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2473 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2474 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2475 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2476 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2477 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2478 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2479 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2480 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2481 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2482 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2483 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2484 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2485 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2486 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2487 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2488 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2489 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2490 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2491 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2492 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2493 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2494 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2495 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2496 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2497 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2498 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2499 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2500 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2501 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2502 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2503 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2504 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2505 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2506 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2507 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2508 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2509 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2510 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2511 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2512 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2513 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2514 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2515 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2516 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2517 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2518 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2519 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2520 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2521 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2422 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2423 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2424 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2425 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2426 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2427 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2428 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2429 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2430 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2431 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2432 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2433 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2434 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2435 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2436 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2437 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2438 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2439 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2440 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2441 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2442 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2443 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2444 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2445 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2446 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2447 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2448 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2449 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2450 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2451 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2452 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2453 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2454 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2455 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2456 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2457 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2458 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2459 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2460 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2461 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2462 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2463 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2464 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2465 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2466 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2467 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2468 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2469 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2470 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2471 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2472 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2366 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2367 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2368 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2369 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2370 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2371 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2372 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2373 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2374 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2375 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2376 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2377 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2378 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2379 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2380 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2381 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2382 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2383 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2384 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2385 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2386 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2387 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2388 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2389 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2390 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2391 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2392 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2393 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2394 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2395 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2396 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2397 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2398 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2399 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2400 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2401 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2402 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2403 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2404 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2405 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2406 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2407 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2408 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2409 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2410 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2411 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2412 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2413 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2414 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2415 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2416 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2417 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2418 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2419 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2420 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2421 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2317 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2318 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2319 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2320 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2321 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2322 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2323 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2324 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2325 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2326 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2327 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2328 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2329 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2330 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2331 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2332 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2333 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2334 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2335 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2336 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2337 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2338 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2339 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2340 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2341 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2342 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2343 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2344 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2345 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2346 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2347 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2348 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2349 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ## 2350 NA      <NA>      <NA> <NA>    NA     NA   NA   NA  NA    NA   NA   NA
    ##      Jpos G...13    AB    R H...16 HR...17  RBI  SB BB...20    BA   OBP   SLG
    ## 1    55.8   2038  7398 1211   2077     316 1287 118     899 0.281 0.364 0.490
    ## 2    53.4   2247  7962 1401   2519     369 1406  37    1335 0.316 0.414 0.539
    ## 3    32.5    810    20    1      2       0    1   0       1 0.100 0.143 0.100
    ## 4    58.1   2196  7599 1204   1933     434 1289 152     891 0.254 0.337 0.486
    ## 5    56.7   2576  9217 1636   2689     509 1676 253    1475 0.292 0.393 0.514
    ## 6    57.0   2298  8498 1320   2461     377 1518  94     801 0.290 0.356 0.500
    ## 7    55.4   2784 10566 2021   3115     696 2086 329    1338 0.295 0.380 0.550
    ## 8    53.4   2302  8244 1544   2574     555 1831  38    1329 0.312 0.411 0.585
    ## 9    55.4   2968 10586 1445   2877      80  951 404    1028 0.272 0.336 0.352
    ## 10   61.4    108   196    6     27       1   13   0       6 0.138 0.163 0.184
    ## 11   56.7   2425  8480 1453   2470     288 1363 400    1476 0.291 0.395 0.475
    ## 12   55.4   2275  9294 1421   2455     231  936 470     813 0.264 0.324 0.418
    ## 13   61.4     55   125    4      9       1    3   0       2 0.072 0.087 0.112
    ## 14   58.1   2372  8857 1296   2452     353 1391 195     661 0.277 0.331 0.461
    ## 15   32.5    304     2    0      0       0    0   0       0 0.000 0.000 0.000
    ## 16   61.4    353   629   35     81       6   29   1      14 0.129 0.148 0.183
    ## 17   44.2   1392  4572  697   1125     267  744  39     650 0.246 0.346 0.475
    ## 18   61.4    121   236    6     26       0    8   1       9 0.110 0.143 0.140
    ## 19   55.4   1798  6599  841   1761     202  873  17     606 0.267 0.329 0.423
    ## 20   56.7   1455  4800  641   1367     162  687  29     519 0.285 0.359 0.463
    ## 21   58.1   1235  4846  749   1376     104  512 343     399 0.284 0.342 0.417
    ## 22   61.4    332   611   27     74       7   34   0      21 0.121 0.151 0.180
    ## 23   61.4     28    55    3      4       0    1   0       2 0.073 0.105 0.073
    ## 24   56.7   1583  5484  883   1465     229  799 132     764 0.267 0.360 0.455
    ## 4312   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4313   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4314   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4315   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4316   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4317   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4318   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4319   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4320   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4321   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4322   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4323   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4324   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4325   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4326   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4327   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4328   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4329   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4330   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4331   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4332   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4333   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4334   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4335   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4336   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4337   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4338   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4339   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4340   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4341   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4342   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4343   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4344   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4345   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4346   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4347   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4287   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4288   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4289   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4290   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4291   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4292   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4293   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4294   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4295   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4296   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4297   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4298   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4299   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4300   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4301   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4302   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4303   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4304   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4305   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4306   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4307   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4308   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4309   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4310   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4311   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4253   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4254   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4255   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4256   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4257   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4258   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4259   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4260   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4261   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4262   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4263   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4264   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4265   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4266   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4267   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4268   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4269   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4270   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4271   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4272   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4273   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4274   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4275   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4276   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4277   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4278   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4279   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4280   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4281   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4282   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4283   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4284   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4285   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4286   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4216   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4217   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4218   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4219   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4220   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4221   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4222   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4223   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4224   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4225   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4226   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4227   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4228   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4229   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4230   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4231   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4232   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4233   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4234   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4235   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4236   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4237   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4238   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4239   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4240   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4241   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4242   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4243   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4244   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4245   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4246   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4247   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4248   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4249   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4250   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4251   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4252   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4181   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4182   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4183   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4184   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4185   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4186   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4187   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4188   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4189   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4190   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4191   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4192   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4193   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4194   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4195   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4196   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4197   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4198   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4199   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4200   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4201   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4202   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4203   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4204   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4205   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4206   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4207   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4208   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4209   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4210   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4211   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4212   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4213   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4214   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4215   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4145   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4146   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4147   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4148   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4149   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4150   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4151   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4152   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4153   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4154   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4155   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4156   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4157   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4158   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4159   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4160   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4161   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4162   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4163   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4164   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4165   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4166   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4167   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4168   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4169   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4170   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4171   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4172   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4173   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4174   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4175   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4176   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4177   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4178   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4179   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4180   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4113   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4114   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4115   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4116   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4117   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4118   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4119   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4120   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4121   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4122   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4123   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4124   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4125   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4126   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4127   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4128   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4129   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4130   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4131   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4132   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4133   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4134   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4135   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4136   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4137   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4138   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4139   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4140   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4141   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4142   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4143   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4144   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4079   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4080   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4081   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4082   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4083   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4084   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4085   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4086   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4087   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4088   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4089   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4090   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4091   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4092   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4093   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4094   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4095   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4096   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4097   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4098   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4099   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4100   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4101   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4102   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4103   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4104   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4105   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4106   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4107   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4108   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4109   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4110   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4111   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4112   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4040   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4041   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4042   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4043   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4044   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4045   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4046   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4047   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4048   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4049   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4050   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4051   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4052   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4053   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4054   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4055   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4056   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4057   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4058   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4059   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4060   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4061   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4062   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4063   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4064   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4065   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4066   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4067   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4068   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4069   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4070   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4071   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4072   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4073   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4074   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4075   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4076   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4077   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4078   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4000   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4001   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4002   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4003   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4004   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4005   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4006   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4007   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4008   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4009   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4010   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4011   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4012   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4013   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4014   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4015   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4016   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4017   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4018   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4019   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4020   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4021   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4022   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4023   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4024   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4025   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4026   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4027   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4028   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4029   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4030   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4031   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4032   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4033   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4034   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4035   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4036   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4037   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4038   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 4039   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3972   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3973   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3974   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3975   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3976   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3977   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3978   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3979   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3980   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3981   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3982   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3983   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3984   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3985   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3986   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3987   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3988   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3989   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3990   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3991   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3992   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3993   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3994   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3995   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3996   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3997   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3998   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3999   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3938   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3939   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3940   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3941   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3942   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3943   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3944   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3945   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3946   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3947   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3948   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3949   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3950   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3951   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3952   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3953   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3954   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3955   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3956   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3957   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3958   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3959   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3960   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3961   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3962   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3963   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3964   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3965   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3966   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3967   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3968   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3969   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3970   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3971   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3910   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3911   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3912   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3913   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3914   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3915   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3916   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3917   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3918   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3919   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3920   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3921   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3922   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3923   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3924   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3925   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3926   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3927   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3928   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3929   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3930   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3931   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3932   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3933   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3934   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3935   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3936   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3937   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3886   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3887   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3888   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3889   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3890   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3891   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3892   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3893   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3894   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3895   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3896   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3897   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3898   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3899   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3900   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3901   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3902   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3903   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3904   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3905   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3906   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3907   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3908   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3909   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3856   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3857   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3858   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3859   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3860   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3861   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3862   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3863   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3864   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3865   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3866   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3867   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3868   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3869   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3870   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3871   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3872   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3873   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3874   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3875   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3876   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3877   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3878   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3879   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3880   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3881   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3882   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3883   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3884   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3885   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3824   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3825   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3826   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3827   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3828   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3829   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3830   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3831   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3832   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3833   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3834   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3835   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3836   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3837   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3838   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3839   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3840   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3841   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3842   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3843   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3844   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3845   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3846   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3847   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3848   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3849   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3850   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3851   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3852   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3853   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3854   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3855   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3778   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3779   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3780   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3781   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3782   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3783   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3784   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3785   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3786   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3787   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3788   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3789   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3790   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3791   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3792   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3793   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3794   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3795   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3796   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3797   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3798   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3799   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3800   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3801   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3802   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3803   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3804   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3805   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3806   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3807   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3808   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3809   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3810   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3811   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3812   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3813   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3814   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3815   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3816   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3817   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3818   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3819   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3820   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3821   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3822   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3823   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3751   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3752   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3753   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3754   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3755   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3756   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3757   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3758   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3759   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3760   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3761   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3762   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3763   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3764   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3765   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3766   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3767   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3768   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3769   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3770   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3771   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3772   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3773   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3774   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3775   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3776   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3777   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3719   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3720   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3721   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3722   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3723   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3724   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3725   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3726   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3727   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3728   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3729   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3730   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3731   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3732   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3733   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3734   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3735   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3736   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3737   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3738   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3739   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3740   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3741   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3742   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3743   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3744   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3745   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3746   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3747   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3748   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3749   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3750   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3686   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3687   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3688   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3689   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3690   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3691   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3692   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3693   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3694   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3695   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3696   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3697   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3698   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3699   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3700   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3701   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3702   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3703   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3704   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3705   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3706   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3707   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3708   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3709   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3710   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3711   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3712   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3713   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3714   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3715   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3716   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3717   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3718   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3658   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3659   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3660   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3661   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3662   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3663   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3664   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3665   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3666   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3667   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3668   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3669   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3670   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3671   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3672   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3673   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3674   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3675   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3676   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3677   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3678   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3679   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3680   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3681   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3682   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3683   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3684   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3685   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3624   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3625   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3626   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3627   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3628   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3629   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3630   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3631   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3632   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3633   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3634   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3635   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3636   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3637   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3638   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3639   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3640   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3641   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3642   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3643   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3644   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3645   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3646   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3647   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3648   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3649   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3650   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3651   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3652   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3653   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3654   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3655   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3656   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3657   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3591   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3592   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3593   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3594   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3595   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3596   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3597   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3598   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3599   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3600   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3601   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3602   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3603   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3604   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3605   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3606   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3607   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3608   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3609   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3610   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3611   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3612   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3613   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3614   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3615   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3616   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3617   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3618   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3619   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3620   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3621   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3622   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3623   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3559   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3560   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3561   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3562   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3563   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3564   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3565   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3566   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3567   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3568   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3569   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3570   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3571   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3572   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3573   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3574   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3575   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3576   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3577   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3578   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3579   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3580   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3581   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3582   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3583   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3584   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3585   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3586   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3587   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3588   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3589   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3590   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3529   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3530   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3531   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3532   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3533   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3534   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3535   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3536   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3537   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3538   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3539   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3540   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3541   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3542   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3543   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3544   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3545   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3546   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3547   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3548   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3549   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3550   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3551   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3552   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3553   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3554   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3555   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3556   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3557   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3558   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3496   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3497   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3498   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3499   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3500   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3501   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3502   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3503   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3504   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3505   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3506   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3507   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3508   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3509   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3510   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3511   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3512   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3513   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3514   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3515   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3516   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3517   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3518   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3519   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3520   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3521   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3522   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3523   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3524   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3525   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3526   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3527   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3528   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3457   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3458   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3459   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3460   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3461   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3462   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3463   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3464   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3465   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3466   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3467   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3468   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3469   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3470   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3471   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3472   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3473   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3474   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3475   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3476   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3477   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3478   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3479   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3480   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3481   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3482   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3483   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3484   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3485   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3486   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3487   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3488   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3489   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3490   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3491   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3492   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3493   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3494   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3495   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3414   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3415   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3416   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3417   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3418   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3419   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3420   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3421   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3422   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3423   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3424   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3425   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3426   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3427   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3428   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3429   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3430   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3431   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3432   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3433   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3434   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3435   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3436   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3437   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3438   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3439   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3440   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3441   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3442   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3443   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3444   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3445   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3446   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3447   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3448   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3449   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3450   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3451   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3452   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3453   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3454   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3455   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3456   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3373   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3374   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3375   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3376   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3377   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3378   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3379   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3380   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3381   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3382   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3383   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3384   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3385   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3386   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3387   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3388   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3389   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3390   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3391   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3392   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3393   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3394   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3395   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3396   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3397   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3398   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3399   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3400   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3401   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3402   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3403   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3404   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3405   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3406   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3407   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3408   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3409   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3410   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3411   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3412   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3413   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3340   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3341   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3342   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3343   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3344   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3345   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3346   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3347   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3348   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3349   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3350   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3351   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3352   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3353   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3354   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3355   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3356   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3357   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3358   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3359   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3360   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3361   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3362   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3363   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3364   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3365   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3366   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3367   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3368   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3369   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3370   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3371   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3372   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3301   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3302   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3303   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3304   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3305   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3306   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3307   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3308   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3309   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3310   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3311   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3312   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3313   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3314   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3315   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3316   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3317   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3318   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3319   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3320   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3321   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3322   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3323   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3324   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3325   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3326   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3327   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3328   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3329   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3330   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3331   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3332   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3333   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3334   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3335   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3336   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3337   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3338   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3339   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3254   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3255   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3256   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3257   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3258   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3259   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3260   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3261   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3262   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3263   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3264   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3265   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3266   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3267   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3268   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3269   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3270   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3271   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3272   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3273   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3274   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3275   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3276   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3277   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3278   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3279   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3280   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3281   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3282   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3283   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3284   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3285   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3286   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3287   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3288   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3289   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3290   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3291   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3292   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3293   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3294   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3295   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3296   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3297   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3298   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3299   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3300   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3210   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3211   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3212   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3213   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3214   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3215   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3216   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3217   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3218   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3219   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3220   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3221   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3222   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3223   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3224   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3225   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3226   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3227   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3228   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3229   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3230   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3231   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3232   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3233   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3234   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3235   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3236   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3237   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3238   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3239   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3240   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3241   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3242   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3243   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3244   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3245   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3246   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3247   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3248   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3249   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3250   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3251   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3252   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3253   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3167   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3168   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3169   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3170   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3171   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3172   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3173   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3174   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3175   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3176   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3177   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3178   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3179   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3180   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3181   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3182   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3183   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3184   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3185   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3186   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3187   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3188   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3189   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3190   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3191   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3192   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3193   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3194   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3195   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3196   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3197   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3198   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3199   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3200   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3201   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3202   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3203   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3204   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3205   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3206   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3207   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3208   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3209   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3123   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3124   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3125   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3126   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3127   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3128   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3129   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3130   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3131   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3132   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3133   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3134   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3135   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3136   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3137   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3138   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3139   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3140   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3141   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3142   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3143   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3144   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3145   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3146   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3147   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3148   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3149   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3150   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3151   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3152   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3153   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3154   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3155   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3156   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3157   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3158   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3159   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3160   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3161   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3162   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3163   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3164   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3165   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3166   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3094   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3095   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3096   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3097   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3098   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3099   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3100   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3101   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3102   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3103   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3104   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3105   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3106   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3107   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3108   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3109   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3110   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3111   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3112   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3113   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3114   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3115   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3116   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3117   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3118   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3119   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3120   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3121   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3122   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3051   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3052   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3053   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3054   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3055   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3056   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3057   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3058   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3059   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3060   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3061   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3062   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3063   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3064   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3065   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3066   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3067   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3068   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3069   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3070   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3071   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3072   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3073   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3074   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3075   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3076   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3077   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3078   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3079   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3080   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3081   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3082   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3083   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3084   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3085   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3086   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3087   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3088   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3089   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3090   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3091   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3092   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3093   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3008   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3009   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3010   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3011   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3012   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3013   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3014   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3015   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3016   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3017   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3018   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3019   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3020   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3021   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3022   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3023   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3024   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3025   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3026   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3027   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3028   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3029   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3030   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3031   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3032   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3033   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3034   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3035   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3036   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3037   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3038   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3039   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3040   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3041   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3042   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3043   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3044   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3045   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3046   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3047   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3048   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3049   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3050   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2977   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2978   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2979   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2980   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2981   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2982   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2983   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2984   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2985   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2986   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2987   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2988   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2989   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2990   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2991   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2992   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2993   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2994   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2995   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2996   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2997   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2998   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2999   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3000   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3001   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3002   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3003   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3004   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3005   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3006   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 3007   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2929   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2930   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2931   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2932   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2933   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2934   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2935   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2936   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2937   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2938   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2939   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2940   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2941   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2942   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2943   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2944   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2945   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2946   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2947   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2948   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2949   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2950   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2951   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2952   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2953   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2954   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2955   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2956   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2957   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2958   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2959   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2960   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2961   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2962   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2963   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2964   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2965   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2966   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2967   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2968   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2969   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2970   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2971   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2972   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2973   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2974   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2975   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2976   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2885   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2886   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2887   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2888   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2889   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2890   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2891   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2892   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2893   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2894   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2895   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2896   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2897   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2898   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2899   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2900   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2901   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2902   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2903   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2904   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2905   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2906   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2907   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2908   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2909   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2910   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2911   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2912   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2913   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2914   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2915   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2916   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2917   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2918   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2919   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2920   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2921   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2922   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2923   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2924   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2925   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2926   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2927   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2928   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2844   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2845   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2846   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2847   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2848   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2849   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2850   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2851   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2852   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2853   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2854   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2855   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2856   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2857   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2858   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2859   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2860   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2861   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2862   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2863   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2864   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2865   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2866   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2867   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2868   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2869   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2870   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2871   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2872   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2873   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2874   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2875   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2876   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2877   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2878   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2879   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2880   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2881   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2882   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2883   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2884   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2781   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2782   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2783   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2784   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2785   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2786   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2787   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2788   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2789   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2790   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2791   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2792   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2793   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2794   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2795   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2796   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2797   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2798   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2799   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2800   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2801   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2802   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2803   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2804   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2805   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2806   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2807   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2808   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2809   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2810   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2811   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2812   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2813   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2814   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2815   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2816   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2817   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2818   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2819   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2820   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2821   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2822   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2823   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2824   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2825   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2826   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2827   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2828   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2829   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2830   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2831   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2832   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2833   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2834   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2835   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2836   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2837   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2838   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2839   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2840   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2841   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2842   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2843   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2725   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2726   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2727   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2728   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2729   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2730   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2731   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2732   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2733   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2734   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2735   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2736   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2737   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2738   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2739   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2740   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2741   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2742   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2743   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2744   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2745   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2746   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2747   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2748   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2749   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2750   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2751   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2752   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2753   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2754   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2755   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2756   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2757   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2758   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2759   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2760   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2761   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2762   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2763   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2764   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2765   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2766   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2767   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2768   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2769   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2770   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2771   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2772   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2773   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2774   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2775   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2776   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2777   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2778   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2779   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2780   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2686   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2687   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2688   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2689   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2690   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2691   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2692   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2693   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2694   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2695   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2696   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2697   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2698   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2699   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2700   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2701   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2702   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2703   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2704   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2705   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2706   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2707   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2708   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2709   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2710   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2711   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2712   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2713   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2714   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2715   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2716   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2717   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2718   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2719   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2720   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2721   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2722   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2723   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2724   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2647   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2648   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2649   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2650   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2651   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2652   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2653   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2654   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2655   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2656   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2657   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2658   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2659   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2660   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2661   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2662   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2663   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2664   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2665   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2666   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2667   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2668   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2669   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2670   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2671   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2672   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2673   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2674   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2675   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2676   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2677   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2678   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2679   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2680   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2681   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2682   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2683   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2684   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2685   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2611   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2612   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2613   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2614   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2615   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2616   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2617   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2618   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2619   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2620   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2621   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2622   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2623   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2624   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2625   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2626   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2627   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2628   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2629   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2630   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2631   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2632   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2633   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2634   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2635   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2636   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2637   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2638   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2639   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2640   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2641   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2642   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2643   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2644   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2645   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2646   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2570   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2571   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2572   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2573   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2574   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2575   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2576   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2577   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2578   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2579   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2580   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2581   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2582   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2583   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2584   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2585   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2586   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2587   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2588   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2589   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2590   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2591   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2592   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2593   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2594   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2595   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2596   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2597   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2598   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2599   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2600   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2601   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2602   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2603   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2604   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2605   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2606   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2607   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2608   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2609   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2610   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2522   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2523   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2524   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2525   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2526   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2527   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2528   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2529   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2530   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2531   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2532   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2533   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2534   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2535   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2536   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2537   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2538   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2539   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2540   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2541   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2542   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2543   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2544   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2545   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2546   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2547   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2548   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2549   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2550   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2551   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2552   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2553   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2554   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2555   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2556   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2557   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2558   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2559   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2560   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2561   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2562   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2563   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2564   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2565   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2566   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2567   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2568   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2569   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2473   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2474   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2475   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2476   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2477   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2478   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2479   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2480   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2481   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2482   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2483   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2484   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2485   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2486   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2487   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2488   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2489   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2490   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2491   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2492   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2493   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2494   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2495   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2496   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2497   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2498   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2499   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2500   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2501   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2502   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2503   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2504   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2505   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2506   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2507   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2508   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2509   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2510   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2511   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2512   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2513   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2514   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2515   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2516   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2517   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2518   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2519   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2520   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2521   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2422   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2423   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2424   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2425   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2426   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2427   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2428   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2429   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2430   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2431   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2432   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2433   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2434   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2435   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2436   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2437   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2438   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2439   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2440   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2441   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2442   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2443   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2444   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2445   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2446   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2447   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2448   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2449   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2450   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2451   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2452   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2453   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2454   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2455   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2456   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2457   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2458   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2459   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2460   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2461   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2462   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2463   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2464   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2465   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2466   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2467   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2468   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2469   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2470   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2471   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2472   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2366   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2367   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2368   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2369   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2370   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2371   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2372   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2373   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2374   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2375   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2376   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2377   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2378   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2379   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2380   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2381   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2382   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2383   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2384   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2385   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2386   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2387   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2388   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2389   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2390   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2391   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2392   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2393   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2394   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2395   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2396   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2397   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2398   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2399   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2400   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2401   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2402   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2403   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2404   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2405   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2406   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2407   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2408   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2409   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2410   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2411   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2412   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2413   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2414   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2415   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2416   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2417   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2418   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2419   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2420   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2421   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2317   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2318   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2319   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2320   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2321   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2322   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2323   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2324   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2325   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2326   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2327   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2328   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2329   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2330   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2331   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2332   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2333   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2334   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2335   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2336   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2337   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2338   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2339   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2340   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2341   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2342   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2343   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2344   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2345   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2346   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2347   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2348   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2349   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ## 2350   NA     NA    NA   NA     NA      NA   NA  NA      NA    NA    NA    NA
    ##        OPS OPS.   W   L  ERA ERA.  WHIP G...31  GS  SV     IP H...35 HR...36
    ## 1    0.855  122  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2    0.953  133  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3    0.243  -35  47  40 2.31  187 0.998    853   0 422  903.0    601      82
    ## 4    0.823  111  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 5    0.907  140  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 6    0.855  123  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 7    0.930  140  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 8    0.996  154  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 9    0.688   82  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 10   0.346  -10 256 153 3.85  117 1.351    531 521   0 3316.0   3448     288
    ## 11   0.870  128  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 12   0.743   95  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 13   0.199  -47 214 160 3.81  117 1.281    518 493   0 3283.1   3472     361
    ## 14   0.793  110  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 15   0.000 -100  42  34 2.95  141 1.066    668   0 324  680.0    542      70
    ## 16   0.331  -13 148 137 4.28  101 1.301    419 383   1 2435.2   2507     347
    ## 17   0.821  117  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 18   0.283  -24 188 147 3.92  110 1.295    448 446   0 2840.1   2862     319
    ## 19   0.752  102  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 20   0.822  122  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 21   0.760  103  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 22   0.331  -10 104 118 3.68  108 1.228    342 331   0 2085.2   1849     211
    ## 23   0.178  -50 150  98 3.63  111 1.191    331 331   0 2067.1   1912     262
    ## 24   0.816  117  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4312    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4313    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4314    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4315    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4316    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4317    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4318    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4319    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4320    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4321    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4322    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4323    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4324    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4325    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4326    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4327    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4328    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4329    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4330    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4331    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4332    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4333    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4334    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4335    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4336    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4337    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4338    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4339    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4340    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4341    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4342    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4343    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4344    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4345    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4346    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4347    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4287    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4288    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4289    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4290    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4291    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4292    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4293    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4294    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4295    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4296    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4297    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4298    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4299    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4300    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4301    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4302    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4303    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4304    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4305    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4306    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4307    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4308    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4309    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4310    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4311    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4253    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4254    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4255    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4256    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4257    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4258    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4259    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4260    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4261    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4262    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4263    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4264    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4265    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4266    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4267    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4268    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4269    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4270    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4271    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4272    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4273    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4274    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4275    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4276    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4277    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4278    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4279    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4280    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4281    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4282    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4283    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4284    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4285    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4286    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4216    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4217    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4218    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4219    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4220    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4221    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4222    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4223    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4224    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4225    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4226    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4227    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4228    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4229    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4230    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4231    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4232    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4233    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4234    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4235    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4236    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4237    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4238    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4239    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4240    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4241    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4242    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4243    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4244    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4245    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4246    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4247    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4248    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4249    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4250    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4251    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4252    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4181    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4182    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4183    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4184    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4185    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4186    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4187    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4188    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4189    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4190    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4191    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4192    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4193    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4194    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4195    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4196    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4197    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4198    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4199    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4200    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4201    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4202    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4203    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4204    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4205    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4206    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4207    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4208    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4209    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4210    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4211    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4212    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4213    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4214    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4215    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4145    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4146    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4147    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4148    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4149    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4150    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4151    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4152    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4153    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4154    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4155    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4156    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4157    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4158    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4159    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4160    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4161    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4162    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4163    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4164    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4165    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4166    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4167    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4168    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4169    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4170    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4171    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4172    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4173    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4174    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4175    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4176    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4177    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4178    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4179    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4180    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4113    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4114    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4115    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4116    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4117    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4118    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4119    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4120    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4121    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4122    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4123    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4124    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4125    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4126    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4127    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4128    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4129    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4130    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4131    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4132    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4133    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4134    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4135    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4136    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4137    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4138    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4139    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4140    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4141    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4142    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4143    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4144    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4079    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4080    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4081    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4082    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4083    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4084    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4085    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4086    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4087    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4088    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4089    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4090    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4091    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4092    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4093    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4094    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4095    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4096    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4097    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4098    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4099    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4100    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4101    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4102    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4103    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4104    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4105    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4106    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4107    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4108    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4109    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4110    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4111    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4112    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4040    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4041    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4042    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4043    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4044    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4045    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4046    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4047    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4048    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4049    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4050    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4051    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4052    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4053    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4054    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4055    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4056    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4057    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4058    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4059    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4060    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4061    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4062    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4063    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4064    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4065    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4066    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4067    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4068    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4069    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4070    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4071    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4072    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4073    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4074    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4075    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4076    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4077    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4078    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4000    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4001    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4002    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4003    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4004    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4005    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4006    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4007    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4008    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4009    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4010    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4011    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4012    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4013    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4014    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4015    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4016    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4017    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4018    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4019    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4020    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4021    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4022    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4023    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4024    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4025    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4026    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4027    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4028    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4029    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4030    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4031    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4032    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4033    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4034    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4035    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4036    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4037    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4038    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 4039    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3972    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3973    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3974    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3975    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3976    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3977    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3978    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3979    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3980    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3981    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3982    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3983    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3984    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3985    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3986    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3987    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3988    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3989    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3990    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3991    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3992    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3993    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3994    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3995    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3996    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3997    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3998    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3999    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3938    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3939    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3940    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3941    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3942    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3943    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3944    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3945    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3946    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3947    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3948    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3949    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3950    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3951    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3952    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3953    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3954    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3955    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3956    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3957    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3958    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3959    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3960    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3961    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3962    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3963    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3964    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3965    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3966    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3967    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3968    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3969    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3970    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3971    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3910    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3911    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3912    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3913    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3914    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3915    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3916    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3917    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3918    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3919    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3920    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3921    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3922    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3923    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3924    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3925    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3926    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3927    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3928    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3929    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3930    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3931    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3932    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3933    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3934    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3935    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3936    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3937    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3886    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3887    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3888    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3889    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3890    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3891    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3892    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3893    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3894    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3895    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3896    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3897    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3898    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3899    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3900    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3901    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3902    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3903    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3904    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3905    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3906    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3907    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3908    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3909    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3856    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3857    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3858    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3859    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3860    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3861    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3862    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3863    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3864    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3865    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3866    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3867    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3868    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3869    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3870    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3871    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3872    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3873    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3874    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3875    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3876    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3877    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3878    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3879    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3880    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3881    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3882    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3883    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3884    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3885    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3824    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3825    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3826    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3827    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3828    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3829    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3830    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3831    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3832    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3833    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3834    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3835    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3836    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3837    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3838    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3839    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3840    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3841    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3842    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3843    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3844    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3845    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3846    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3847    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3848    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3849    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3850    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3851    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3852    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3853    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3854    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3855    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3778    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3779    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3780    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3781    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3782    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3783    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3784    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3785    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3786    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3787    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3788    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3789    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3790    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3791    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3792    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3793    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3794    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3795    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3796    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3797    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3798    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3799    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3800    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3801    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3802    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3803    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3804    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3805    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3806    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3807    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3808    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3809    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3810    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3811    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3812    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3813    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3814    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3815    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3816    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3817    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3818    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3819    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3820    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3821    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3822    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3823    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3751    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3752    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3753    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3754    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3755    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3756    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3757    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3758    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3759    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3760    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3761    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3762    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3763    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3764    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3765    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3766    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3767    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3768    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3769    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3770    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3771    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3772    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3773    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3774    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3775    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3776    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3777    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3719    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3720    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3721    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3722    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3723    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3724    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3725    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3726    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3727    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3728    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3729    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3730    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3731    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3732    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3733    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3734    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3735    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3736    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3737    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3738    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3739    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3740    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3741    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3742    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3743    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3744    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3745    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3746    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3747    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3748    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3749    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3750    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3686    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3687    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3688    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3689    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3690    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3691    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3692    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3693    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3694    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3695    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3696    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3697    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3698    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3699    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3700    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3701    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3702    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3703    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3704    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3705    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3706    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3707    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3708    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3709    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3710    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3711    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3712    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3713    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3714    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3715    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3716    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3717    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3718    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3658    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3659    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3660    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3661    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3662    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3663    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3664    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3665    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3666    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3667    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3668    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3669    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3670    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3671    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3672    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3673    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3674    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3675    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3676    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3677    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3678    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3679    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3680    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3681    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3682    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3683    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3684    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3685    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3624    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3625    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3626    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3627    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3628    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3629    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3630    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3631    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3632    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3633    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3634    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3635    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3636    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3637    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3638    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3639    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3640    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3641    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3642    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3643    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3644    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3645    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3646    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3647    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3648    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3649    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3650    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3651    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3652    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3653    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3654    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3655    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3656    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3657    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3591    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3592    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3593    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3594    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3595    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3596    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3597    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3598    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3599    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3600    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3601    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3602    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3603    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3604    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3605    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3606    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3607    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3608    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3609    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3610    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3611    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3612    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3613    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3614    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3615    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3616    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3617    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3618    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3619    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3620    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3621    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3622    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3623    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3559    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3560    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3561    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3562    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3563    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3564    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3565    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3566    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3567    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3568    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3569    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3570    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3571    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3572    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3573    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3574    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3575    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3576    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3577    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3578    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3579    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3580    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3581    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3582    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3583    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3584    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3585    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3586    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3587    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3588    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3589    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3590    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3529    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3530    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3531    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3532    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3533    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3534    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3535    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3536    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3537    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3538    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3539    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3540    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3541    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3542    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3543    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3544    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3545    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3546    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3547    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3548    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3549    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3550    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3551    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3552    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3553    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3554    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3555    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3556    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3557    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3558    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3496    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3497    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3498    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3499    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3500    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3501    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3502    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3503    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3504    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3505    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3506    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3507    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3508    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3509    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3510    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3511    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3512    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3513    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3514    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3515    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3516    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3517    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3518    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3519    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3520    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3521    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3522    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3523    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3524    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3525    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3526    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3527    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3528    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3457    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3458    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3459    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3460    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3461    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3462    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3463    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3464    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3465    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3466    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3467    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3468    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3469    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3470    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3471    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3472    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3473    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3474    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3475    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3476    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3477    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3478    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3479    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3480    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3481    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3482    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3483    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3484    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3485    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3486    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3487    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3488    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3489    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3490    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3491    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3492    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3493    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3494    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3495    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3414    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3415    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3416    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3417    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3418    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3419    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3420    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3421    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3422    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3423    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3424    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3425    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3426    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3427    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3428    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3429    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3430    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3431    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3432    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3433    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3434    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3435    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3436    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3437    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3438    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3439    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3440    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3441    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3442    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3443    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3444    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3445    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3446    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3447    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3448    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3449    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3450    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3451    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3452    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3453    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3454    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3455    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3456    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3373    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3374    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3375    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3376    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3377    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3378    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3379    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3380    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3381    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3382    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3383    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3384    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3385    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3386    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3387    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3388    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3389    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3390    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3391    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3392    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3393    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3394    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3395    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3396    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3397    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3398    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3399    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3400    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3401    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3402    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3403    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3404    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3405    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3406    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3407    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3408    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3409    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3410    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3411    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3412    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3413    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3340    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3341    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3342    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3343    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3344    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3345    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3346    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3347    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3348    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3349    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3350    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3351    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3352    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3353    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3354    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3355    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3356    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3357    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3358    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3359    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3360    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3361    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3362    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3363    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3364    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3365    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3366    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3367    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3368    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3369    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3370    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3371    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3372    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3301    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3302    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3303    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3304    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3305    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3306    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3307    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3308    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3309    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3310    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3311    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3312    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3313    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3314    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3315    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3316    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3317    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3318    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3319    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3320    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3321    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3322    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3323    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3324    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3325    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3326    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3327    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3328    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3329    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3330    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3331    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3332    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3333    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3334    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3335    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3336    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3337    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3338    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3339    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3254    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3255    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3256    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3257    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3258    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3259    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3260    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3261    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3262    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3263    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3264    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3265    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3266    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3267    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3268    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3269    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3270    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3271    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3272    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3273    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3274    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3275    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3276    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3277    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3278    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3279    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3280    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3281    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3282    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3283    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3284    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3285    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3286    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3287    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3288    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3289    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3290    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3291    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3292    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3293    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3294    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3295    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3296    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3297    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3298    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3299    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3300    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3210    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3211    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3212    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3213    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3214    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3215    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3216    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3217    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3218    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3219    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3220    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3221    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3222    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3223    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3224    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3225    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3226    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3227    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3228    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3229    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3230    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3231    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3232    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3233    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3234    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3235    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3236    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3237    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3238    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3239    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3240    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3241    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3242    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3243    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3244    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3245    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3246    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3247    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3248    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3249    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3250    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3251    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3252    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3253    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3167    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3168    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3169    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3170    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3171    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3172    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3173    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3174    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3175    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3176    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3177    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3178    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3179    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3180    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3181    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3182    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3183    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3184    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3185    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3186    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3187    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3188    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3189    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3190    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3191    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3192    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3193    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3194    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3195    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3196    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3197    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3198    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3199    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3200    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3201    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3202    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3203    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3204    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3205    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3206    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3207    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3208    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3209    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3123    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3124    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3125    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3126    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3127    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3128    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3129    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3130    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3131    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3132    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3133    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3134    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3135    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3136    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3137    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3138    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3139    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3140    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3141    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3142    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3143    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3144    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3145    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3146    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3147    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3148    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3149    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3150    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3151    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3152    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3153    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3154    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3155    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3156    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3157    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3158    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3159    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3160    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3161    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3162    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3163    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3164    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3165    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3166    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3094    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3095    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3096    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3097    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3098    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3099    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3100    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3101    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3102    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3103    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3104    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3105    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3106    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3107    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3108    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3109    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3110    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3111    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3112    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3113    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3114    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3115    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3116    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3117    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3118    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3119    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3120    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3121    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3122    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3051    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3052    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3053    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3054    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3055    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3056    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3057    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3058    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3059    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3060    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3061    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3062    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3063    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3064    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3065    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3066    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3067    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3068    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3069    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3070    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3071    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3072    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3073    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3074    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3075    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3076    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3077    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3078    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3079    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3080    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3081    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3082    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3083    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3084    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3085    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3086    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3087    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3088    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3089    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3090    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3091    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3092    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3093    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3008    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3009    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3010    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3011    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3012    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3013    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3014    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3015    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3016    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3017    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3018    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3019    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3020    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3021    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3022    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3023    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3024    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3025    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3026    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3027    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3028    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3029    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3030    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3031    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3032    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3033    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3034    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3035    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3036    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3037    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3038    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3039    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3040    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3041    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3042    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3043    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3044    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3045    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3046    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3047    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3048    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3049    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3050    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2977    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2978    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2979    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2980    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2981    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2982    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2983    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2984    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2985    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2986    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2987    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2988    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2989    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2990    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2991    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2992    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2993    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2994    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2995    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2996    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2997    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2998    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2999    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3000    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3001    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3002    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3003    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3004    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3005    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3006    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 3007    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2929    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2930    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2931    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2932    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2933    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2934    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2935    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2936    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2937    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2938    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2939    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2940    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2941    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2942    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2943    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2944    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2945    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2946    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2947    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2948    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2949    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2950    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2951    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2952    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2953    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2954    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2955    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2956    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2957    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2958    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2959    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2960    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2961    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2962    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2963    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2964    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2965    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2966    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2967    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2968    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2969    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2970    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2971    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2972    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2973    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2974    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2975    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2976    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2885    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2886    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2887    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2888    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2889    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2890    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2891    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2892    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2893    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2894    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2895    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2896    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2897    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2898    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2899    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2900    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2901    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2902    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2903    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2904    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2905    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2906    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2907    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2908    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2909    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2910    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2911    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2912    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2913    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2914    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2915    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2916    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2917    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2918    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2919    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2920    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2921    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2922    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2923    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2924    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2925    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2926    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2927    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2928    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2844    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2845    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2846    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2847    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2848    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2849    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2850    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2851    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2852    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2853    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2854    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2855    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2856    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2857    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2858    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2859    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2860    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2861    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2862    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2863    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2864    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2865    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2866    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2867    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2868    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2869    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2870    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2871    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2872    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2873    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2874    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2875    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2876    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2877    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2878    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2879    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2880    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2881    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2882    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2883    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2884    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2781    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2782    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2783    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2784    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2785    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2786    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2787    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2788    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2789    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2790    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2791    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2792    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2793    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2794    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2795    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2796    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2797    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2798    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2799    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2800    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2801    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2802    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2803    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2804    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2805    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2806    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2807    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2808    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2809    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2810    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2811    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2812    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2813    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2814    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2815    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2816    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2817    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2818    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2819    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2820    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2821    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2822    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2823    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2824    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2825    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2826    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2827    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2828    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2829    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2830    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2831    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2832    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2833    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2834    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2835    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2836    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2837    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2838    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2839    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2840    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2841    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2842    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2843    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2725    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2726    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2727    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2728    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2729    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2730    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2731    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2732    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2733    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2734    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2735    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2736    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2737    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2738    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2739    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2740    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2741    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2742    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2743    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2744    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2745    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2746    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2747    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2748    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2749    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2750    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2751    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2752    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2753    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2754    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2755    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2756    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2757    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2758    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2759    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2760    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2761    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2762    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2763    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2764    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2765    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2766    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2767    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2768    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2769    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2770    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2771    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2772    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2773    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2774    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2775    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2776    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2777    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2778    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2779    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2780    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2686    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2687    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2688    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2689    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2690    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2691    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2692    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2693    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2694    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2695    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2696    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2697    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2698    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2699    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2700    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2701    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2702    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2703    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2704    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2705    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2706    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2707    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2708    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2709    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2710    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2711    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2712    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2713    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2714    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2715    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2716    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2717    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2718    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2719    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2720    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2721    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2722    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2723    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2724    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2647    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2648    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2649    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2650    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2651    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2652    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2653    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2654    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2655    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2656    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2657    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2658    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2659    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2660    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2661    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2662    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2663    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2664    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2665    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2666    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2667    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2668    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2669    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2670    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2671    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2672    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2673    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2674    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2675    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2676    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2677    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2678    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2679    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2680    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2681    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2682    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2683    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2684    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2685    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2611    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2612    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2613    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2614    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2615    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2616    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2617    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2618    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2619    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2620    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2621    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2622    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2623    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2624    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2625    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2626    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2627    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2628    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2629    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2630    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2631    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2632    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2633    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2634    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2635    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2636    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2637    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2638    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2639    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2640    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2641    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2642    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2643    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2644    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2645    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2646    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2570    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2571    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2572    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2573    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2574    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2575    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2576    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2577    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2578    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2579    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2580    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2581    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2582    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2583    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2584    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2585    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2586    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2587    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2588    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2589    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2590    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2591    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2592    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2593    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2594    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2595    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2596    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2597    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2598    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2599    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2600    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2601    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2602    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2603    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2604    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2605    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2606    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2607    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2608    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2609    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2610    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2522    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2523    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2524    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2525    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2526    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2527    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2528    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2529    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2530    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2531    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2532    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2533    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2534    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2535    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2536    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2537    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2538    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2539    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2540    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2541    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2542    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2543    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2544    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2545    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2546    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2547    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2548    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2549    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2550    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2551    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2552    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2553    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2554    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2555    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2556    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2557    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2558    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2559    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2560    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2561    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2562    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2563    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2564    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2565    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2566    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2567    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2568    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2569    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2473    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2474    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2475    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2476    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2477    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2478    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2479    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2480    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2481    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2482    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2483    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2484    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2485    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2486    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2487    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2488    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2489    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2490    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2491    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2492    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2493    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2494    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2495    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2496    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2497    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2498    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2499    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2500    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2501    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2502    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2503    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2504    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2505    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2506    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2507    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2508    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2509    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2510    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2511    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2512    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2513    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2514    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2515    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2516    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2517    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2518    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2519    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2520    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2521    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2422    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2423    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2424    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2425    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2426    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2427    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2428    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2429    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2430    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2431    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2432    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2433    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2434    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2435    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2436    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2437    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2438    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2439    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2440    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2441    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2442    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2443    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2444    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2445    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2446    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2447    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2448    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2449    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2450    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2451    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2452    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2453    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2454    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2455    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2456    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2457    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2458    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2459    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2460    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2461    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2462    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2463    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2464    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2465    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2466    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2467    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2468    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2469    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2470    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2471    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2472    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2366    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2367    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2368    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2369    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2370    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2371    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2372    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2373    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2374    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2375    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2376    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2377    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2378    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2379    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2380    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2381    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2382    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2383    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2384    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2385    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2386    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2387    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2388    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2389    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2390    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2391    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2392    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2393    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2394    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2395    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2396    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2397    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2398    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2399    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2400    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2401    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2402    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2403    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2404    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2405    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2406    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2407    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2408    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2409    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2410    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2411    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2412    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2413    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2414    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2415    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2416    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2417    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2418    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2419    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2420    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2421    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2317    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2318    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2319    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2320    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2321    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2322    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2323    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2324    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2325    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2326    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2327    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2328    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2329    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2330    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2331    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2332    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2333    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2334    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2335    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2336    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2337    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2338    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2339    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2340    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2341    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2342    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2343    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2344    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2345    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2346    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2347    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2348    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2349    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ## 2350    NA   NA  NA  NA   NA   NA    NA     NA  NA  NA     NA     NA      NA
    ##      BB...37   SO  Pos.Summary yearID  playerID          votedBy ballots needed
    ## 1         NA   NA         *5/H   2023 rolensc01             <NA>      NA     NA
    ## 2         NA   NA      *3H/7D9   2023 heltoto01             <NA>      NA     NA
    ## 3        300 1196           *1   2023 wagnebi02             <NA>      NA     NA
    ## 4         NA   NA     *89H7D/3   2023 jonesan01             <NA>      NA     NA
    ## 5         NA   NA *9*7*5*D6H/3   2023 sheffga01             <NA>      NA     NA
    ## 6         NA   NA     *453H/D6   2023  kentje01             <NA>      NA     NA
    ## 7         NA   NA     *6*5DH/3   2023 rodrial01             <NA>      NA     NA
    ## 8         NA   NA      *7*9*DH   2023 ramirma02             <NA>      NA     NA
    ## 9         NA   NA   *65H4/D379   2023 vizquom01             <NA>      NA     NA
    ## 10      1031 2448         *1/H   2023 pettian01             <NA>      NA     NA
    ## 11        NA   NA      *9DH7/8   2023 abreubo01             <NA>      NA     NA
    ## 12        NA   NA       *6H/D4   2023 rolliji01             <NA>      NA     NA
    ## 13       734 1870           *1   2023 buehrma01             <NA>      NA     NA
    ## 14        NA   NA     *8*9DH/7   2023 hunteto01             <NA>      NA     NA
    ## 15       183  665           *1   2023 streehu01             <NA>      NA     NA
    ## 16       661 1571         *1/H   2023 arroybr01             <NA>      NA     NA
    ## 17        NA   NA     *3*2DH/7   2023 napolmi01             <NA>      NA     NA
    ## 18       815 2294         *1/H   2023 lackejo01             <NA>      NA     NA
    ## 19        NA   NA     *65H/D37   2023 peraljh01             <NA>      NA     NA
    ## 20        NA   NA     *97H8/D3   2023 ethiean01             <NA>      NA     NA
    ## 21        NA   NA      *87H9/D   2023 ellsbja01             <NA>      NA     NA
    ## 22       712 1694         *1/H   2023  cainma01             <NA>      NA     NA
    ## 23       551 1621           *1   2023 weaveje02             <NA>      NA     NA
    ## 24        NA   NA    *9*78H/D3   2023 werthja01             <NA>      NA     NA
    ## 4312      NA   NA         <NA>   2022 ortizda01            BBWAA     394    296
    ## 4313      NA   NA         <NA>   2022 bondsba01            BBWAA     394    296
    ## 4314      NA   NA         <NA>   2022 clemero02            BBWAA     394    296
    ## 4315      NA   NA         <NA>   2022 rolensc01            BBWAA     394    296
    ## 4316      NA   NA         <NA>   2022 schilcu01            BBWAA     394    296
    ## 4317      NA   NA         <NA>   2022 heltoto01            BBWAA     394    296
    ## 4318      NA   NA         <NA>   2022 wagnebi02            BBWAA     394    296
    ## 4319      NA   NA         <NA>   2022 jonesan01            BBWAA     394    296
    ## 4320      NA   NA         <NA>   2022 sheffga01            BBWAA     394    296
    ## 4321      NA   NA         <NA>   2022 rodrial01            BBWAA     394    296
    ## 4322      NA   NA         <NA>   2022  kentje01            BBWAA     394    296
    ## 4323      NA   NA         <NA>   2022 ramirma02            BBWAA     394    296
    ## 4324      NA   NA         <NA>   2022 vizquom01            BBWAA     394    296
    ## 4325      NA   NA         <NA>   2022  sosasa01            BBWAA     394    296
    ## 4326      NA   NA         <NA>   2022 pettian01            BBWAA     394    296
    ## 4327      NA   NA         <NA>   2022 rolliji01            BBWAA     394    296
    ## 4328      NA   NA         <NA>   2022 abreubo01            BBWAA     394    296
    ## 4329      NA   NA         <NA>   2022 buehrma01            BBWAA     394    296
    ## 4330      NA   NA         <NA>   2022 hunteto01            BBWAA     394    296
    ## 4331      NA   NA         <NA>   2022 nathajo01            BBWAA     394    296
    ## 4332      NA   NA         <NA>   2022 hudsoti01            BBWAA     394    296
    ## 4333      NA   NA         <NA>   2022 linceti01            BBWAA     394    296
    ## 4334      NA   NA         <NA>   2022 howarry01            BBWAA     394    296
    ## 4335      NA   NA         <NA>   2022 teixema01            BBWAA     394    296
    ## 4336      NA   NA         <NA>   2022 papeljo01            BBWAA     394    296
    ## 4337      NA   NA         <NA>   2022 morneju01            BBWAA     394    296
    ## 4338      NA   NA         <NA>   2022 pierzaj01            BBWAA     394    296
    ## 4339      NA   NA         <NA>   2022 fieldpr01            BBWAA     394    296
    ## 4340      NA   NA         <NA>   2022 peavyja01            BBWAA     394    296
    ## 4341      NA   NA         <NA>   2022 crawfca02            BBWAA     394    296
    ## 4342      NA   NA         <NA>   2022 fowlebu99         Veterans      NA     NA
    ## 4343      NA   NA         <NA>   2022 hodgegi01         Veterans      NA     NA
    ## 4344      NA   NA         <NA>   2022  kaatji01         Veterans      NA     NA
    ## 4345      NA   NA         <NA>   2022 minosmi01         Veterans      NA     NA
    ## 4346      NA   NA         <NA>   2022 oneilbu01         Veterans      NA     NA
    ## 4347      NA   NA         <NA>   2022 olivato01         Veterans      NA     NA
    ## 4287      NA   NA         <NA>   2021 schilcu01            BBWAA     401    301
    ## 4288      NA   NA         <NA>   2021 bondsba01            BBWAA     401    301
    ## 4289      NA   NA         <NA>   2021 clemero02            BBWAA     401    301
    ## 4290      NA   NA         <NA>   2021 rolensc01            BBWAA     401    301
    ## 4291      NA   NA         <NA>   2021 vizquom01            BBWAA     401    301
    ## 4292      NA   NA         <NA>   2021 wagnebi02            BBWAA     401    301
    ## 4293      NA   NA         <NA>   2021 heltoto01            BBWAA     401    301
    ## 4294      NA   NA         <NA>   2021 sheffga01            BBWAA     401    301
    ## 4295      NA   NA         <NA>   2021 jonesan01            BBWAA     401    301
    ## 4296      NA   NA         <NA>   2021  kentje01            BBWAA     401    301
    ## 4297      NA   NA         <NA>   2021 ramirma02            BBWAA     401    301
    ## 4298      NA   NA         <NA>   2021  sosasa01            BBWAA     401    301
    ## 4299      NA   NA         <NA>   2021 pettian01            BBWAA     401    301
    ## 4300      NA   NA         <NA>   2021 buehrma01            BBWAA     401    301
    ## 4301      NA   NA         <NA>   2021 hunteto01            BBWAA     401    301
    ## 4302      NA   NA         <NA>   2021 abreubo01            BBWAA     401    301
    ## 4303      NA   NA         <NA>   2021 hudsoti01            BBWAA     401    301
    ## 4304      NA   NA         <NA>   2021 ramirar01            BBWAA     401    301
    ## 4305      NA   NA         <NA>   2021 hawkila01            BBWAA     401    301
    ## 4306      NA   NA         <NA>   2021  zitoba01            BBWAA     401    301
    ## 4307      NA   NA         <NA>   2021 harenda01            BBWAA     401    301
    ## 4308      NA   NA         <NA>   2021 cuddymi01            BBWAA     401    301
    ## 4309      NA   NA         <NA>   2021 swishni01            BBWAA     401    301
    ## 4310      NA   NA         <NA>   2021 victosh01            BBWAA     401    301
    ## 4311      NA   NA         <NA>   2021 burneaj01            BBWAA     401    301
    ## 4253      NA   NA         <NA>   2020 jeterde01            BBWAA     397    298
    ## 4254      NA   NA         <NA>   2020 walkela01            BBWAA     397    298
    ## 4255      NA   NA         <NA>   2020 schilcu01            BBWAA     397    298
    ## 4256      NA   NA         <NA>   2020 clemero02            BBWAA     397    298
    ## 4257      NA   NA         <NA>   2020 bondsba01            BBWAA     397    298
    ## 4258      NA   NA         <NA>   2020 vizquom01            BBWAA     397    298
    ## 4259      NA   NA         <NA>   2020 rolensc01            BBWAA     397    298
    ## 4260      NA   NA         <NA>   2020 wagnebi02            BBWAA     397    298
    ## 4261      NA   NA         <NA>   2020 sheffga01            BBWAA     397    298
    ## 4262      NA   NA         <NA>   2020 heltoto01            BBWAA     397    298
    ## 4263      NA   NA         <NA>   2020 ramirma02            BBWAA     397    298
    ## 4264      NA   NA         <NA>   2020  kentje01            BBWAA     397    298
    ## 4265      NA   NA         <NA>   2020 jonesan01            BBWAA     397    298
    ## 4266      NA   NA         <NA>   2020  sosasa01            BBWAA     397    298
    ## 4267      NA   NA         <NA>   2020 pettian01            BBWAA     397    298
    ## 4268      NA   NA         <NA>   2020 abreubo01            BBWAA     397    298
    ## 4269      NA   NA         <NA>   2020 konerpa01            BBWAA     397    298
    ## 4270      NA   NA         <NA>   2020 giambja01            BBWAA     397    298
    ## 4271      NA   NA         <NA>   2020 soriaal01            BBWAA     397    298
    ## 4272      NA   NA         <NA>   2020   leecl02            BBWAA     397    298
    ## 4273      NA   NA         <NA>   2020 chaveer01            BBWAA     397    298
    ## 4274      NA   NA         <NA>   2020  putzjj01            BBWAA     397    298
    ## 4275      NA   NA         <NA>   2020 pennybr01            BBWAA     397    298
    ## 4276      NA   NA         <NA>   2020 ibanera01            BBWAA     397    298
    ## 4277      NA   NA         <NA>   2020  dunnad01            BBWAA     397    298
    ## 4278      NA   NA         <NA>   2020 roberbr01            BBWAA     397    298
    ## 4279      NA   NA         <NA>   2020  penaca01            BBWAA     397    298
    ## 4280      NA   NA         <NA>   2020 furcara01            BBWAA     397    298
    ## 4281      NA   NA         <NA>   2020 figgich01            BBWAA     397    298
    ## 4282      NA   NA         <NA>   2020 valvejo01            BBWAA     397    298
    ## 4283      NA   NA         <NA>   2020  bellhe01            BBWAA     397    298
    ## 4284      NA   NA         <NA>   2020 beckejo02            BBWAA     397    298
    ## 4285      NA   NA         <NA>   2020 millema99         Veterans      NA     NA
    ## 4286      NA   NA         <NA>   2020 simmote01         Veterans      NA     NA
    ## 4216      NA   NA         <NA>   2019 riverma01            BBWAA     425    319
    ## 4217      NA   NA         <NA>   2019 hallaro01            BBWAA     425    319
    ## 4218      NA   NA         <NA>   2019 martied01            BBWAA     425    319
    ## 4219      NA   NA         <NA>   2019 mussimi01            BBWAA     425    319
    ## 4220      NA   NA         <NA>   2019 schilcu01            BBWAA     425    319
    ## 4221      NA   NA         <NA>   2019 clemero02            BBWAA     425    319
    ## 4222      NA   NA         <NA>   2019 bondsba01            BBWAA     425    319
    ## 4223      NA   NA         <NA>   2019 walkela01            BBWAA     425    319
    ## 4224      NA   NA         <NA>   2019 vizquom01            BBWAA     425    319
    ## 4225      NA   NA         <NA>   2019 mcgrifr01            BBWAA     425    319
    ## 4226      NA   NA         <NA>   2019 ramirma02            BBWAA     425    319
    ## 4227      NA   NA         <NA>   2019  kentje01            BBWAA     425    319
    ## 4228      NA   NA         <NA>   2019 rolensc01            BBWAA     425    319
    ## 4229      NA   NA         <NA>   2019 wagnebi02            BBWAA     425    319
    ## 4230      NA   NA         <NA>   2019 heltoto01            BBWAA     425    319
    ## 4231      NA   NA         <NA>   2019 sheffga01            BBWAA     425    319
    ## 4232      NA   NA         <NA>   2019 pettian01            BBWAA     425    319
    ## 4233      NA   NA         <NA>   2019  sosasa01            BBWAA     425    319
    ## 4234      NA   NA         <NA>   2019 jonesan01            BBWAA     425    319
    ## 4235      NA   NA         <NA>   2019 youngmi02            BBWAA     425    319
    ## 4236      NA   NA         <NA>   2019 tejadmi01            BBWAA     425    319
    ## 4237      NA   NA         <NA>   2019 berkmla01            BBWAA     425    319
    ## 4238      NA   NA         <NA>   2019 oswalro01            BBWAA     425    319
    ## 4239      NA   NA         <NA>   2019 polanpl01            BBWAA     425    319
    ## 4240      NA   NA         <NA>   2019 wellsve01            BBWAA     425    319
    ## 4241      NA   NA         <NA>   2019 youklke01            BBWAA     425    319
    ## 4242      NA   NA         <NA>   2019 oliveda02            BBWAA     425    319
    ## 4243      NA   NA         <NA>   2019 pierrju01            BBWAA     425    319
    ## 4244      NA   NA         <NA>   2019 ankieri01            BBWAA     425    319
    ## 4245      NA   NA         <NA>   2019  lowede01            BBWAA     425    319
    ## 4246      NA   NA         <NA>   2019 lillyte01            BBWAA     425    319
    ## 4247      NA   NA         <NA>   2019 hafnetr01            BBWAA     425    319
    ## 4248      NA   NA         <NA>   2019 garlajo01            BBWAA     425    319
    ## 4249      NA   NA         <NA>   2019 garcifr02            BBWAA     425    319
    ## 4250      NA   NA         <NA>   2019   bayja01            BBWAA     425    319
    ## 4251      NA   NA         <NA>   2019 baineha01         Veterans      NA     NA
    ## 4252      NA   NA         <NA>   2019 smithle02         Veterans      NA     NA
    ## 4181      NA   NA         <NA>   2018 jonesch06            BBWAA     422    317
    ## 4182      NA   NA         <NA>   2018 guerrvl01            BBWAA     422    317
    ## 4183      NA   NA         <NA>   2018 thomeji01            BBWAA     422    317
    ## 4184      NA   NA         <NA>   2018 hoffmtr01            BBWAA     422    317
    ## 4185      NA   NA         <NA>   2018 martied01            BBWAA     422    317
    ## 4186      NA   NA         <NA>   2018 mussimi01            BBWAA     422    317
    ## 4187      NA   NA         <NA>   2018 clemero02            BBWAA     422    317
    ## 4188      NA   NA         <NA>   2018 bondsba01            BBWAA     422    317
    ## 4189      NA   NA         <NA>   2018 schilcu01            BBWAA     422    317
    ## 4190      NA   NA         <NA>   2018 vizquom01            BBWAA     422    317
    ## 4191      NA   NA         <NA>   2018 walkela01            BBWAA     422    317
    ## 4192      NA   NA         <NA>   2018 mcgrifr01            BBWAA     422    317
    ## 4193      NA   NA         <NA>   2018 ramirma02            BBWAA     422    317
    ## 4194      NA   NA         <NA>   2018  kentje01            BBWAA     422    317
    ## 4195      NA   NA         <NA>   2018 sheffga01            BBWAA     422    317
    ## 4196      NA   NA         <NA>   2018 wagnebi02            BBWAA     422    317
    ## 4197      NA   NA         <NA>   2018 rolensc01            BBWAA     422    317
    ## 4198      NA   NA         <NA>   2018  sosasa01            BBWAA     422    317
    ## 4199      NA   NA         <NA>   2018 jonesan01            BBWAA     422    317
    ## 4200      NA   NA         <NA>   2018 moyerja01            BBWAA     422    317
    ## 4201      NA   NA         <NA>   2018 santajo01            BBWAA     422    317
    ## 4202      NA   NA         <NA>   2018 damonjo01            BBWAA     422    317
    ## 4203      NA   NA         <NA>   2018 matsuhi01            BBWAA     422    317
    ## 4204      NA   NA         <NA>   2018 carpech01            BBWAA     422    317
    ## 4205      NA   NA         <NA>   2018  woodke02            BBWAA     422    317
    ## 4206      NA   NA         <NA>   2018 hernali01            BBWAA     422    317
    ## 4207      NA   NA         <NA>   2018   leeca01            BBWAA     422    317
    ## 4208      NA   NA         <NA>   2018 hudsoor01            BBWAA     422    317
    ## 4209      NA   NA         <NA>   2018  huffau01            BBWAA     422    317
    ## 4210      NA   NA         <NA>   2018 isrinja01            BBWAA     422    317
    ## 4211      NA   NA         <NA>   2018 lidgebr01            BBWAA     422    317
    ## 4212      NA   NA         <NA>   2018 millwke01            BBWAA     422    317
    ## 4213      NA   NA         <NA>   2018 zambrca01            BBWAA     422    317
    ## 4214      NA   NA         <NA>   2018 morrija02         Veterans      NA     NA
    ## 4215      NA   NA         <NA>   2018 trammal01         Veterans      NA     NA
    ## 4145      NA   NA         <NA>   2017 bagweje01            BBWAA     442    332
    ## 4146      NA   NA         <NA>   2017 raineti01            BBWAA     442    332
    ## 4147      NA   NA         <NA>   2017 rodriiv01            BBWAA     442    332
    ## 4148      NA   NA         <NA>   2017 hoffmtr01            BBWAA     442    332
    ## 4149      NA   NA         <NA>   2017 guerrvl01            BBWAA     442    332
    ## 4150      NA   NA         <NA>   2017 martied01            BBWAA     442    332
    ## 4151      NA   NA         <NA>   2017 clemero02            BBWAA     442    332
    ## 4152      NA   NA         <NA>   2017 bondsba01            BBWAA     442    332
    ## 4153      NA   NA         <NA>   2017 mussimi01            BBWAA     442    332
    ## 4154      NA   NA         <NA>   2017 schilcu01            BBWAA     442    332
    ## 4155      NA   NA         <NA>   2017 smithle02            BBWAA     442    332
    ## 4156      NA   NA         <NA>   2017 ramirma02            BBWAA     442    332
    ## 4157      NA   NA         <NA>   2017 walkela01            BBWAA     442    332
    ## 4158      NA   NA         <NA>   2017 mcgrifr01            BBWAA     442    332
    ## 4159      NA   NA         <NA>   2017  kentje01            BBWAA     442    332
    ## 4160      NA   NA         <NA>   2017 sheffga01            BBWAA     442    332
    ## 4161      NA   NA         <NA>   2017 wagnebi02            BBWAA     442    332
    ## 4162      NA   NA         <NA>   2017  sosasa01            BBWAA     442    332
    ## 4163      NA   NA         <NA>   2017 posadjo01            BBWAA     442    332
    ## 4164      NA   NA         <NA>   2017 ordonma01            BBWAA     442    332
    ## 4165      NA   NA         <NA>   2017 renteed01            BBWAA     442    332
    ## 4166      NA   NA         <NA>   2017 varitja01            BBWAA     442    332
    ## 4167      NA   NA         <NA>   2017 wakefti01            BBWAA     442    332
    ## 4168      NA   NA         <NA>   2017 blakeca01            BBWAA     442    332
    ## 4169      NA   NA         <NA>   2017 burrepa01            BBWAA     442    332
    ## 4170      NA   NA         <NA>   2017 cabreor01            BBWAA     442    332
    ## 4171      NA   NA         <NA>   2017 camermi01            BBWAA     442    332
    ## 4172      NA   NA         <NA>   2017  drewjd01            BBWAA     442    332
    ## 4173      NA   NA         <NA>   2017 guillca01            BBWAA     442    332
    ## 4174      NA   NA         <NA>   2017   leede02            BBWAA     442    332
    ## 4175      NA   NA         <NA>   2017  morame01            BBWAA     442    332
    ## 4176      NA   NA         <NA>   2017 rhodear01            BBWAA     442    332
    ## 4177      NA   NA         <NA>   2017 sanchfr01            BBWAA     442    332
    ## 4178      NA   NA         <NA>   2017 stairma01            BBWAA     442    332
    ## 4179      NA   NA         <NA>   2017 seligbu99         Veterans      NA     NA
    ## 4180      NA   NA         <NA>   2017 schurjo99         Veterans      NA     NA
    ## 4113      NA   NA         <NA>   2016 griffke02            BBWAA     440    330
    ## 4114      NA   NA         <NA>   2016 piazzmi01            BBWAA     440    330
    ## 4115      NA   NA         <NA>   2016 bagweje01            BBWAA     440    330
    ## 4116      NA   NA         <NA>   2016 raineti01            BBWAA     440    330
    ## 4117      NA   NA         <NA>   2016 hoffmtr01            BBWAA     440    330
    ## 4118      NA   NA         <NA>   2016 schilcu01            BBWAA     440    330
    ## 4119      NA   NA         <NA>   2016 clemero02            BBWAA     440    330
    ## 4120      NA   NA         <NA>   2016 bondsba01            BBWAA     440    330
    ## 4121      NA   NA         <NA>   2016 martied01            BBWAA     440    330
    ## 4122      NA   NA         <NA>   2016 mussimi01            BBWAA     440    330
    ## 4123      NA   NA         <NA>   2016 trammal01            BBWAA     440    330
    ## 4124      NA   NA         <NA>   2016 smithle02            BBWAA     440    330
    ## 4125      NA   NA         <NA>   2016 mcgrifr01            BBWAA     440    330
    ## 4126      NA   NA         <NA>   2016  kentje01            BBWAA     440    330
    ## 4127      NA   NA         <NA>   2016 walkela01            BBWAA     440    330
    ## 4128      NA   NA         <NA>   2016 mcgwima01            BBWAA     440    330
    ## 4129      NA   NA         <NA>   2016 sheffga01            BBWAA     440    330
    ## 4130      NA   NA         <NA>   2016 wagnebi02            BBWAA     440    330
    ## 4131      NA   NA         <NA>   2016  sosasa01            BBWAA     440    330
    ## 4132      NA   NA         <NA>   2016 edmonji01            BBWAA     440    330
    ## 4133      NA   NA         <NA>   2016 garcino01            BBWAA     440    330
    ## 4134      NA   NA         <NA>   2016 sweenmi01            BBWAA     440    330
    ## 4135      NA   NA         <NA>   2016 kendaja01            BBWAA     440    330
    ## 4136      NA   NA         <NA>   2016 eckstda01            BBWAA     440    330
    ## 4137      NA   NA         <NA>   2016 anderga01            BBWAA     440    330
    ## 4138      NA   NA         <NA>   2016 glaustr01            BBWAA     440    330
    ## 4139      NA   NA         <NA>   2016  winnra01            BBWAA     440    330
    ## 4140      NA   NA         <NA>   2016 grudzma01            BBWAA     440    330
    ## 4141      NA   NA         <NA>   2016 ausmubr01            BBWAA     440    330
    ## 4142      NA   NA         <NA>   2016 hamptmi01            BBWAA     440    330
    ## 4143      NA   NA         <NA>   2016 castilu01            BBWAA     440    330
    ## 4144      NA   NA         <NA>   2016 lowelmi01            BBWAA     440    330
    ## 4079      NA   NA         <NA>   2015 johnsra05            BBWAA     549    412
    ## 4080      NA   NA         <NA>   2015 martipe02            BBWAA     549    412
    ## 4081      NA   NA         <NA>   2015 smoltjo01            BBWAA     549    412
    ## 4082      NA   NA         <NA>   2015 biggicr01            BBWAA     549    412
    ## 4083      NA   NA         <NA>   2015 piazzmi01            BBWAA     549    412
    ## 4084      NA   NA         <NA>   2015 bagweje01            BBWAA     549    412
    ## 4085      NA   NA         <NA>   2015 raineti01            BBWAA     549    412
    ## 4086      NA   NA         <NA>   2015 schilcu01            BBWAA     549    412
    ## 4087      NA   NA         <NA>   2015 clemero02            BBWAA     549    412
    ## 4088      NA   NA         <NA>   2015 bondsba01            BBWAA     549    412
    ## 4089      NA   NA         <NA>   2015 smithle02            BBWAA     549    412
    ## 4090      NA   NA         <NA>   2015 martied01            BBWAA     549    412
    ## 4091      NA   NA         <NA>   2015 trammal01            BBWAA     549    412
    ## 4092      NA   NA         <NA>   2015 mussimi01            BBWAA     549    412
    ## 4093      NA   NA         <NA>   2015  kentje01            BBWAA     549    412
    ## 4094      NA   NA         <NA>   2015 mcgrifr01            BBWAA     549    412
    ## 4095      NA   NA         <NA>   2015 walkela01            BBWAA     549    412
    ## 4096      NA   NA         <NA>   2015 sheffga01            BBWAA     549    412
    ## 4097      NA   NA         <NA>   2015 mcgwima01            BBWAA     549    412
    ## 4098      NA   NA         <NA>   2015 mattido01            BBWAA     549    412
    ## 4099      NA   NA         <NA>   2015  sosasa01            BBWAA     549    412
    ## 4100      NA   NA         <NA>   2015 garcino01            BBWAA     549    412
    ## 4101      NA   NA         <NA>   2015 delgaca01            BBWAA     549    412
    ## 4102      NA   NA         <NA>   2015 percitr01            BBWAA     549    412
    ## 4103      NA   NA         <NA>   2015 gordoto01            BBWAA     549    412
    ## 4104      NA   NA         <NA>   2015 booneaa01            BBWAA     549    412
    ## 4105      NA   NA         <NA>   2015 erstada01            BBWAA     549    412
    ## 4106      NA   NA         <NA>   2015 clarkto02            BBWAA     549    412
    ## 4107      NA   NA         <NA>   2015 gilesbr02            BBWAA     549    412
    ## 4108      NA   NA         <NA>   2015 aurilri01            BBWAA     549    412
    ## 4109      NA   NA         <NA>   2015 guarded01            BBWAA     549    412
    ## 4110      NA   NA         <NA>   2015 schmija01            BBWAA     549    412
    ## 4111      NA   NA         <NA>   2015   dyeje01            BBWAA     549    412
    ## 4112      NA   NA         <NA>   2015 floydcl01            BBWAA     549    412
    ## 4040      NA   NA         <NA>   2014   coxbo01         Veterans      NA     NA
    ## 4041      NA   NA         <NA>   2014 larusto01         Veterans      NA     NA
    ## 4042      NA   NA         <NA>   2014 torrejo01         Veterans      NA     NA
    ## 4043      NA   NA         <NA>   2014 maddugr01            BBWAA     571    429
    ## 4044      NA   NA         <NA>   2014 glavito02            BBWAA     571    429
    ## 4045      NA   NA         <NA>   2014 thomafr04            BBWAA     571    429
    ## 4046      NA   NA         <NA>   2014 biggicr01            BBWAA     571    429
    ## 4047      NA   NA         <NA>   2014 piazzmi01            BBWAA     571    429
    ## 4048      NA   NA         <NA>   2014 morrija02            BBWAA     571    429
    ## 4049      NA   NA         <NA>   2014 bagweje01            BBWAA     571    429
    ## 4050      NA   NA         <NA>   2014 raineti01            BBWAA     571    429
    ## 4051      NA   NA         <NA>   2014 clemero02            BBWAA     571    429
    ## 4052      NA   NA         <NA>   2014 bondsba01            BBWAA     571    429
    ## 4053      NA   NA         <NA>   2014 smithle02            BBWAA     571    429
    ## 4054      NA   NA         <NA>   2014 schilcu01            BBWAA     571    429
    ## 4055      NA   NA         <NA>   2014 martied01            BBWAA     571    429
    ## 4056      NA   NA         <NA>   2014 trammal01            BBWAA     571    429
    ## 4057      NA   NA         <NA>   2014 mussimi01            BBWAA     571    429
    ## 4058      NA   NA         <NA>   2014  kentje01            BBWAA     571    429
    ## 4059      NA   NA         <NA>   2014 mcgrifr01            BBWAA     571    429
    ## 4060      NA   NA         <NA>   2014 mcgwima01            BBWAA     571    429
    ## 4061      NA   NA         <NA>   2014 walkela01            BBWAA     571    429
    ## 4062      NA   NA         <NA>   2014 mattido01            BBWAA     571    429
    ## 4063      NA   NA         <NA>   2014  sosasa01            BBWAA     571    429
    ## 4064      NA   NA         <NA>   2014 palmera01            BBWAA     571    429
    ## 4065      NA   NA         <NA>   2014  aloumo01            BBWAA     571    429
    ## 4066      NA   NA         <NA>   2014  nomohi01            BBWAA     571    429
    ## 4067      NA   NA         <NA>   2014 gonzalu01            BBWAA     571    429
    ## 4068      NA   NA         <NA>   2014 gagneer01            BBWAA     571    429
    ## 4069      NA   NA         <NA>   2014  snowjt01            BBWAA     571    429
    ## 4070      NA   NA         <NA>   2014 benitar01            BBWAA     571    429
    ## 4071      NA   NA         <NA>   2014 rogerke01            BBWAA     571    429
    ## 4072      NA   NA         <NA>   2014 jonesja05            BBWAA     571    429
    ## 4073      NA   NA         <NA>   2014 timlimi01            BBWAA     571    429
    ## 4074      NA   NA         <NA>   2014 loducpa01            BBWAA     571    429
    ## 4075      NA   NA         <NA>   2014 sexsori01            BBWAA     571    429
    ## 4076      NA   NA         <NA>   2014 caseyse01            BBWAA     571    429
    ## 4077      NA   NA         <NA>   2014 jonesto02            BBWAA     571    429
    ## 4078      NA   NA         <NA>   2014 durhara01            BBWAA     571    429
    ## 4000      NA   NA         <NA>   2013 biggicr01            BBWAA     569    427
    ## 4001      NA   NA         <NA>   2013 morrija02            BBWAA     569    427
    ## 4002      NA   NA         <NA>   2013 bagweje01            BBWAA     569    427
    ## 4003      NA   NA         <NA>   2013 piazzmi01            BBWAA     569    427
    ## 4004      NA   NA         <NA>   2013 raineti01            BBWAA     569    427
    ## 4005      NA   NA         <NA>   2013 smithle02            BBWAA     569    427
    ## 4006      NA   NA         <NA>   2013 schilcu01            BBWAA     569    427
    ## 4007      NA   NA         <NA>   2013 clemero02            BBWAA     569    427
    ## 4008      NA   NA         <NA>   2013 bondsba01            BBWAA     569    427
    ## 4009      NA   NA         <NA>   2013 martied01            BBWAA     569    427
    ## 4010      NA   NA         <NA>   2013 trammal01            BBWAA     569    427
    ## 4011      NA   NA         <NA>   2013 walkela01            BBWAA     569    427
    ## 4012      NA   NA         <NA>   2013 mcgrifr01            BBWAA     569    427
    ## 4013      NA   NA         <NA>   2013 murphda05            BBWAA     569    427
    ## 4014      NA   NA         <NA>   2013 mcgwima01            BBWAA     569    427
    ## 4015      NA   NA         <NA>   2013 mattido01            BBWAA     569    427
    ## 4016      NA   NA         <NA>   2013  sosasa01            BBWAA     569    427
    ## 4017      NA   NA         <NA>   2013 palmera01            BBWAA     569    427
    ## 4018      NA   NA         <NA>   2013 willibe02            BBWAA     569    427
    ## 4019      NA   NA         <NA>   2013 loftoke01            BBWAA     569    427
    ## 4020      NA   NA         <NA>   2013 alomasa02            BBWAA     569    427
    ## 4021      NA   NA         <NA>   2013 francju02            BBWAA     569    427
    ## 4022      NA   NA         <NA>   2013 wellsda01            BBWAA     569    427
    ## 4023      NA   NA         <NA>   2013 finlest01            BBWAA     569    427
    ## 4024      NA   NA         <NA>   2013 greensh01            BBWAA     569    427
    ## 4025      NA   NA         <NA>   2013  seleaa01            BBWAA     569    427
    ## 4026      NA   NA         <NA>   2013 cirilje01            BBWAA     569    427
    ## 4027      NA   NA         <NA>   2013 coninje01            BBWAA     569    427
    ## 4028      NA   NA         <NA>   2013  mesajo01            BBWAA     569    427
    ## 4029      NA   NA         <NA>   2013 stantmi01            BBWAA     569    427
    ## 4030      NA   NA         <NA>   2013 sandere02            BBWAA     569    427
    ## 4031      NA   NA         <NA>   2013 hernaro01            BBWAA     569    427
    ## 4032      NA   NA         <NA>   2013 whitero02            BBWAA     569    427
    ## 4033      NA   NA         <NA>   2013 claytro01            BBWAA     569    427
    ## 4034      NA   NA         <NA>   2013 kleskry01            BBWAA     569    427
    ## 4035      NA   NA         <NA>   2013 walketo04            BBWAA     569    427
    ## 4036      NA   NA         <NA>   2013 williwo02            BBWAA     569    427
    ## 4037      NA   NA         <NA>   2013 ruperja99         Veterans      NA     NA
    ## 4038      NA   NA         <NA>   2013  odayha01         Veterans      NA     NA
    ## 4039      NA   NA         <NA>   2013 whitede01         Veterans      NA     NA
    ## 3972      NA   NA         <NA>   2012 larkiba01            BBWAA     573    430
    ## 3973      NA   NA         <NA>   2012 morrija02            BBWAA     573    430
    ## 3974      NA   NA         <NA>   2012 bagweje01            BBWAA     573    430
    ## 3975      NA   NA         <NA>   2012 smithle02            BBWAA     573    430
    ## 3976      NA   NA         <NA>   2012 raineti01            BBWAA     573    430
    ## 3977      NA   NA         <NA>   2012 trammal01            BBWAA     573    430
    ## 3978      NA   NA         <NA>   2012 martied01            BBWAA     573    430
    ## 3979      NA   NA         <NA>   2012 mcgrifr01            BBWAA     573    430
    ## 3980      NA   NA         <NA>   2012 walkela01            BBWAA     573    430
    ## 3981      NA   NA         <NA>   2012 mcgwima01            BBWAA     573    430
    ## 3982      NA   NA         <NA>   2012 mattido01            BBWAA     573    430
    ## 3983      NA   NA         <NA>   2012 murphda05            BBWAA     573    430
    ## 3984      NA   NA         <NA>   2012 palmera01            BBWAA     573    430
    ## 3985      NA   NA         <NA>   2012 willibe02            BBWAA     573    430
    ## 3986      NA   NA         <NA>   2012 gonzaju03            BBWAA     573    430
    ## 3987      NA   NA         <NA>   2012 castivi02            BBWAA     573    430
    ## 3988      NA   NA         <NA>   2012 salmoti01            BBWAA     573    430
    ## 3989      NA   NA         <NA>   2012 muellbi02            BBWAA     573    430
    ## 3990      NA   NA         <NA>   2012 radkebr01            BBWAA     573    430
    ## 3991      NA   NA         <NA>   2012 lopezja01            BBWAA     573    430
    ## 3992      NA   NA         <NA>   2012 younger01            BBWAA     573    430
    ## 3993      NA   NA         <NA>   2012 nevinph01            BBWAA     573    430
    ## 3994      NA   NA         <NA>   2012 jordabr01            BBWAA     573    430
    ## 3995      NA   NA         <NA>   2012 sierrru01            BBWAA     573    430
    ## 3996      NA   NA         <NA>   2012 burnije01            BBWAA     573    430
    ## 3997      NA   NA         <NA>   2012 mulhote01            BBWAA     573    430
    ## 3998      NA   NA         <NA>   2012 womacto01            BBWAA     573    430
    ## 3999      NA   NA         <NA>   2012 santoro01         Veterans      NA     NA
    ## 3938      NA   NA         <NA>   2011 alomaro01            BBWAA     581    436
    ## 3939      NA   NA         <NA>   2011 blylebe01            BBWAA     581    436
    ## 3940      NA   NA         <NA>   2011 larkiba01            BBWAA     581    436
    ## 3941      NA   NA         <NA>   2011 morrija02            BBWAA     581    436
    ## 3942      NA   NA         <NA>   2011 smithle02            BBWAA     581    436
    ## 3943      NA   NA         <NA>   2011 bagweje01            BBWAA     581    436
    ## 3944      NA   NA         <NA>   2011 raineti01            BBWAA     581    436
    ## 3945      NA   NA         <NA>   2011 martied01            BBWAA     581    436
    ## 3946      NA   NA         <NA>   2011 trammal01            BBWAA     581    436
    ## 3947      NA   NA         <NA>   2011 walkela01            BBWAA     581    436
    ## 3948      NA   NA         <NA>   2011 mcgwima01            BBWAA     581    436
    ## 3949      NA   NA         <NA>   2011 mcgrifr01            BBWAA     581    436
    ## 3950      NA   NA         <NA>   2011 parkeda01            BBWAA     581    436
    ## 3951      NA   NA         <NA>   2011 mattido01            BBWAA     581    436
    ## 3952      NA   NA         <NA>   2011 murphda05            BBWAA     581    436
    ## 3953      NA   NA         <NA>   2011 palmera01            BBWAA     581    436
    ## 3954      NA   NA         <NA>   2011 gonzaju03            BBWAA     581    436
    ## 3955      NA   NA         <NA>   2011 baineha01            BBWAA     581    436
    ## 3956      NA   NA         <NA>   2011 francjo01            BBWAA     581    436
    ## 3957      NA   NA         <NA>   2011 brownke01            BBWAA     581    436
    ## 3958      NA   NA         <NA>   2011 martiti02            BBWAA     581    436
    ## 3959      NA   NA         <NA>   2011 grissma02            BBWAA     581    436
    ## 3960      NA   NA         <NA>   2011 leiteal01            BBWAA     581    436
    ## 3961      NA   NA         <NA>   2011 olerujo01            BBWAA     581    436
    ## 3962      NA   NA         <NA>   2011 surhobj01            BBWAA     581    436
    ## 3963      NA   NA         <NA>   2011 boonebr01            BBWAA     581    436
    ## 3964      NA   NA         <NA>   2011 santibe01            BBWAA     581    436
    ## 3965      NA   NA         <NA>   2011 baergca01            BBWAA     581    436
    ## 3966      NA   NA         <NA>   2011 harrile01            BBWAA     581    436
    ## 3967      NA   NA         <NA>   2011 higgibo02            BBWAA     581    436
    ## 3968      NA   NA         <NA>   2011 johnsch04            BBWAA     581    436
    ## 3969      NA   NA         <NA>   2011 mondera01            BBWAA     581    436
    ## 3970      NA   NA         <NA>   2011 rueteki01            BBWAA     581    436
    ## 3971      NA   NA         <NA>   2011 gillipa99         Veterans      NA     NA
    ## 3910      NA   NA         <NA>   2010 dawsoan01            BBWAA     539    405
    ## 3911      NA   NA         <NA>   2010 blylebe01            BBWAA     539    405
    ## 3912      NA   NA         <NA>   2010 alomaro01            BBWAA     539    405
    ## 3913      NA   NA         <NA>   2010 morrija02            BBWAA     539    405
    ## 3914      NA   NA         <NA>   2010 larkiba01            BBWAA     539    405
    ## 3915      NA   NA         <NA>   2010 smithle02            BBWAA     539    405
    ## 3916      NA   NA         <NA>   2010 martied01            BBWAA     539    405
    ## 3917      NA   NA         <NA>   2010 raineti01            BBWAA     539    405
    ## 3918      NA   NA         <NA>   2010 mcgwima01            BBWAA     539    405
    ## 3919      NA   NA         <NA>   2010 trammal01            BBWAA     539    405
    ## 3920      NA   NA         <NA>   2010 mcgrifr01            BBWAA     539    405
    ## 3921      NA   NA         <NA>   2010 mattido01            BBWAA     539    405
    ## 3922      NA   NA         <NA>   2010 parkeda01            BBWAA     539    405
    ## 3923      NA   NA         <NA>   2010 murphda05            BBWAA     539    405
    ## 3924      NA   NA         <NA>   2010 baineha01            BBWAA     539    405
    ## 3925      NA   NA         <NA>   2010 galaran01            BBWAA     539    405
    ## 3926      NA   NA         <NA>   2010 venturo01            BBWAA     539    405
    ## 3927      NA   NA         <NA>   2010 karroer01            BBWAA     539    405
    ## 3928      NA   NA         <NA>   2010 burksel01            BBWAA     539    405
    ## 3929      NA   NA         <NA>   2010 hentgpa01            BBWAA     539    405
    ## 3930      NA   NA         <NA>   2010 appieke01            BBWAA     539    405
    ## 3931      NA   NA         <NA>   2010 seguida01            BBWAA     539    405
    ## 3932      NA   NA         <NA>   2010 reynosh01            BBWAA     539    405
    ## 3933      NA   NA         <NA>   2010 lankfra01            BBWAA     539    405
    ## 3934      NA   NA         <NA>   2010 zeileto01            BBWAA     539    405
    ## 3935      NA   NA         <NA>   2010 jacksmi02            BBWAA     539    405
    ## 3936      NA   NA         <NA>   2010 herzowh01         Veterans      NA     NA
    ## 3937      NA   NA         <NA>   2010 harvedo99         Veterans      NA     NA
    ## 3886      NA   NA         <NA>   2009 henderi01            BBWAA     539    405
    ## 3887      NA   NA         <NA>   2009  riceji01            BBWAA     539    405
    ## 3888      NA   NA         <NA>   2009 dawsoan01            BBWAA     539    405
    ## 3889      NA   NA         <NA>   2009 blylebe01            BBWAA     539    405
    ## 3890      NA   NA         <NA>   2009 smithle02            BBWAA     539    405
    ## 3891      NA   NA         <NA>   2009 morrija02            BBWAA     539    405
    ## 3892      NA   NA         <NA>   2009  johnto01            BBWAA     539    405
    ## 3893      NA   NA         <NA>   2009 raineti01            BBWAA     539    405
    ## 3894      NA   NA         <NA>   2009 mcgwima01            BBWAA     539    405
    ## 3895      NA   NA         <NA>   2009 trammal01            BBWAA     539    405
    ## 3896      NA   NA         <NA>   2009 parkeda01            BBWAA     539    405
    ## 3897      NA   NA         <NA>   2009 mattido01            BBWAA     539    405
    ## 3898      NA   NA         <NA>   2009 murphda05            BBWAA     539    405
    ## 3899      NA   NA         <NA>   2009 baineha01            BBWAA     539    405
    ## 3900      NA   NA         <NA>   2009 gracema01            BBWAA     539    405
    ## 3901      NA   NA         <NA>   2009  coneda01            BBWAA     539    405
    ## 3902      NA   NA         <NA>   2009 willima04            BBWAA     539    405
    ## 3903      NA   NA         <NA>   2009 vaughmo01            BBWAA     539    405
    ## 3904      NA   NA         <NA>   2009  bellja01            BBWAA     539    405
    ## 3905      NA   NA         <NA>   2009 oroscje01            BBWAA     539    405
    ## 3906      NA   NA         <NA>   2009  gantro01            BBWAA     539    405
    ## 3907      NA   NA         <NA>   2009 plesada01            BBWAA     539    405
    ## 3908      NA   NA         <NA>   2009 vaughgr01            BBWAA     539    405
    ## 3909      NA   NA         <NA>   2009 gordojo01         Veterans      NA     NA
    ## 3856      NA   NA         <NA>   2008 gossari01            BBWAA     543    408
    ## 3857      NA   NA         <NA>   2008  riceji01            BBWAA     543    408
    ## 3858      NA   NA         <NA>   2008 dawsoan01            BBWAA     543    408
    ## 3859      NA   NA         <NA>   2008 blylebe01            BBWAA     543    408
    ## 3860      NA   NA         <NA>   2008 smithle02            BBWAA     543    408
    ## 3861      NA   NA         <NA>   2008 morrija02            BBWAA     543    408
    ## 3862      NA   NA         <NA>   2008  johnto01            BBWAA     543    408
    ## 3863      NA   NA         <NA>   2008 raineti01            BBWAA     543    408
    ## 3864      NA   NA         <NA>   2008 mcgwima01            BBWAA     543    408
    ## 3865      NA   NA         <NA>   2008 trammal01            BBWAA     543    408
    ## 3866      NA   NA         <NA>   2008 conceda01            BBWAA     543    408
    ## 3867      NA   NA         <NA>   2008 mattido01            BBWAA     543    408
    ## 3868      NA   NA         <NA>   2008 parkeda01            BBWAA     543    408
    ## 3869      NA   NA         <NA>   2008 murphda05            BBWAA     543    408
    ## 3870      NA   NA         <NA>   2008 baineha01            BBWAA     543    408
    ## 3871      NA   NA         <NA>   2008  beckro01            BBWAA     543    408
    ## 3872      NA   NA         <NA>   2008 frymatr01            BBWAA     543    408
    ## 3873      NA   NA         <NA>   2008   nenro01            BBWAA     543    408
    ## 3874      NA   NA         <NA>   2008 dunstsh01            BBWAA     543    408
    ## 3875      NA   NA         <NA>   2008 finlech01            BBWAA     543    408
    ## 3876      NA   NA         <NA>   2008 justida01            BBWAA     543    408
    ## 3877      NA   NA         <NA>   2008 knoblch01            BBWAA     543    408
    ## 3878      NA   NA         <NA>   2008 stottto01            BBWAA     543    408
    ## 3879      NA   NA         <NA>   2008 anderbr01            BBWAA     543    408
    ## 3880      NA   NA         <NA>   2008  rijojo01            BBWAA     543    408
    ## 3881      NA   NA         <NA>   2008 dreyfba99         Veterans      NA     NA
    ## 3882      NA   NA         <NA>   2008  kuhnbo99         Veterans      NA     NA
    ## 3883      NA   NA         <NA>   2008 omallwa99         Veterans      NA     NA
    ## 3884      NA   NA         <NA>   2008 southbi01         Veterans      NA     NA
    ## 3885      NA   NA         <NA>   2008 willidi02         Veterans      NA     NA
    ## 3824      NA   NA         <NA>   2007 ripkeca01            BBWAA     545    409
    ## 3825      NA   NA         <NA>   2007 gwynnto01            BBWAA     545    409
    ## 3826      NA   NA         <NA>   2007 gossari01            BBWAA     545    409
    ## 3827      NA   NA         <NA>   2007  riceji01            BBWAA     545    409
    ## 3828      NA   NA         <NA>   2007 dawsoan01            BBWAA     545    409
    ## 3829      NA   NA         <NA>   2007 blylebe01            BBWAA     545    409
    ## 3830      NA   NA         <NA>   2007 smithle02            BBWAA     545    409
    ## 3831      NA   NA         <NA>   2007 morrija02            BBWAA     545    409
    ## 3832      NA   NA         <NA>   2007 mcgwima01            BBWAA     545    409
    ## 3833      NA   NA         <NA>   2007  johnto01            BBWAA     545    409
    ## 3834      NA   NA         <NA>   2007 garvest01            BBWAA     545    409
    ## 3835      NA   NA         <NA>   2007 conceda01            BBWAA     545    409
    ## 3836      NA   NA         <NA>   2007 trammal01            BBWAA     545    409
    ## 3837      NA   NA         <NA>   2007 parkeda01            BBWAA     545    409
    ## 3838      NA   NA         <NA>   2007 mattido01            BBWAA     545    409
    ## 3839      NA   NA         <NA>   2007 murphda05            BBWAA     545    409
    ## 3840      NA   NA         <NA>   2007 baineha01            BBWAA     545    409
    ## 3841      NA   NA         <NA>   2007 hershor01            BBWAA     545    409
    ## 3842      NA   NA         <NA>   2007 belleal01            BBWAA     545    409
    ## 3843      NA   NA         <NA>   2007 oneilpa01            BBWAA     545    409
    ## 3844      NA   NA         <NA>   2007 saberbr01            BBWAA     545    409
    ## 3845      NA   NA         <NA>   2007 cansejo01            BBWAA     545    409
    ## 3846      NA   NA         <NA>   2007 fernato01            BBWAA     545    409
    ## 3847      NA   NA         <NA>   2007 bicheda01            BBWAA     545    409
    ## 3848      NA   NA         <NA>   2007 daviser01            BBWAA     545    409
    ## 3849      NA   NA         <NA>   2007 bonilbo01            BBWAA     545    409
    ## 3850      NA   NA         <NA>   2007 caminke01            BBWAA     545    409
    ## 3851      NA   NA         <NA>   2007 buhneja01            BBWAA     545    409
    ## 3852      NA   NA         <NA>   2007 brosisc01            BBWAA     545    409
    ## 3853      NA   NA         <NA>   2007 joynewa01            BBWAA     545    409
    ## 3854      NA   NA         <NA>   2007 whitede03            BBWAA     545    409
    ## 3855      NA   NA         <NA>   2007  wittbo01            BBWAA     545    409
    ## 3778      NA   NA         <NA>   2006 suttebr01            BBWAA     520    390
    ## 3779      NA   NA         <NA>   2006  riceji01            BBWAA     520    390
    ## 3780      NA   NA         <NA>   2006 gossari01            BBWAA     520    390
    ## 3781      NA   NA         <NA>   2006 dawsoan01            BBWAA     520    390
    ## 3782      NA   NA         <NA>   2006 blylebe01            BBWAA     520    390
    ## 3783      NA   NA         <NA>   2006 smithle02            BBWAA     520    390
    ## 3784      NA   NA         <NA>   2006 morrija02            BBWAA     520    390
    ## 3785      NA   NA         <NA>   2006  johnto01            BBWAA     520    390
    ## 3786      NA   NA         <NA>   2006 garvest01            BBWAA     520    390
    ## 3787      NA   NA         <NA>   2006 trammal01            BBWAA     520    390
    ## 3788      NA   NA         <NA>   2006 parkeda01            BBWAA     520    390
    ## 3789      NA   NA         <NA>   2006 conceda01            BBWAA     520    390
    ## 3790      NA   NA         <NA>   2006 mattido01            BBWAA     520    390
    ## 3791      NA   NA         <NA>   2006 hershor01            BBWAA     520    390
    ## 3792      NA   NA         <NA>   2006 murphda05            BBWAA     520    390
    ## 3793      NA   NA         <NA>   2006 belleal01            BBWAA     520    390
    ## 3794      NA   NA         <NA>   2006 clarkwi02            BBWAA     520    390
    ## 3795      NA   NA         <NA>   2006 goodedw01            BBWAA     520    390
    ## 3796      NA   NA         <NA>   2006 mcgeewi01            BBWAA     520    390
    ## 3797      NA   NA         <NA>   2006 guilloz01            BBWAA     520    390
    ## 3798      NA   NA         <NA>   2006 morriha02            BBWAA     520    390
    ## 3799      NA   NA         <NA>   2006 gaettga01            BBWAA     520    390
    ## 3800      NA   NA         <NA>   2006 wettejo01            BBWAA     520    390
    ## 3801      NA   NA         <NA>   2006 aguilri01            BBWAA     520    390
    ## 3802      NA   NA         <NA>   2006 jeffegr01            BBWAA     520    390
    ## 3803      NA   NA         <NA>   2006 jonesdo01            BBWAA     520    390
    ## 3804      NA   NA         <NA>   2006 weisswa01            BBWAA     520    390
    ## 3805      NA   NA         <NA>   2006 disarga01            BBWAA     520    390
    ## 3806      NA   NA         <NA>   2006 fernaal01            BBWAA     520    390
    ## 3807      NA   NA         <NA>   2006 brownra99     Negro League      NA     NA
    ## 3808      NA   NA         <NA>   2006 brownwi02     Negro League      NA     NA
    ## 3809      NA   NA         <NA>   2006 coopean99     Negro League      NA     NA
    ## 3810      NA   NA         <NA>   2006 grantfr99     Negro League      NA     NA
    ## 3811      NA   NA         <NA>   2006  hillpe99     Negro League      NA     NA
    ## 3812      NA   NA         <NA>   2006 mackebi99     Negro League      NA     NA
    ## 3813      NA   NA         <NA>   2006 manleef99     Negro League      NA     NA
    ## 3814      NA   NA         <NA>   2006 mendejo99     Negro League      NA     NA
    ## 3815      NA   NA         <NA>   2006 pompeal99     Negro League      NA     NA
    ## 3816      NA   NA         <NA>   2006 poseycu99     Negro League      NA     NA
    ## 3817      NA   NA         <NA>   2006 santolo99     Negro League      NA     NA
    ## 3818      NA   NA         <NA>   2006 suttlmu99     Negro League      NA     NA
    ## 3819      NA   NA         <NA>   2006 taylobe99     Negro League      NA     NA
    ## 3820      NA   NA         <NA>   2006 torricr99     Negro League      NA     NA
    ## 3821      NA   NA         <NA>   2006 whiteso99     Negro League      NA     NA
    ## 3822      NA   NA         <NA>   2006 wilkijl99     Negro League      NA     NA
    ## 3823      NA   NA         <NA>   2006 wilsoju99     Negro League      NA     NA
    ## 3751      NA   NA         <NA>   2005 boggswa01            BBWAA     516    387
    ## 3752      NA   NA         <NA>   2005 sandbry01            BBWAA     516    387
    ## 3753      NA   NA         <NA>   2005 suttebr01            BBWAA     516    387
    ## 3754      NA   NA         <NA>   2005  riceji01            BBWAA     516    387
    ## 3755      NA   NA         <NA>   2005 gossari01            BBWAA     516    387
    ## 3756      NA   NA         <NA>   2005 dawsoan01            BBWAA     516    387
    ## 3757      NA   NA         <NA>   2005 blylebe01            BBWAA     516    387
    ## 3758      NA   NA         <NA>   2005 smithle02            BBWAA     516    387
    ## 3759      NA   NA         <NA>   2005 morrija02            BBWAA     516    387
    ## 3760      NA   NA         <NA>   2005  johnto01            BBWAA     516    387
    ## 3761      NA   NA         <NA>   2005 garvest01            BBWAA     516    387
    ## 3762      NA   NA         <NA>   2005 trammal01            BBWAA     516    387
    ## 3763      NA   NA         <NA>   2005 parkeda01            BBWAA     516    387
    ## 3764      NA   NA         <NA>   2005 mattido01            BBWAA     516    387
    ## 3765      NA   NA         <NA>   2005 conceda01            BBWAA     516    387
    ## 3766      NA   NA         <NA>   2005 murphda05            BBWAA     516    387
    ## 3767      NA   NA         <NA>   2005 mcgeewi01            BBWAA     516    387
    ## 3768      NA   NA         <NA>   2005 abbotji01            BBWAA     516    387
    ## 3769      NA   NA         <NA>   2005 strawda01            BBWAA     516    387
    ## 3770      NA   NA         <NA>   2005 mcdowja01            BBWAA     516    387
    ## 3771      NA   NA         <NA>   2005 davisch01            BBWAA     516    387
    ## 3772      NA   NA         <NA>   2005 candito01            BBWAA     516    387
    ## 3773      NA   NA         <NA>   2005 montgje01            BBWAA     516    387
    ## 3774      NA   NA         <NA>   2005 phillto02            BBWAA     516    387
    ## 3775      NA   NA         <NA>   2005 steinte01            BBWAA     516    387
    ## 3776      NA   NA         <NA>   2005 langsma01            BBWAA     516    387
    ## 3777      NA   NA         <NA>   2005 nixonot01            BBWAA     516    387
    ## 3719      NA   NA         <NA>   2004 molitpa01            BBWAA     506    380
    ## 3720      NA   NA         <NA>   2004 eckerde01            BBWAA     506    380
    ## 3721      NA   NA         <NA>   2004 sandbry01            BBWAA     506    380
    ## 3722      NA   NA         <NA>   2004 suttebr01            BBWAA     506    380
    ## 3723      NA   NA         <NA>   2004  riceji01            BBWAA     506    380
    ## 3724      NA   NA         <NA>   2004 dawsoan01            BBWAA     506    380
    ## 3725      NA   NA         <NA>   2004 gossari01            BBWAA     506    380
    ## 3726      NA   NA         <NA>   2004 smithle02            BBWAA     506    380
    ## 3727      NA   NA         <NA>   2004 blylebe01            BBWAA     506    380
    ## 3728      NA   NA         <NA>   2004 morrija02            BBWAA     506    380
    ## 3729      NA   NA         <NA>   2004 garvest01            BBWAA     506    380
    ## 3730      NA   NA         <NA>   2004  johnto01            BBWAA     506    380
    ## 3731      NA   NA         <NA>   2004 trammal01            BBWAA     506    380
    ## 3732      NA   NA         <NA>   2004 mattido01            BBWAA     506    380
    ## 3733      NA   NA         <NA>   2004 conceda01            BBWAA     506    380
    ## 3734      NA   NA         <NA>   2004 parkeda01            BBWAA     506    380
    ## 3735      NA   NA         <NA>   2004 murphda05            BBWAA     506    380
    ## 3736      NA   NA         <NA>   2004 hernake01            BBWAA     506    380
    ## 3737      NA   NA         <NA>   2004 cartejo01            BBWAA     506    380
    ## 3738      NA   NA         <NA>   2004 valenfe01            BBWAA     506    380
    ## 3739      NA   NA         <NA>   2004 martide01            BBWAA     506    380
    ## 3740      NA   NA         <NA>   2004 stiebda01            BBWAA     506    380
    ## 3741      NA   NA         <NA>   2004 eisenji01            BBWAA     506    380
    ## 3742      NA   NA         <NA>   2004   keyji01            BBWAA     506    380
    ## 3743      NA   NA         <NA>   2004 drabedo01            BBWAA     506    380
    ## 3744      NA   NA         <NA>   2004 mitchke01            BBWAA     506    380
    ## 3745      NA   NA         <NA>   2004 samueju01            BBWAA     506    380
    ## 3746      NA   NA         <NA>   2004 fieldce01            BBWAA     506    380
    ## 3747      NA   NA         <NA>   2004 myersra01            BBWAA     506    380
    ## 3748      NA   NA         <NA>   2004 pendlte01            BBWAA     506    380
    ## 3749      NA   NA         <NA>   2004 darwida01            BBWAA     506    380
    ## 3750      NA   NA         <NA>   2004 tewksbo01            BBWAA     506    380
    ## 3686      NA   NA         <NA>   2003 murraed02            BBWAA     496    372
    ## 3687      NA   NA         <NA>   2003 cartega01            BBWAA     496    372
    ## 3688      NA   NA         <NA>   2003 suttebr01            BBWAA     496    372
    ## 3689      NA   NA         <NA>   2003  riceji01            BBWAA     496    372
    ## 3690      NA   NA         <NA>   2003 dawsoan01            BBWAA     496    372
    ## 3691      NA   NA         <NA>   2003 sandbry01            BBWAA     496    372
    ## 3692      NA   NA         <NA>   2003 smithle02            BBWAA     496    372
    ## 3693      NA   NA         <NA>   2003 gossari01            BBWAA     496    372
    ## 3694      NA   NA         <NA>   2003 blylebe01            BBWAA     496    372
    ## 3695      NA   NA         <NA>   2003 garvest01            BBWAA     496    372
    ## 3696      NA   NA         <NA>   2003  kaatji01            BBWAA     496    372
    ## 3697      NA   NA         <NA>   2003  johnto01            BBWAA     496    372
    ## 3698      NA   NA         <NA>   2003 morrija02            BBWAA     496    372
    ## 3699      NA   NA         <NA>   2003 trammal01            BBWAA     496    372
    ## 3700      NA   NA         <NA>   2003 mattido01            BBWAA     496    372
    ## 3701      NA   NA         <NA>   2003 murphda05            BBWAA     496    372
    ## 3702      NA   NA         <NA>   2003 conceda01            BBWAA     496    372
    ## 3703      NA   NA         <NA>   2003 parkeda01            BBWAA     496    372
    ## 3704      NA   NA         <NA>   2003 valenfe01            BBWAA     496    372
    ## 3705      NA   NA         <NA>   2003 hernake01            BBWAA     496    372
    ## 3706      NA   NA         <NA>   2003  kileda01            BBWAA     496    372
    ## 3707      NA   NA         <NA>   2003 colemvi01            BBWAA     496    372
    ## 3708      NA   NA         <NA>   2003 butlebr01            BBWAA     496    372
    ## 3709      NA   NA         <NA>   2003 fernasi01            BBWAA     496    372
    ## 3710      NA   NA         <NA>   2003 honeyri01            BBWAA     496    372
    ## 3711      NA   NA         <NA>   2003  penato01            BBWAA     496    372
    ## 3712      NA   NA         <NA>   2003 daultda01            BBWAA     496    372
    ## 3713      NA   NA         <NA>   2003 davisma01            BBWAA     496    372
    ## 3714      NA   NA         <NA>   2003 tartada01            BBWAA     496    372
    ## 3715      NA   NA         <NA>   2003 jacksda02            BBWAA     496    372
    ## 3716      NA   NA         <NA>   2003 tettlmi01            BBWAA     496    372
    ## 3717      NA   NA         <NA>   2003 willimi02            BBWAA     496    372
    ## 3718      NA   NA         <NA>   2003 worreto01            BBWAA     496    372
    ## 3658      NA   NA         <NA>   2002 smithoz01            BBWAA     472    354
    ## 3659      NA   NA         <NA>   2002 cartega01            BBWAA     472    354
    ## 3660      NA   NA         <NA>   2002  riceji01            BBWAA     472    354
    ## 3661      NA   NA         <NA>   2002 suttebr01            BBWAA     472    354
    ## 3662      NA   NA         <NA>   2002 dawsoan01            BBWAA     472    354
    ## 3663      NA   NA         <NA>   2002 gossari01            BBWAA     472    354
    ## 3664      NA   NA         <NA>   2002 garvest01            BBWAA     472    354
    ## 3665      NA   NA         <NA>   2002  johnto01            BBWAA     472    354
    ## 3666      NA   NA         <NA>   2002 blylebe01            BBWAA     472    354
    ## 3667      NA   NA         <NA>   2002  kaatji01            BBWAA     472    354
    ## 3668      NA   NA         <NA>   2002 morrija02            BBWAA     472    354
    ## 3669      NA   NA         <NA>   2002 mattido01            BBWAA     472    354
    ## 3670      NA   NA         <NA>   2002 tiantlu01            BBWAA     472    354
    ## 3671      NA   NA         <NA>   2002 trammal01            BBWAA     472    354
    ## 3672      NA   NA         <NA>   2002 murphda05            BBWAA     472    354
    ## 3673      NA   NA         <NA>   2002 parkeda01            BBWAA     472    354
    ## 3674      NA   NA         <NA>   2002 conceda01            BBWAA     472    354
    ## 3675      NA   NA         <NA>   2002 hernake01            BBWAA     472    354
    ## 3676      NA   NA         <NA>   2002 guidrro01            BBWAA     472    354
    ## 3677      NA   NA         <NA>   2002 stewada01            BBWAA     472    354
    ## 3678      NA   NA         <NA>   2002 greenmi01            BBWAA     472    354
    ## 3679      NA   NA         <NA>   2002 violafr01            BBWAA     472    354
    ## 3680      NA   NA         <NA>   2002 dykstle01            BBWAA     472    354
    ## 3681      NA   NA         <NA>   2002 wallati01            BBWAA     472    354
    ## 3682      NA   NA         <NA>   2002 hennemi01            BBWAA     472    354
    ## 3683      NA   NA         <NA>   2002 russeje01            BBWAA     472    354
    ## 3684      NA   NA         <NA>   2002 sandesc02            BBWAA     472    354
    ## 3685      NA   NA         <NA>   2002 thompro01            BBWAA     472    354
    ## 3624      NA   NA         <NA>   2001 winfida01            BBWAA     515    387
    ## 3625      NA   NA         <NA>   2001 puckeki01            BBWAA     515    387
    ## 3626      NA   NA         <NA>   2001 cartega01            BBWAA     515    387
    ## 3627      NA   NA         <NA>   2001  riceji01            BBWAA     515    387
    ## 3628      NA   NA         <NA>   2001 suttebr01            BBWAA     515    387
    ## 3629      NA   NA         <NA>   2001 gossari01            BBWAA     515    387
    ## 3630      NA   NA         <NA>   2001 garvest01            BBWAA     515    387
    ## 3631      NA   NA         <NA>   2001  johnto01            BBWAA     515    387
    ## 3632      NA   NA         <NA>   2001 mattido01            BBWAA     515    387
    ## 3633      NA   NA         <NA>   2001  kaatji01            BBWAA     515    387
    ## 3634      NA   NA         <NA>   2001 blylebe01            BBWAA     515    387
    ## 3635      NA   NA         <NA>   2001 morrija02            BBWAA     515    387
    ## 3636      NA   NA         <NA>   2001 murphda05            BBWAA     515    387
    ## 3637      NA   NA         <NA>   2001 parkeda01            BBWAA     515    387
    ## 3638      NA   NA         <NA>   2001 conceda01            BBWAA     515    387
    ## 3639      NA   NA         <NA>   2001 tiantlu01            BBWAA     515    387
    ## 3640      NA   NA         <NA>   2001 hernake01            BBWAA     515    387
    ## 3641      NA   NA         <NA>   2001 stewada01            BBWAA     515    387
    ## 3642      NA   NA         <NA>   2001 guidrro01            BBWAA     515    387
    ## 3643      NA   NA         <NA>   2001 whitalo01            BBWAA     515    387
    ## 3644      NA   NA         <NA>   2001 gibsoki01            BBWAA     515    387
    ## 3645      NA   NA         <NA>   2001 parrila02            BBWAA     515    387
    ## 3646      NA   NA         <NA>   2001 henketo01            BBWAA     515    387
    ## 3647      NA   NA         <NA>   2001 righeda01            BBWAA     515    387
    ## 3648      NA   NA         <NA>   2001 bedrost01            BBWAA     515    387
    ## 3649      NA   NA         <NA>   2001 brownto05            BBWAA     515    387
    ## 3650      NA   NA         <NA>   2001 darliro01            BBWAA     515    387
    ## 3651      NA   NA         <NA>   2001 deshaji01            BBWAA     515    387
    ## 3652      NA   NA         <NA>   2001  krukjo01            BBWAA     515    387
    ## 3653      NA   NA         <NA>   2001  rijojo01            BBWAA     515    387
    ## 3654      NA   NA         <NA>   2001 johnsho01            BBWAA     515    387
    ## 3655      NA   NA         <NA>   2001 vanslan01            BBWAA     515    387
    ## 3656      NA   NA         <NA>   2001 mazerbi01         Veterans      NA     NA
    ## 3657      NA   NA         <NA>   2001 smithhi99         Veterans      NA     NA
    ## 3591      NA   NA         <NA>   2000  fiskca01            BBWAA     499    375
    ## 3592      NA   NA         <NA>   2000 perezto01            BBWAA     499    375
    ## 3593      NA   NA         <NA>   2000  riceji01            BBWAA     499    375
    ## 3594      NA   NA         <NA>   2000 cartega01            BBWAA     499    375
    ## 3595      NA   NA         <NA>   2000 suttebr01            BBWAA     499    375
    ## 3596      NA   NA         <NA>   2000 gossari01            BBWAA     499    375
    ## 3597      NA   NA         <NA>   2000 garvest01            BBWAA     499    375
    ## 3598      NA   NA         <NA>   2000  johnto01            BBWAA     499    375
    ## 3599      NA   NA         <NA>   2000  kaatji01            BBWAA     499    375
    ## 3600      NA   NA         <NA>   2000 murphda05            BBWAA     499    375
    ## 3601      NA   NA         <NA>   2000 morrija02            BBWAA     499    375
    ## 3602      NA   NA         <NA>   2000 parkeda01            BBWAA     499    375
    ## 3603      NA   NA         <NA>   2000 blylebe01            BBWAA     499    375
    ## 3604      NA   NA         <NA>   2000 tiantlu01            BBWAA     499    375
    ## 3605      NA   NA         <NA>   2000 conceda01            BBWAA     499    375
    ## 3606      NA   NA         <NA>   2000 hernake01            BBWAA     499    375
    ## 3607      NA   NA         <NA>   2000 guidrro01            BBWAA     499    375
    ## 3608      NA   NA         <NA>   2000 reardje01            BBWAA     499    375
    ## 3609      NA   NA         <NA>   2000 boonebo01            BBWAA     499    375
    ## 3610      NA   NA         <NA>   2000 wilsowi02            BBWAA     499    375
    ## 3611      NA   NA         <NA>   2000 sutclri01            BBWAA     499    375
    ## 3612      NA   NA         <NA>   2000 hrbekke01            BBWAA     499    375
    ## 3613      NA   NA         <NA>   2000 houghch01            BBWAA     499    375
    ## 3614      NA   NA         <NA>   2000 hendeda01            BBWAA     499    375
    ## 3615      NA   NA         <NA>   2000   saxst01            BBWAA     499    375
    ## 3616      NA   NA         <NA>   2000 gullibi01            BBWAA     499    375
    ## 3617      NA   NA         <NA>   2000 hurstbr01            BBWAA     499    375
    ## 3618      NA   NA         <NA>   2000 smithlo01            BBWAA     499    375
    ## 3619      NA   NA         <NA>   2000 welchbo01            BBWAA     499    375
    ## 3620      NA   NA         <NA>   2000 brookhu01            BBWAA     499    375
    ## 3621      NA   NA         <NA>   2000 andersp01         Veterans      NA     NA
    ## 3622      NA   NA         <NA>   2000 mcphebi01         Veterans      NA     NA
    ## 3623      NA   NA         <NA>   2000 steartu99         Veterans      NA     NA
    ## 3559      NA   NA         <NA>   1999  ryanno01            BBWAA     497    373
    ## 3560      NA   NA         <NA>   1999 brettge01            BBWAA     497    373
    ## 3561      NA   NA         <NA>   1999 yountro01            BBWAA     497    373
    ## 3562      NA   NA         <NA>   1999  fiskca01            BBWAA     497    373
    ## 3563      NA   NA         <NA>   1999 perezto01            BBWAA     497    373
    ## 3564      NA   NA         <NA>   1999 cartega01            BBWAA     497    373
    ## 3565      NA   NA         <NA>   1999 garvest01            BBWAA     497    373
    ## 3566      NA   NA         <NA>   1999  riceji01            BBWAA     497    373
    ## 3567      NA   NA         <NA>   1999 suttebr01            BBWAA     497    373
    ## 3568      NA   NA         <NA>   1999  kaatji01            BBWAA     497    373
    ## 3569      NA   NA         <NA>   1999 murphda05            BBWAA     497    373
    ## 3570      NA   NA         <NA>   1999  johnto01            BBWAA     497    373
    ## 3571      NA   NA         <NA>   1999 parkeda01            BBWAA     497    373
    ## 3572      NA   NA         <NA>   1999 minosmi01            BBWAA     497    373
    ## 3573      NA   NA         <NA>   1999 blylebe01            BBWAA     497    373
    ## 3574      NA   NA         <NA>   1999 conceda01            BBWAA     497    373
    ## 3575      NA   NA         <NA>   1999 tiantlu01            BBWAA     497    373
    ## 3576      NA   NA         <NA>   1999 hernake01            BBWAA     497    373
    ## 3577      NA   NA         <NA>   1999 guidrro01            BBWAA     497    373
    ## 3578      NA   NA         <NA>   1999 boonebo01            BBWAA     497    373
    ## 3579      NA   NA         <NA>   1999 lolicmi01            BBWAA     497    373
    ## 3580      NA   NA         <NA>   1999 evansdw01            BBWAA     497    373
    ## 3581      NA   NA         <NA>   1999  bellge02            BBWAA     497    373
    ## 3582      NA   NA         <NA>   1999 candejo01            BBWAA     497    373
    ## 3583      NA   NA         <NA>   1999 boddimi01            BBWAA     497    373
    ## 3584      NA   NA         <NA>   1999 leibrch01            BBWAA     497    373
    ## 3585      NA   NA         <NA>   1999 tananfr01            BBWAA     497    373
    ## 3586      NA   NA         <NA>   1999  wittmi01            BBWAA     497    373
    ## 3587      NA   NA         <NA>   1999 cepedor01         Veterans      NA     NA
    ## 3588      NA   NA         <NA>   1999 chylane99         Veterans      NA     NA
    ## 3589      NA   NA         <NA>   1999 seleefr99         Veterans      NA     NA
    ## 3590      NA   NA         <NA>   1999 willijo99         Veterans      NA     NA
    ## 3529      NA   NA         <NA>   1998 suttodo01            BBWAA     473    355
    ## 3530      NA   NA         <NA>   1998 perezto01            BBWAA     473    355
    ## 3531      NA   NA         <NA>   1998 santoro01            BBWAA     473    355
    ## 3532      NA   NA         <NA>   1998  riceji01            BBWAA     473    355
    ## 3533      NA   NA         <NA>   1998 cartega01            BBWAA     473    355
    ## 3534      NA   NA         <NA>   1998 garvest01            BBWAA     473    355
    ## 3535      NA   NA         <NA>   1998 suttebr01            BBWAA     473    355
    ## 3536      NA   NA         <NA>   1998  johnto01            BBWAA     473    355
    ## 3537      NA   NA         <NA>   1998  kaatji01            BBWAA     473    355
    ## 3538      NA   NA         <NA>   1998 parkeda01            BBWAA     473    355
    ## 3539      NA   NA         <NA>   1998 blylebe01            BBWAA     473    355
    ## 3540      NA   NA         <NA>   1998 conceda01            BBWAA     473    355
    ## 3541      NA   NA         <NA>   1998 minosmi01            BBWAA     473    355
    ## 3542      NA   NA         <NA>   1998 tiantlu01            BBWAA     473    355
    ## 3543      NA   NA         <NA>   1998 hernake01            BBWAA     473    355
    ## 3544      NA   NA         <NA>   1998 evansdw01            BBWAA     473    355
    ## 3545      NA   NA         <NA>   1998 lolicmi01            BBWAA     473    355
    ## 3546      NA   NA         <NA>   1998 guidrro01            BBWAA     473    355
    ## 3547      NA   NA         <NA>   1998 boonebo01            BBWAA     473    355
    ## 3548      NA   NA         <NA>   1998 clarkja01            BBWAA     473    355
    ## 3549      NA   NA         <NA>   1998 guerrpe01            BBWAA     473    355
    ## 3550      NA   NA         <NA>   1998 randowi01            BBWAA     473    355
    ## 3551      NA   NA         <NA>   1998 lansfca01            BBWAA     473    355
    ## 3552      NA   NA         <NA>   1998 downibr01            BBWAA     473    355
    ## 3553      NA   NA         <NA>   1998 flanami01            BBWAA     473    355
    ## 3554      NA   NA         <NA>   1998 dempsri01            BBWAA     473    355
    ## 3555      NA   NA         <NA>   1998 davisge01         Veterans      NA     NA
    ## 3556      NA   NA         <NA>   1998  dobyla01         Veterans      NA     NA
    ## 3557      NA   NA         <NA>   1998 macphle99         Veterans      NA     NA
    ## 3558      NA   NA         <NA>   1998 roganbu99         Veterans      NA     NA
    ## 3496      NA   NA         <NA>   1997 niekrph01            BBWAA     473    353
    ## 3497      NA   NA         <NA>   1997 suttodo01            BBWAA     473    353
    ## 3498      NA   NA         <NA>   1997 perezto01            BBWAA     473    353
    ## 3499      NA   NA         <NA>   1997 santoro01            BBWAA     473    353
    ## 3500      NA   NA         <NA>   1997  riceji01            BBWAA     473    353
    ## 3501      NA   NA         <NA>   1997 garvest01            BBWAA     473    353
    ## 3502      NA   NA         <NA>   1997 suttebr01            BBWAA     473    353
    ## 3503      NA   NA         <NA>   1997  kaatji01            BBWAA     473    353
    ## 3504      NA   NA         <NA>   1997 torrejo01            BBWAA     473    353
    ## 3505      NA   NA         <NA>   1997  johnto01            BBWAA     473    353
    ## 3506      NA   NA         <NA>   1997 minosmi01            BBWAA     473    353
    ## 3507      NA   NA         <NA>   1997 parkeda01            BBWAA     473    353
    ## 3508      NA   NA         <NA>   1997 allendi01            BBWAA     473    353
    ## 3509      NA   NA         <NA>   1997 conceda01            BBWAA     473    353
    ## 3510      NA   NA         <NA>   1997 tiantlu01            BBWAA     473    353
    ## 3511      NA   NA         <NA>   1997 hernake01            BBWAA     473    353
    ## 3512      NA   NA         <NA>   1997 lolicmi01            BBWAA     473    353
    ## 3513      NA   NA         <NA>   1997 guidrro01            BBWAA     473    353
    ## 3514      NA   NA         <NA>   1997 boonebo01            BBWAA     473    353
    ## 3515      NA   NA         <NA>   1997 evansdw01            BBWAA     473    353
    ## 3516      NA   NA         <NA>   1997 griffke01            BBWAA     473    353
    ## 3517      NA   NA         <NA>   1997  lynnfr01            BBWAA     473    353
    ## 3518      NA   NA         <NA>   1997 nettlgr01            BBWAA     473    353
    ## 3519      NA   NA         <NA>   1997 bondsbo01            BBWAA     473    353
    ## 3520      NA   NA         <NA>   1997 staubru01            BBWAA     473    353
    ## 3521      NA   NA         <NA>   1997 reuscri01            BBWAA     473    353
    ## 3522      NA   NA         <NA>   1997 scottmi03            BBWAA     473    353
    ## 3523      NA   NA         <NA>   1997 templga01            BBWAA     473    353
    ## 3524      NA   NA         <NA>   1997 kennete02            BBWAA     473    353
    ## 3525      NA   NA         <NA>   1997  puhlte01            BBWAA     473    353
    ## 3526      NA   NA         <NA>   1997   foxne01         Veterans      NA     NA
    ## 3527      NA   NA         <NA>   1997 lasorto01         Veterans      NA     NA
    ## 3528      NA   NA         <NA>   1997 wellswi99         Veterans      NA     NA
    ## 3457      NA   NA         <NA>   1996 niekrph01            BBWAA     470    353
    ## 3458      NA   NA         <NA>   1996 perezto01            BBWAA     470    353
    ## 3459      NA   NA         <NA>   1996 suttodo01            BBWAA     470    353
    ## 3460      NA   NA         <NA>   1996 garvest01            BBWAA     470    353
    ## 3461      NA   NA         <NA>   1996 santoro01            BBWAA     470    353
    ## 3462      NA   NA         <NA>   1996 olivato01            BBWAA     470    353
    ## 3463      NA   NA         <NA>   1996  riceji01            BBWAA     470    353
    ## 3464      NA   NA         <NA>   1996 suttebr01            BBWAA     470    353
    ## 3465      NA   NA         <NA>   1996  johnto01            BBWAA     470    353
    ## 3466      NA   NA         <NA>   1996  kaatji01            BBWAA     470    353
    ## 3467      NA   NA         <NA>   1996 allendi01            BBWAA     470    353
    ## 3468      NA   NA         <NA>   1996 floodcu01            BBWAA     470    353
    ## 3469      NA   NA         <NA>   1996 tiantlu01            BBWAA     470    353
    ## 3470      NA   NA         <NA>   1996 conceda01            BBWAA     470    353
    ## 3471      NA   NA         <NA>   1996 minosmi01            BBWAA     470    353
    ## 3472      NA   NA         <NA>   1996 pinsova01            BBWAA     470    353
    ## 3473      NA   NA         <NA>   1996 torrejo01            BBWAA     470    353
    ## 3474      NA   NA         <NA>   1996 guidrro01            BBWAA     470    353
    ## 3475      NA   NA         <NA>   1996 nettlgr01            BBWAA     470    353
    ## 3476      NA   NA         <NA>   1996 boonebo01            BBWAA     470    353
    ## 3477      NA   NA         <NA>   1996 lolicmi01            BBWAA     470    353
    ## 3478      NA   NA         <NA>   1996  lynnfr01            BBWAA     470    353
    ## 3479      NA   NA         <NA>   1996 bondsbo01            BBWAA     470    353
    ## 3480      NA   NA         <NA>   1996 hernake01            BBWAA     470    353
    ## 3481      NA   NA         <NA>   1996 staubru01            BBWAA     470    353
    ## 3482      NA   NA         <NA>   1996 quiseda01            BBWAA     470    353
    ## 3483      NA   NA         <NA>   1996 whitefr01            BBWAA     470    353
    ## 3484      NA   NA         <NA>   1996 bucknbi01            BBWAA     470    353
    ## 3485      NA   NA         <NA>   1996 reussje01            BBWAA     470    353
    ## 3486      NA   NA         <NA>   1996 tudorjo01            BBWAA     470    353
    ## 3487      NA   NA         <NA>   1996 lemonch01            BBWAA     470    353
    ## 3488      NA   NA         <NA>   1996 kneppbo01            BBWAA     470    353
    ## 3489      NA   NA         <NA>   1996 leonaje01            BBWAA     470    353
    ## 3490      NA   NA         <NA>   1996   rayjo01            BBWAA     470    353
    ## 3491      NA   NA         <NA>   1996 washicl01            BBWAA     470    353
    ## 3492      NA   NA         <NA>   1996 bunniji01         Veterans      NA     NA
    ## 3493      NA   NA         <NA>   1996 fostebi99         Veterans      NA     NA
    ## 3494      NA   NA         <NA>   1996 hanlone01         Veterans      NA     NA
    ## 3495      NA   NA         <NA>   1996 weaveea99         Veterans      NA     NA
    ## 3414      NA   NA         <NA>   1995 schmimi01            BBWAA     460    345
    ## 3415      NA   NA         <NA>   1995 niekrph01            BBWAA     460    345
    ## 3416      NA   NA         <NA>   1995 suttodo01            BBWAA     460    345
    ## 3417      NA   NA         <NA>   1995 perezto01            BBWAA     460    345
    ## 3418      NA   NA         <NA>   1995 garvest01            BBWAA     460    345
    ## 3419      NA   NA         <NA>   1995 olivato01            BBWAA     460    345
    ## 3420      NA   NA         <NA>   1995 santoro01            BBWAA     460    345
    ## 3421      NA   NA         <NA>   1995  riceji01            BBWAA     460    345
    ## 3422      NA   NA         <NA>   1995 suttebr01            BBWAA     460    345
    ## 3423      NA   NA         <NA>   1995  kaatji01            BBWAA     460    345
    ## 3424      NA   NA         <NA>   1995  johnto01            BBWAA     460    345
    ## 3425      NA   NA         <NA>   1995 allendi01            BBWAA     460    345
    ## 3426      NA   NA         <NA>   1995 minosmi01            BBWAA     460    345
    ## 3427      NA   NA         <NA>   1995 floodcu01            BBWAA     460    345
    ## 3428      NA   NA         <NA>   1995 torrejo01            BBWAA     460    345
    ## 3429      NA   NA         <NA>   1995 tiantlu01            BBWAA     460    345
    ## 3430      NA   NA         <NA>   1995 conceda01            BBWAA     460    345
    ## 3431      NA   NA         <NA>   1995 bondsbo01            BBWAA     460    345
    ## 3432      NA   NA         <NA>   1995 pinsova01            BBWAA     460    345
    ## 3433      NA   NA         <NA>   1995 munsoth01            BBWAA     460    345
    ## 3434      NA   NA         <NA>   1995 nettlgr01            BBWAA     460    345
    ## 3435      NA   NA         <NA>   1995  bluevi01            BBWAA     460    345
    ## 3436      NA   NA         <NA>   1995 lolicmi01            BBWAA     460    345
    ## 3437      NA   NA         <NA>   1995 guidrro01            BBWAA     460    345
    ## 3438      NA   NA         <NA>   1995 staubru01            BBWAA     460    345
    ## 3439      NA   NA         <NA>   1995 fostege01            BBWAA     460    345
    ## 3440      NA   NA         <NA>   1995 baylodo01            BBWAA     460    345
    ## 3441      NA   NA         <NA>   1995  bellbu01            BBWAA     460    345
    ## 3442      NA   NA         <NA>   1995 evansda01            BBWAA     460    345
    ## 3443      NA   NA         <NA>   1995 tekulke01            BBWAA     460    345
    ## 3444      NA   NA         <NA>   1995 forscbo01            BBWAA     460    345
    ## 3445      NA   NA         <NA>   1995 hernawi01            BBWAA     460    345
    ## 3446      NA   NA         <NA>   1995 krukomi01            BBWAA     460    345
    ## 3447      NA   NA         <NA>   1995 speiech01            BBWAA     460    345
    ## 3448      NA   NA         <NA>   1995 sundbji01            BBWAA     460    345
    ## 3449      NA   NA         <NA>   1995 alexado01            BBWAA     460    345
    ## 3450      NA   NA         <NA>   1995 grossgr01            BBWAA     460    345
    ## 3451      NA   NA         <NA>   1995 rhoderi01            BBWAA     460    345
    ## 3452      NA   NA         <NA>   1995 trillma01            BBWAA     460    345
    ## 3453      NA   NA         <NA>   1995 ashburi01         Veterans      NA     NA
    ## 3454      NA   NA         <NA>   1995   dayle99         Veterans      NA     NA
    ## 3455      NA   NA         <NA>   1995 hulbewi99         Veterans      NA     NA
    ## 3456      NA   NA         <NA>   1995 willivi01         Veterans      NA     NA
    ## 3373      NA   NA         <NA>   1994 carltst01            BBWAA     456    342
    ## 3374      NA   NA         <NA>   1994 cepedor01            BBWAA     456    342
    ## 3375      NA   NA         <NA>   1994 niekrph01            BBWAA     456    342
    ## 3376      NA   NA         <NA>   1994 perezto01            BBWAA     456    342
    ## 3377      NA   NA         <NA>   1994 suttodo01            BBWAA     456    342
    ## 3378      NA   NA         <NA>   1994 garvest01            BBWAA     456    342
    ## 3379      NA   NA         <NA>   1994 olivato01            BBWAA     456    342
    ## 3380      NA   NA         <NA>   1994 santoro01            BBWAA     456    342
    ## 3381      NA   NA         <NA>   1994 suttebr01            BBWAA     456    342
    ## 3382      NA   NA         <NA>   1994  kaatji01            BBWAA     456    342
    ## 3383      NA   NA         <NA>   1994 allendi01            BBWAA     456    342
    ## 3384      NA   NA         <NA>   1994 boyerke01            BBWAA     456    342
    ## 3385      NA   NA         <NA>   1994 torrejo01            BBWAA     456    342
    ## 3386      NA   NA         <NA>   1994 pinsova01            BBWAA     456    342
    ## 3387      NA   NA         <NA>   1994 minosmi01            BBWAA     456    342
    ## 3388      NA   NA         <NA>   1994 tiantlu01            BBWAA     456    342
    ## 3389      NA   NA         <NA>   1994 floodcu01            BBWAA     456    342
    ## 3390      NA   NA         <NA>   1994 nettlgr01            BBWAA     456    342
    ## 3391      NA   NA         <NA>   1994 bondsbo01            BBWAA     456    342
    ## 3392      NA   NA         <NA>   1994 staubru01            BBWAA     456    342
    ## 3393      NA   NA         <NA>   1994 conceda01            BBWAA     456    342
    ## 3394      NA   NA         <NA>   1994 munsoth01            BBWAA     456    342
    ## 3395      NA   NA         <NA>   1994 guidrro01            BBWAA     456    342
    ## 3396      NA   NA         <NA>   1994 lolicmi01            BBWAA     456    342
    ## 3397      NA   NA         <NA>   1994  rosepe01            BBWAA     456    342
    ## 3398      NA   NA         <NA>   1994 simmote01            BBWAA     456    342
    ## 3399      NA   NA         <NA>   1994 fostege01            BBWAA     456    342
    ## 3400      NA   NA         <NA>   1994  bluevi01            BBWAA     456    342
    ## 3401      NA   NA         <NA>   1994 baylodo01            BBWAA     456    342
    ## 3402      NA   NA         <NA>   1994 niekrjo01            BBWAA     456    342
    ## 3403      NA   NA         <NA>   1994  cruzjo01            BBWAA     456    342
    ## 3404      NA   NA         <NA>   1994 garneph01            BBWAA     456    342
    ## 3405      NA   NA         <NA>   1994 parrila01            BBWAA     456    342
    ## 3406      NA   NA         <NA>   1994 knighra01            BBWAA     456    342
    ## 3407      NA   NA         <NA>   1994 chambch01            BBWAA     456    342
    ## 3408      NA   NA         <NA>   1994 hendrge01            BBWAA     456    342
    ## 3409      NA   NA         <NA>   1994 hornebo01            BBWAA     456    342
    ## 3410      NA   NA         <NA>   1994 mcgresc01            BBWAA     456    342
    ## 3411      NA   NA         <NA>   1994  sotoma01            BBWAA     456    342
    ## 3412      NA   NA         <NA>   1994 durocle01         Veterans      NA     NA
    ## 3413      NA   NA         <NA>   1994 rizzuph01         Veterans      NA     NA
    ## 3340      NA   NA         <NA>   1993 jacksre01            BBWAA     423    318
    ## 3341      NA   NA         <NA>   1993 niekrph01            BBWAA     423    318
    ## 3342      NA   NA         <NA>   1993 cepedor01            BBWAA     423    318
    ## 3343      NA   NA         <NA>   1993 perezto01            BBWAA     423    318
    ## 3344      NA   NA         <NA>   1993 garvest01            BBWAA     423    318
    ## 3345      NA   NA         <NA>   1993 olivato01            BBWAA     423    318
    ## 3346      NA   NA         <NA>   1993 santoro01            BBWAA     423    318
    ## 3347      NA   NA         <NA>   1993  kaatji01            BBWAA     423    318
    ## 3348      NA   NA         <NA>   1993 allendi01            BBWAA     423    318
    ## 3349      NA   NA         <NA>   1993 boyerke01            BBWAA     423    318
    ## 3350      NA   NA         <NA>   1993 minosmi01            BBWAA     423    318
    ## 3351      NA   NA         <NA>   1993 torrejo01            BBWAA     423    318
    ## 3352      NA   NA         <NA>   1993 tiantlu01            BBWAA     423    318
    ## 3353      NA   NA         <NA>   1993 bondsbo01            BBWAA     423    318
    ## 3354      NA   NA         <NA>   1993 lolicmi01            BBWAA     423    318
    ## 3355      NA   NA         <NA>   1993 munsoth01            BBWAA     423    318
    ## 3356      NA   NA         <NA>   1993 pinsova01            BBWAA     423    318
    ## 3357      NA   NA         <NA>   1993  bluevi01            BBWAA     423    318
    ## 3358      NA   NA         <NA>   1993 floodcu01            BBWAA     423    318
    ## 3359      NA   NA         <NA>   1993 staubru01            BBWAA     423    318
    ## 3360      NA   NA         <NA>   1993 fostege01            BBWAA     423    318
    ## 3361      NA   NA         <NA>   1993 madlobi01            BBWAA     423    318
    ## 3362      NA   NA         <NA>   1993  rosepe01            BBWAA     423    318
    ## 3363      NA   NA         <NA>   1993   ceyro01            BBWAA     423    318
    ## 3364      NA   NA         <NA>   1993 decindo01            BBWAA     423    318
    ## 3365      NA   NA         <NA>   1993 lopesda01            BBWAA     423    318
    ## 3366      NA   NA         <NA>   1993 thornan01            BBWAA     423    318
    ## 3367      NA   NA         <NA>   1993 campbbi02            BBWAA     423    318
    ## 3368      NA   NA         <NA>   1993 burleri01            BBWAA     423    318
    ## 3369      NA   NA         <NA>   1993 coopece01            BBWAA     423    318
    ## 3370      NA   NA         <NA>   1993 matthga01            BBWAA     423    318
    ## 3371      NA   NA         <NA>   1993 mcraeha01            BBWAA     423    318
    ## 3372      NA   NA         <NA>   1993 porteda02            BBWAA     423    318
    ## 3301      NA   NA         <NA>   1992 seaveto01            BBWAA     430    323
    ## 3302      NA   NA         <NA>   1992 fingero01            BBWAA     430    323
    ## 3303      NA   NA         <NA>   1992 cepedor01            BBWAA     430    323
    ## 3304      NA   NA         <NA>   1992 perezto01            BBWAA     430    323
    ## 3305      NA   NA         <NA>   1992 mazerbi01            BBWAA     430    323
    ## 3306      NA   NA         <NA>   1992 olivato01            BBWAA     430    323
    ## 3307      NA   NA         <NA>   1992 santoro01            BBWAA     430    323
    ## 3308      NA   NA         <NA>   1992  kaatji01            BBWAA     430    323
    ## 3309      NA   NA         <NA>   1992 willsma01            BBWAA     430    323
    ## 3310      NA   NA         <NA>   1992 boyerke01            BBWAA     430    323
    ## 3311      NA   NA         <NA>   1992 allendi01            BBWAA     430    323
    ## 3312      NA   NA         <NA>   1992 minosmi01            BBWAA     430    323
    ## 3313      NA   NA         <NA>   1992 torrejo01            BBWAA     430    323
    ## 3314      NA   NA         <NA>   1992 tiantlu01            BBWAA     430    323
    ## 3315      NA   NA         <NA>   1992 lolicmi01            BBWAA     430    323
    ## 3316      NA   NA         <NA>   1992 floodcu01            BBWAA     430    323
    ## 3317      NA   NA         <NA>   1992  rosepe01            BBWAA     430    323
    ## 3318      NA   NA         <NA>   1992 bondsbo01            BBWAA     430    323
    ## 3319      NA   NA         <NA>   1992 pinsova01            BBWAA     430    323
    ## 3320      NA   NA         <NA>   1992 munsoth01            BBWAA     430    323
    ## 3321      NA   NA         <NA>   1992 staubru01            BBWAA     430    323
    ## 3322      NA   NA         <NA>   1992 fostege01            BBWAA     430    323
    ## 3323      NA   NA         <NA>   1992  bluevi01            BBWAA     430    323
    ## 3324      NA   NA         <NA>   1992 grichbo01            BBWAA     430    323
    ## 3325      NA   NA         <NA>   1992 bakerdu01            BBWAA     430    323
    ## 3326      NA   NA         <NA>   1992 kingmda01            BBWAA     430    323
    ## 3327      NA   NA         <NA>   1992 russebi01            BBWAA     430    323
    ## 3328      NA   NA         <NA>   1992 cedence01            BBWAA     430    323
    ## 3329      NA   NA         <NA>   1992 yeagest01            BBWAA     430    323
    ## 3330      NA   NA         <NA>   1992 harrato01            BBWAA     430    323
    ## 3331      NA   NA         <NA>   1992 leonade01            BBWAA     430    323
    ## 3332      NA   NA         <NA>   1992 dennyjo01            BBWAA     430    323
    ## 3333      NA   NA         <NA>   1992 forscke01            BBWAA     430    323
    ## 3334      NA   NA         <NA>   1992 maddoga01            BBWAA     430    323
    ## 3335      NA   NA         <NA>   1992 oglivbe01            BBWAA     430    323
    ## 3336      NA   NA         <NA>   1992 thomago01            BBWAA     430    323
    ## 3337      NA   NA         <NA>   1992 vuckope01            BBWAA     430    323
    ## 3338      NA   NA         <NA>   1992 mcgowbi99         Veterans      NA     NA
    ## 3339      NA   NA         <NA>   1992 newhoha01         Veterans      NA     NA
    ## 3254      NA   NA         <NA>   1991 carewro01            BBWAA     443    333
    ## 3255      NA   NA         <NA>   1991 perryga01            BBWAA     443    333
    ## 3256      NA   NA         <NA>   1991 jenkife01            BBWAA     443    333
    ## 3257      NA   NA         <NA>   1991 fingero01            BBWAA     443    333
    ## 3258      NA   NA         <NA>   1991 bunniji01            BBWAA     443    333
    ## 3259      NA   NA         <NA>   1991 cepedor01            BBWAA     443    333
    ## 3260      NA   NA         <NA>   1991 olivato01            BBWAA     443    333
    ## 3261      NA   NA         <NA>   1991 mazerbi01            BBWAA     443    333
    ## 3262      NA   NA         <NA>   1991 santoro01            BBWAA     443    333
    ## 3263      NA   NA         <NA>   1991 kuennha01            BBWAA     443    333
    ## 3264      NA   NA         <NA>   1991  kaatji01            BBWAA     443    333
    ## 3265      NA   NA         <NA>   1991 willsma01            BBWAA     443    333
    ## 3266      NA   NA         <NA>   1991 allendi01            BBWAA     443    333
    ## 3267      NA   NA         <NA>   1991 boyerke01            BBWAA     443    333
    ## 3268      NA   NA         <NA>   1991 torrejo01            BBWAA     443    333
    ## 3269      NA   NA         <NA>   1991 bondsbo01            BBWAA     443    333
    ## 3270      NA   NA         <NA>   1991 minosmi01            BBWAA     443    333
    ## 3271      NA   NA         <NA>   1991 lolicmi01            BBWAA     443    333
    ## 3272      NA   NA         <NA>   1991 tiantlu01            BBWAA     443    333
    ## 3273      NA   NA         <NA>   1991 pinsova01            BBWAA     443    333
    ## 3274      NA   NA         <NA>   1991 munsoth01            BBWAA     443    333
    ## 3275      NA   NA         <NA>   1991 staubru01            BBWAA     443    333
    ## 3276      NA   NA         <NA>   1991 floodcu01            BBWAA     443    333
    ## 3277      NA   NA         <NA>   1991 oliveal01            BBWAA     443    333
    ## 3278      NA   NA         <NA>   1991  lylesp01            BBWAA     443    333
    ## 3279      NA   NA         <NA>   1991  bowala01            BBWAA     443    333
    ## 3280      NA   NA         <NA>   1991 koosmje01            BBWAA     443    333
    ## 3281      NA   NA         <NA>   1991 burroje01            BBWAA     443    333
    ## 3282      NA   NA         <NA>   1991 hargrmi01            BBWAA     443    333
    ## 3283      NA   NA         <NA>   1991 hebneri01            BBWAA     443    333
    ## 3284      NA   NA         <NA>   1991 hootobu01            BBWAA     443    333
    ## 3285      NA   NA         <NA>   1991 jorgemi01            BBWAA     443    333
    ## 3286      NA   NA         <NA>   1991 lowenjo01            BBWAA     443    333
    ## 3287      NA   NA         <NA>   1991 valenel01            BBWAA     443    333
    ## 3288      NA   NA         <NA>   1991 bailobo01            BBWAA     443    333
    ## 3289      NA   NA         <NA>   1991 bumbral01            BBWAA     443    333
    ## 3290      NA   NA         <NA>   1991 dauerri01            BBWAA     443    333
    ## 3291      NA   NA         <NA>   1991 gamblos01            BBWAA     443    333
    ## 3292      NA   NA         <NA>   1991  gurala01            BBWAA     443    333
    ## 3293      NA   NA         <NA>   1991  howear01            BBWAA     443    333
    ## 3294      NA   NA         <NA>   1991 kisonbr01            BBWAA     443    333
    ## 3295      NA   NA         <NA>   1991 rogerst01            BBWAA     443    333
    ## 3296      NA   NA         <NA>   1991 wathajo01            BBWAA     443    333
    ## 3297      NA   NA         <NA>   1991 zachrpa01            BBWAA     443    333
    ## 3298      NA   NA         <NA>   1991  zahnge01            BBWAA     443    333
    ## 3299      NA   NA         <NA>   1991 lazzeto01         Veterans      NA     NA
    ## 3300      NA   NA         <NA>   1991 veeckbi99         Veterans      NA     NA
    ## 3210      NA   NA         <NA>   1990 palmeji01            BBWAA     444    333
    ## 3211      NA   NA         <NA>   1990 morgajo02            BBWAA     444    333
    ## 3212      NA   NA         <NA>   1990 perryga01            BBWAA     444    333
    ## 3213      NA   NA         <NA>   1990 jenkife01            BBWAA     444    333
    ## 3214      NA   NA         <NA>   1990 bunniji01            BBWAA     444    333
    ## 3215      NA   NA         <NA>   1990 cepedor01            BBWAA     444    333
    ## 3216      NA   NA         <NA>   1990 olivato01            BBWAA     444    333
    ## 3217      NA   NA         <NA>   1990 mazerbi01            BBWAA     444    333
    ## 3218      NA   NA         <NA>   1990 kuennha01            BBWAA     444    333
    ## 3219      NA   NA         <NA>   1990 santoro01            BBWAA     444    333
    ## 3220      NA   NA         <NA>   1990 willsma01            BBWAA     444    333
    ## 3221      NA   NA         <NA>   1990  kaatji01            BBWAA     444    333
    ## 3222      NA   NA         <NA>   1990 boyerke01            BBWAA     444    333
    ## 3223      NA   NA         <NA>   1990 allendi01            BBWAA     444    333
    ## 3224      NA   NA         <NA>   1990 torrejo01            BBWAA     444    333
    ## 3225      NA   NA         <NA>   1990 minosmi01            BBWAA     444    333
    ## 3226      NA   NA         <NA>   1990  facero01            BBWAA     444    333
    ## 3227      NA   NA         <NA>   1990 tiantlu01            BBWAA     444    333
    ## 3228      NA   NA         <NA>   1990 pinsova01            BBWAA     444    333
    ## 3229      NA   NA         <NA>   1990 floodcu01            BBWAA     444    333
    ## 3230      NA   NA         <NA>   1990 munsoth01            BBWAA     444    333
    ## 3231      NA   NA         <NA>   1990 bondsbo01            BBWAA     444    333
    ## 3232      NA   NA         <NA>   1990 lolicmi01            BBWAA     444    333
    ## 3233      NA   NA         <NA>   1990  lylesp01            BBWAA     444    333
    ## 3234      NA   NA         <NA>   1990 mcgratu01            BBWAA     444    333
    ## 3235      NA   NA         <NA>   1990  dentbu01            BBWAA     444    333
    ## 3236      NA   NA         <NA>   1990 watsobo01            BBWAA     444    333
    ## 3237      NA   NA         <NA>   1990 mondari01            BBWAA     444    333
    ## 3238      NA   NA         <NA>   1990 pinielo01            BBWAA     444    333
    ## 3239      NA   NA         <NA>   1990 rivermi01            BBWAA     444    333
    ## 3240      NA   NA         <NA>   1990 bibbyji01            BBWAA     444    333
    ## 3241      NA   NA         <NA>   1990 luzingr01            BBWAA     444    333
    ## 3242      NA   NA         <NA>   1990  remyje01            BBWAA     444    333
    ## 3243      NA   NA         <NA>   1990 torremi01            BBWAA     444    333
    ## 3244      NA   NA         <NA>   1990 caldwmi01            BBWAA     444    333
    ## 3245      NA   NA         <NA>   1990 howelro02            BBWAA     444    333
    ## 3246      NA   NA         <NA>   1990 moraljo01            BBWAA     444    333
    ## 3247      NA   NA         <NA>   1990  otisam01            BBWAA     444    333
    ## 3248      NA   NA         <NA>   1990 scottto01            BBWAA     444    333
    ## 3249      NA   NA         <NA>   1990 singlke01            BBWAA     444    333
    ## 3250      NA   NA         <NA>   1990 splitpa01            BBWAA     444    333
    ## 3251      NA   NA         <NA>   1990 stearjo01            BBWAA     444    333
    ## 3252      NA   NA         <NA>   1990 summech01            BBWAA     444    333
    ## 3253      NA   NA         <NA>   1990 tidrodi01            BBWAA     444    333
    ## 3167      NA   NA         <NA>   1989 benchjo01            BBWAA     447    336
    ## 3168      NA   NA         <NA>   1989 yastrca01            BBWAA     447    336
    ## 3169      NA   NA         <NA>   1989 perryga01            BBWAA     447    336
    ## 3170      NA   NA         <NA>   1989 bunniji01            BBWAA     447    336
    ## 3171      NA   NA         <NA>   1989 jenkife01            BBWAA     447    336
    ## 3172      NA   NA         <NA>   1989 cepedor01            BBWAA     447    336
    ## 3173      NA   NA         <NA>   1989 olivato01            BBWAA     447    336
    ## 3174      NA   NA         <NA>   1989 mazerbi01            BBWAA     447    336
    ## 3175      NA   NA         <NA>   1989 kuennha01            BBWAA     447    336
    ## 3176      NA   NA         <NA>   1989 willsma01            BBWAA     447    336
    ## 3177      NA   NA         <NA>   1989  kaatji01            BBWAA     447    336
    ## 3178      NA   NA         <NA>   1989 santoro01            BBWAA     447    336
    ## 3179      NA   NA         <NA>   1989 boyerke01            BBWAA     447    336
    ## 3180      NA   NA         <NA>   1989 minosmi01            BBWAA     447    336
    ## 3181      NA   NA         <NA>   1989  facero01            BBWAA     447    336
    ## 3182      NA   NA         <NA>   1989 lolicmi01            BBWAA     447    336
    ## 3183      NA   NA         <NA>   1989 tiantlu01            BBWAA     447    336
    ## 3184      NA   NA         <NA>   1989 torrejo01            BBWAA     447    336
    ## 3185      NA   NA         <NA>   1989 allendi01            BBWAA     447    336
    ## 3186      NA   NA         <NA>   1989 pinsova01            BBWAA     447    336
    ## 3187      NA   NA         <NA>   1989 munsoth01            BBWAA     447    336
    ## 3188      NA   NA         <NA>   1989 bondsbo01            BBWAA     447    336
    ## 3189      NA   NA         <NA>   1989 floodcu01            BBWAA     447    336
    ## 3190      NA   NA         <NA>   1989  lylesp01            BBWAA     447    336
    ## 3191      NA   NA         <NA>   1989 campabe01            BBWAA     447    336
    ## 3192      NA   NA         <NA>   1989  woodwi01            BBWAA     447    336
    ## 3193      NA   NA         <NA>   1989  motama01            BBWAA     447    336
    ## 3194      NA   NA         <NA>   1989 murcebo01            BBWAA     447    336
    ## 3195      NA   NA         <NA>   1989 moneydo01            BBWAA     447    336
    ## 3196      NA   NA         <NA>   1989 tenacge01            BBWAA     447    336
    ## 3197      NA   NA         <NA>   1989  barrji01            BBWAA     447    336
    ## 3198      NA   NA         <NA>   1989 crowlte01            BBWAA     447    336
    ## 3199      NA   NA         <NA>   1989 fergujo01            BBWAA     447    336
    ## 3200      NA   NA         <NA>   1989 frymawo01            BBWAA     447    336
    ## 3201      NA   NA         <NA>   1989 geronce01            BBWAA     447    336
    ## 3202      NA   NA         <NA>   1989 goltzda01            BBWAA     447    336
    ## 3203      NA   NA         <NA>   1989 matlajo01            BBWAA     447    336
    ## 3204      NA   NA         <NA>   1989   mayru01            BBWAA     447    336
    ## 3205      NA   NA         <NA>   1989 mcbriba01            BBWAA     447    336
    ## 3206      NA   NA         <NA>   1989 robinbi02            BBWAA     447    336
    ## 3207      NA   NA         <NA>   1989  ziskri01            BBWAA     447    336
    ## 3208      NA   NA         <NA>   1989 barlial99         Veterans      NA     NA
    ## 3209      NA   NA         <NA>   1989 schoere01         Veterans      NA     NA
    ## 3123      NA   NA         <NA>   1988 stargwi01            BBWAA     427    321
    ## 3124      NA   NA         <NA>   1988 bunniji01            BBWAA     427    321
    ## 3125      NA   NA         <NA>   1988 olivato01            BBWAA     427    321
    ## 3126      NA   NA         <NA>   1988 cepedor01            BBWAA     427    321
    ## 3127      NA   NA         <NA>   1988 marisro01            BBWAA     427    321
    ## 3128      NA   NA         <NA>   1988 kuennha01            BBWAA     427    321
    ## 3129      NA   NA         <NA>   1988 mazerbi01            BBWAA     427    321
    ## 3130      NA   NA         <NA>   1988 tiantlu01            BBWAA     427    321
    ## 3131      NA   NA         <NA>   1988 willsma01            BBWAA     427    321
    ## 3132      NA   NA         <NA>   1988 boyerke01            BBWAA     427    321
    ## 3133      NA   NA         <NA>   1988 lolicmi01            BBWAA     427    321
    ## 3134      NA   NA         <NA>   1988 santoro01            BBWAA     427    321
    ## 3135      NA   NA         <NA>   1988 minosmi01            BBWAA     427    321
    ## 3136      NA   NA         <NA>   1988  facero01            BBWAA     427    321
    ## 3137      NA   NA         <NA>   1988 pinsova01            BBWAA     427    321
    ## 3138      NA   NA         <NA>   1988 torrejo01            BBWAA     427    321
    ## 3139      NA   NA         <NA>   1988  lylesp01            BBWAA     427    321
    ## 3140      NA   NA         <NA>   1988 howarel01            BBWAA     427    321
    ## 3141      NA   NA         <NA>   1988 allendi01            BBWAA     427    321
    ## 3142      NA   NA         <NA>   1988 floodcu01            BBWAA     427    321
    ## 3143      NA   NA         <NA>   1988 munsoth01            BBWAA     427    321
    ## 3144      NA   NA         <NA>   1988 larsedo01            BBWAA     427    321
    ## 3145      NA   NA         <NA>   1988  woodwi01            BBWAA     427    321
    ## 3146      NA   NA         <NA>   1988 bondsbo01            BBWAA     427    321
    ## 3147      NA   NA         <NA>   1988  motama01            BBWAA     427    321
    ## 3148      NA   NA         <NA>   1988 belanma01            BBWAA     427    321
    ## 3149      NA   NA         <NA>   1988   leebi03            BBWAA     427    321
    ## 3150      NA   NA         <NA>   1988   mayle01            BBWAA     427    321
    ## 3151      NA   NA         <NA>   1988 smithre06            BBWAA     427    321
    ## 3152      NA   NA         <NA>   1988 hraboal01            BBWAA     427    321
    ## 3153      NA   NA         <NA>   1988 bahnsst01            BBWAA     427    321
    ## 3154      NA   NA         <NA>   1988 grimsro02            BBWAA     427    321
    ## 3155      NA   NA         <NA>   1988 hislela01            BBWAA     427    321
    ## 3156      NA   NA         <NA>   1988 jacksgr01            BBWAA     427    321
    ## 3157      NA   NA         <NA>   1988 jonesra01            BBWAA     427    321
    ## 3158      NA   NA         <NA>   1988 maybejo01            BBWAA     427    321
    ## 3159      NA   NA         <NA>   1988 mcgloly01            BBWAA     427    321
    ## 3160      NA   NA         <NA>   1988 medicdo01            BBWAA     427    321
    ## 3161      NA   NA         <NA>   1988 milnejo01            BBWAA     427    321
    ## 3162      NA   NA         <NA>   1988 montawi01            BBWAA     427    321
    ## 3163      NA   NA         <NA>   1988  rudijo01            BBWAA     427    321
    ## 3164      NA   NA         <NA>   1988 spencji01            BBWAA     427    321
    ## 3165      NA   NA         <NA>   1988 unserde01            BBWAA     427    321
    ## 3166      NA   NA         <NA>   1988  wiseri01            BBWAA     427    321
    ## 3094      NA   NA         <NA>   1987 willibi01            BBWAA     413    310
    ## 3095      NA   NA         <NA>   1987 hunteca01            BBWAA     413    310
    ## 3096      NA   NA         <NA>   1987 bunniji01            BBWAA     413    310
    ## 3097      NA   NA         <NA>   1987 cepedor01            BBWAA     413    310
    ## 3098      NA   NA         <NA>   1987 marisro01            BBWAA     413    310
    ## 3099      NA   NA         <NA>   1987 olivato01            BBWAA     413    310
    ## 3100      NA   NA         <NA>   1987 kuennha01            BBWAA     413    310
    ## 3101      NA   NA         <NA>   1987 mazerbi01            BBWAA     413    310
    ## 3102      NA   NA         <NA>   1987 willsma01            BBWAA     413    310
    ## 3103      NA   NA         <NA>   1987 boyerke01            BBWAA     413    310
    ## 3104      NA   NA         <NA>   1987 burdele01            BBWAA     413    310
    ## 3105      NA   NA         <NA>   1987 lolicmi01            BBWAA     413    310
    ## 3106      NA   NA         <NA>   1987 minosmi01            BBWAA     413    310
    ## 3107      NA   NA         <NA>   1987  facero01            BBWAA     413    310
    ## 3108      NA   NA         <NA>   1987 santoro01            BBWAA     413    310
    ## 3109      NA   NA         <NA>   1987 allendi01            BBWAA     413    310
    ## 3110      NA   NA         <NA>   1987 floodcu01            BBWAA     413    310
    ## 3111      NA   NA         <NA>   1987 pinsova01            BBWAA     413    310
    ## 3112      NA   NA         <NA>   1987 torrejo01            BBWAA     413    310
    ## 3113      NA   NA         <NA>   1987 howarel01            BBWAA     413    310
    ## 3114      NA   NA         <NA>   1987 larsedo01            BBWAA     413    310
    ## 3115      NA   NA         <NA>   1987 munsoth01            BBWAA     413    310
    ## 3116      NA   NA         <NA>   1987  woodwi01            BBWAA     413    310
    ## 3117      NA   NA         <NA>   1987 bondsbo01            BBWAA     413    310
    ## 3118      NA   NA         <NA>   1987 marshmi01            BBWAA     413    310
    ## 3119      NA   NA         <NA>   1987 bandosa01            BBWAA     413    310
    ## 3120      NA   NA         <NA>   1987 groteje01            BBWAA     413    310
    ## 3121      NA   NA         <NA>   1987 stonest01            BBWAA     413    310
    ## 3122      NA   NA         <NA>   1987 dandrra99         Veterans      NA     NA
    ## 3051      NA   NA         <NA>   1986 mccovwi01            BBWAA     425    319
    ## 3052      NA   NA         <NA>   1986 willibi01            BBWAA     425    319
    ## 3053      NA   NA         <NA>   1986 hunteca01            BBWAA     425    319
    ## 3054      NA   NA         <NA>   1986 bunniji01            BBWAA     425    319
    ## 3055      NA   NA         <NA>   1986 marisro01            BBWAA     425    319
    ## 3056      NA   NA         <NA>   1986 olivato01            BBWAA     425    319
    ## 3057      NA   NA         <NA>   1986 cepedor01            BBWAA     425    319
    ## 3058      NA   NA         <NA>   1986 kuennha01            BBWAA     425    319
    ## 3059      NA   NA         <NA>   1986 willsma01            BBWAA     425    319
    ## 3060      NA   NA         <NA>   1986 mazerbi01            BBWAA     425    319
    ## 3061      NA   NA         <NA>   1986 burdele01            BBWAA     425    319
    ## 3062      NA   NA         <NA>   1986 boyerke01            BBWAA     425    319
    ## 3063      NA   NA         <NA>   1986 minosmi01            BBWAA     425    319
    ## 3064      NA   NA         <NA>   1986 lolicmi01            BBWAA     425    319
    ## 3065      NA   NA         <NA>   1986  facero01            BBWAA     425    319
    ## 3066      NA   NA         <NA>   1986 santoro01            BBWAA     425    319
    ## 3067      NA   NA         <NA>   1986 torrejo01            BBWAA     425    319
    ## 3068      NA   NA         <NA>   1986 howarel01            BBWAA     425    319
    ## 3069      NA   NA         <NA>   1986 floodcu01            BBWAA     425    319
    ## 3070      NA   NA         <NA>   1986 pinsova01            BBWAA     425    319
    ## 3071      NA   NA         <NA>   1986 allendi01            BBWAA     425    319
    ## 3072      NA   NA         <NA>   1986 munsoth01            BBWAA     425    319
    ## 3073      NA   NA         <NA>   1986 larsedo01            BBWAA     425    319
    ## 3074      NA   NA         <NA>   1986  woodwi01            BBWAA     425    319
    ## 3075      NA   NA         <NA>   1986 mccarti01            BBWAA     425    319
    ## 3076      NA   NA         <NA>   1986 mcnalda01            BBWAA     425    319
    ## 3077      NA   NA         <NA>   1986 hillejo01            BBWAA     425    319
    ## 3078      NA   NA         <NA>   1986 blairpa01            BBWAA     425    319
    ## 3079      NA   NA         <NA>   1986 richajr01            BBWAA     425    319
    ## 3080      NA   NA         <NA>   1986 holtzke01            BBWAA     425    319
    ## 3081      NA   NA         <NA>   1986 hortowi01            BBWAA     425    319
    ## 3082      NA   NA         <NA>   1986 lonboji01            BBWAA     425    319
    ## 3083      NA   NA         <NA>   1986 messean01            BBWAA     425    319
    ## 3084      NA   NA         <NA>   1986  cashda01            BBWAA     425    319
    ## 3085      NA   NA         <NA>   1986 sanguma01            BBWAA     425    319
    ## 3086      NA   NA         <NA>   1986 billija01            BBWAA     425    319
    ## 3087      NA   NA         <NA>   1986 cardejo02            BBWAA     425    319
    ## 3088      NA   NA         <NA>   1986 harrebu01            BBWAA     425    319
    ## 3089      NA   NA         <NA>   1986 scottge02            BBWAA     425    319
    ## 3090      NA   NA         <NA>   1986 davalvi01            BBWAA     425    319
    ## 3091      NA   NA         <NA>   1986 knowlda01            BBWAA     425    319
    ## 3092      NA   NA         <NA>   1986 doerrbo01         Veterans      NA     NA
    ## 3093      NA   NA         <NA>   1986 lombaer01         Veterans      NA     NA
    ## 3008      NA   NA         <NA>   1985 wilheho01            BBWAA     395    297
    ## 3009      NA   NA         <NA>   1985 brocklo01            BBWAA     395    297
    ## 3010      NA   NA         <NA>   1985   foxne01            BBWAA     395    297
    ## 3011      NA   NA         <NA>   1985 willibi01            BBWAA     395    297
    ## 3012      NA   NA         <NA>   1985 bunniji01            BBWAA     395    297
    ## 3013      NA   NA         <NA>   1985 hunteca01            BBWAA     395    297
    ## 3014      NA   NA         <NA>   1985 marisro01            BBWAA     395    297
    ## 3015      NA   NA         <NA>   1985 kuennha01            BBWAA     395    297
    ## 3016      NA   NA         <NA>   1985 cepedor01            BBWAA     395    297
    ## 3017      NA   NA         <NA>   1985 olivato01            BBWAA     395    297
    ## 3018      NA   NA         <NA>   1985 willsma01            BBWAA     395    297
    ## 3019      NA   NA         <NA>   1985 mazerbi01            BBWAA     395    297
    ## 3020      NA   NA         <NA>   1985 burdele01            BBWAA     395    297
    ## 3021      NA   NA         <NA>   1985 lolicmi01            BBWAA     395    297
    ## 3022      NA   NA         <NA>   1985 boyerke01            BBWAA     395    297
    ## 3023      NA   NA         <NA>   1985  facero01            BBWAA     395    297
    ## 3024      NA   NA         <NA>   1985 howarel01            BBWAA     395    297
    ## 3025      NA   NA         <NA>   1985 santoro01            BBWAA     395    297
    ## 3026      NA   NA         <NA>   1985 torrejo01            BBWAA     395    297
    ## 3027      NA   NA         <NA>   1985 larsedo01            BBWAA     395    297
    ## 3028      NA   NA         <NA>   1985 munsoth01            BBWAA     395    297
    ## 3029      NA   NA         <NA>   1985 allendi01            BBWAA     395    297
    ## 3030      NA   NA         <NA>   1985 floodcu01            BBWAA     395    297
    ## 3031      NA   NA         <NA>   1985 pinsova01            BBWAA     395    297
    ## 3032      NA   NA         <NA>   1985  woodwi01            BBWAA     395    297
    ## 3033      NA   NA         <NA>   1985 haddiha01            BBWAA     395    297
    ## 3034      NA   NA         <NA>   1985 mcnalda01            BBWAA     395    297
    ## 3035      NA   NA         <NA>   1985 holtzke01            BBWAA     395    297
    ## 3036      NA   NA         <NA>   1985 fairlro01            BBWAA     395    297
    ## 3037      NA   NA         <NA>   1985 lonboji01            BBWAA     395    297
    ## 3038      NA   NA         <NA>   1985 messean01            BBWAA     395    297
    ## 3039      NA   NA         <NA>   1985 kessido01            BBWAA     395    297
    ## 3040      NA   NA         <NA>   1985 mclaide01            BBWAA     395    297
    ## 3041      NA   NA         <NA>   1985  alouje01            BBWAA     395    297
    ## 3042      NA   NA         <NA>   1985 cartyri01            BBWAA     395    297
    ## 3043      NA   NA         <NA>   1985 ellisdo01            BBWAA     395    297
    ## 3044      NA   NA         <NA>   1985 carrocl02            BBWAA     395    297
    ## 3045      NA   NA         <NA>   1985 kraneed01            BBWAA     395    297
    ## 3046      NA   NA         <NA>   1985 scottge02            BBWAA     395    297
    ## 3047      NA   NA         <NA>   1985 tolanbo01            BBWAA     395    297
    ## 3048      NA   NA         <NA>   1985 whitero01            BBWAA     395    297
    ## 3049      NA   NA         <NA>   1985 slaugen01         Veterans      NA     NA
    ## 3050      NA   NA         <NA>   1985 vaughar01         Veterans      NA     NA
    ## 2977      NA   NA         <NA>   1984 aparilu01            BBWAA     403    303
    ## 2978      NA   NA         <NA>   1984 killeha01            BBWAA     403    303
    ## 2979      NA   NA         <NA>   1984 drysddo01            BBWAA     403    303
    ## 2980      NA   NA         <NA>   1984 wilheho01            BBWAA     403    303
    ## 2981      NA   NA         <NA>   1984   foxne01            BBWAA     403    303
    ## 2982      NA   NA         <NA>   1984 willibi01            BBWAA     403    303
    ## 2983      NA   NA         <NA>   1984 bunniji01            BBWAA     403    303
    ## 2984      NA   NA         <NA>   1984 cepedor01            BBWAA     403    303
    ## 2985      NA   NA         <NA>   1984 olivato01            BBWAA     403    303
    ## 2986      NA   NA         <NA>   1984 marisro01            BBWAA     403    303
    ## 2987      NA   NA         <NA>   1984 kuennha01            BBWAA     403    303
    ## 2988      NA   NA         <NA>   1984 willsma01            BBWAA     403    303
    ## 2989      NA   NA         <NA>   1984 burdele01            BBWAA     403    303
    ## 2990      NA   NA         <NA>   1984 mazerbi01            BBWAA     403    303
    ## 2991      NA   NA         <NA>   1984  facero01            BBWAA     403    303
    ## 2992      NA   NA         <NA>   1984 howarel01            BBWAA     403    303
    ## 2993      NA   NA         <NA>   1984 torrejo01            BBWAA     403    303
    ## 2994      NA   NA         <NA>   1984 munsoth01            BBWAA     403    303
    ## 2995      NA   NA         <NA>   1984 larsedo01            BBWAA     403    303
    ## 2996      NA   NA         <NA>   1984  woodwi01            BBWAA     403    303
    ## 2997      NA   NA         <NA>   1984 fregoji01            BBWAA     403    303
    ## 2998      NA   NA         <NA>   1984 boutoji01            BBWAA     403    303
    ## 2999      NA   NA         <NA>   1984 johnsda02            BBWAA     403    303
    ## 3000      NA   NA         <NA>   1984 stanlmi01            BBWAA     403    303
    ## 3001      NA   NA         <NA>   1984 bailebo01            BBWAA     403    303
    ## 3002      NA   NA         <NA>   1984 brilene01            BBWAA     403    303
    ## 3003      NA   NA         <NA>   1984 carrocl02            BBWAA     403    303
    ## 3004      NA   NA         <NA>   1984 colboji01            BBWAA     403    303
    ## 3005      NA   NA         <NA>   1984 fairlro01            BBWAA     403    303
    ## 3006      NA   NA         <NA>   1984 ferreri01         Veterans      NA     NA
    ## 3007      NA   NA         <NA>   1984 reesepe01         Veterans      NA     NA
    ## 2929      NA   NA         <NA>   1983 robinbr01            BBWAA     374    281
    ## 2930      NA   NA         <NA>   1983 maricju01            BBWAA     374    281
    ## 2931      NA   NA         <NA>   1983 killeha01            BBWAA     374    281
    ## 2932      NA   NA         <NA>   1983 aparilu01            BBWAA     374    281
    ## 2933      NA   NA         <NA>   1983 wilheho01            BBWAA     374    281
    ## 2934      NA   NA         <NA>   1983 drysddo01            BBWAA     374    281
    ## 2935      NA   NA         <NA>   1983 hodgegi01            BBWAA     374    281
    ## 2936      NA   NA         <NA>   1983   foxne01            BBWAA     374    281
    ## 2937      NA   NA         <NA>   1983 willibi01            BBWAA     374    281
    ## 2938      NA   NA         <NA>   1983 schoere01            BBWAA     374    281
    ## 2939      NA   NA         <NA>   1983 bunniji01            BBWAA     374    281
    ## 2940      NA   NA         <NA>   1983 kuennha01            BBWAA     374    281
    ## 2941      NA   NA         <NA>   1983 willsma01            BBWAA     374    281
    ## 2942      NA   NA         <NA>   1983 olivato01            BBWAA     374    281
    ## 2943      NA   NA         <NA>   1983 marisro01            BBWAA     374    281
    ## 2944      NA   NA         <NA>   1983 cepedor01            BBWAA     374    281
    ## 2945      NA   NA         <NA>   1983 mazerbi01            BBWAA     374    281
    ## 2946      NA   NA         <NA>   1983 burdele01            BBWAA     374    281
    ## 2947      NA   NA         <NA>   1983  facero01            BBWAA     374    281
    ## 2948      NA   NA         <NA>   1983 howarel01            BBWAA     374    281
    ## 2949      NA   NA         <NA>   1983 larsedo01            BBWAA     374    281
    ## 2950      NA   NA         <NA>   1983 torrejo01            BBWAA     374    281
    ## 2951      NA   NA         <NA>   1983 munsoth01            BBWAA     374    281
    ## 2952      NA   NA         <NA>   1983 allendi01            BBWAA     374    281
    ## 2953      NA   NA         <NA>   1983 pinsova01            BBWAA     374    281
    ## 2954      NA   NA         <NA>   1983 perryji01            BBWAA     374    281
    ## 2955      NA   NA         <NA>   1983 powelbo01            BBWAA     374    281
    ## 2956      NA   NA         <NA>   1983 sadecra01            BBWAA     374    281
    ## 2957      NA   NA         <NA>   1983 giustda01            BBWAA     374    281
    ## 2958      NA   NA         <NA>   1983 helmsto01            BBWAA     374    281
    ## 2959      NA   NA         <NA>   1983 millafe01            BBWAA     374    281
    ## 2960      NA   NA         <NA>   1983 cuellmi01            BBWAA     374    281
    ## 2961      NA   NA         <NA>   1983 dierkla01            BBWAA     374    281
    ## 2962      NA   NA         <NA>   1983 dobsopa01            BBWAA     374    281
    ## 2963      NA   NA         <NA>   1983 downial01            BBWAA     374    281
    ## 2964      NA   NA         <NA>   1983 hoernjo01            BBWAA     374    281
    ## 2965      NA   NA         <NA>   1983 hundlra01            BBWAA     374    281
    ## 2966      NA   NA         <NA>   1983   mayca01            BBWAA     374    281
    ## 2967      NA   NA         <NA>   1983 mcmulke01            BBWAA     374    281
    ## 2968      NA   NA         <NA>   1983 meltobi01            BBWAA     374    281
    ## 2969      NA   NA         <NA>   1983 nolanga01            BBWAA     374    281
    ## 2970      NA   NA         <NA>   1983 raderdo02            BBWAA     374    281
    ## 2971      NA   NA         <NA>   1983 rojasco01            BBWAA     374    281
    ## 2972      NA   NA         <NA>   1983 seguidi01            BBWAA     374    281
    ## 2973      NA   NA         <NA>   1983 singebi01            BBWAA     374    281
    ## 2974      NA   NA         <NA>   1983  wynnji01            BBWAA     374    281
    ## 2975      NA   NA         <NA>   1983 alstowa01         Veterans      NA     NA
    ## 2976      NA   NA         <NA>   1983  kellge01         Veterans      NA     NA
    ## 2885      NA   NA         <NA>   1982 aaronha01            BBWAA     415    312
    ## 2886      NA   NA         <NA>   1982 robinfr02            BBWAA     415    312
    ## 2887      NA   NA         <NA>   1982 maricju01            BBWAA     415    312
    ## 2888      NA   NA         <NA>   1982 killeha01            BBWAA     415    312
    ## 2889      NA   NA         <NA>   1982 wilheho01            BBWAA     415    312
    ## 2890      NA   NA         <NA>   1982 drysddo01            BBWAA     415    312
    ## 2891      NA   NA         <NA>   1982 hodgegi01            BBWAA     415    312
    ## 2892      NA   NA         <NA>   1982 aparilu01            BBWAA     415    312
    ## 2893      NA   NA         <NA>   1982 bunniji01            BBWAA     415    312
    ## 2894      NA   NA         <NA>   1982 schoere01            BBWAA     415    312
    ## 2895      NA   NA         <NA>   1982   foxne01            BBWAA     415    312
    ## 2896      NA   NA         <NA>   1982 ashburi01            BBWAA     415    312
    ## 2897      NA   NA         <NA>   1982 willibi01            BBWAA     415    312
    ## 2898      NA   NA         <NA>   1982 willsma01            BBWAA     415    312
    ## 2899      NA   NA         <NA>   1982 marisro01            BBWAA     415    312
    ## 2900      NA   NA         <NA>   1982 olivato01            BBWAA     415    312
    ## 2901      NA   NA         <NA>   1982 kuennha01            BBWAA     415    312
    ## 2902      NA   NA         <NA>   1982 burdele01            BBWAA     415    312
    ## 2903      NA   NA         <NA>   1982 cepedor01            BBWAA     415    312
    ## 2904      NA   NA         <NA>   1982 howarel01            BBWAA     415    312
    ## 2905      NA   NA         <NA>   1982 larsedo01            BBWAA     415    312
    ## 2906      NA   NA         <NA>   1982 mazerbi01            BBWAA     415    312
    ## 2907      NA   NA         <NA>   1982 munsoth01            BBWAA     415    312
    ## 2908      NA   NA         <NA>   1982  facero01            BBWAA     415    312
    ## 2909      NA   NA         <NA>   1982 pinsova01            BBWAA     415    312
    ## 2910      NA   NA         <NA>   1982 davisto02            BBWAA     415    312
    ## 2911      NA   NA         <NA>   1982 mcnalda01            BBWAA     415    312
    ## 2912      NA   NA         <NA>   1982 mcdanli01            BBWAA     415    312
    ## 2913      NA   NA         <NA>   1982 petrori01            BBWAA     415    312
    ## 2914      NA   NA         <NA>   1982 breweji01            BBWAA     415    312
    ## 2915      NA   NA         <NA>   1982 freehbi01            BBWAA     415    312
    ## 2916      NA   NA         <NA>   1982 cardele01            BBWAA     415    312
    ## 2917      NA   NA         <NA>   1982 osteecl01            BBWAA     415    312
    ## 2918      NA   NA         <NA>   1982 brownga01            BBWAA     415    312
    ## 2919      NA   NA         <NA>   1982 harpeto01            BBWAA     415    312
    ## 2920      NA   NA         <NA>   1982 johnsal01            BBWAA     415    312
    ## 2921      NA   NA         <NA>   1982 johnsde01            BBWAA     415    312
    ## 2922      NA   NA         <NA>   1982 jonescl01            BBWAA     415    312
    ## 2923      NA   NA         <NA>   1982 northji01            BBWAA     415    312
    ## 2924      NA   NA         <NA>   1982 siebeso01            BBWAA     415    312
    ## 2925      NA   NA         <NA>   1982 tayloto02            BBWAA     415    312
    ## 2926      NA   NA         <NA>   1982 tovarce01            BBWAA     415    312
    ## 2927      NA   NA         <NA>   1982 chandha99         Veterans      NA     NA
    ## 2928      NA   NA         <NA>   1982 jackstr01         Veterans      NA     NA
    ## 2844      NA   NA         <NA>   1981 gibsobo01            BBWAA     401    301
    ## 2845      NA   NA         <NA>   1981 drysddo01            BBWAA     401    301
    ## 2846      NA   NA         <NA>   1981 hodgegi01            BBWAA     401    301
    ## 2847      NA   NA         <NA>   1981 killeha01            BBWAA     401    301
    ## 2848      NA   NA         <NA>   1981 wilheho01            BBWAA     401    301
    ## 2849      NA   NA         <NA>   1981 maricju01            BBWAA     401    301
    ## 2850      NA   NA         <NA>   1981   foxne01            BBWAA     401    301
    ## 2851      NA   NA         <NA>   1981 schoere01            BBWAA     401    301
    ## 2852      NA   NA         <NA>   1981 bunniji01            BBWAA     401    301
    ## 2853      NA   NA         <NA>   1981 willsma01            BBWAA     401    301
    ## 2854      NA   NA         <NA>   1981 ashburi01            BBWAA     401    301
    ## 2855      NA   NA         <NA>   1981 marisro01            BBWAA     401    301
    ## 2856      NA   NA         <NA>   1981 kuennha01            BBWAA     401    301
    ## 2857      NA   NA         <NA>   1981 howarel01            BBWAA     401    301
    ## 2858      NA   NA         <NA>   1981 cepedor01            BBWAA     401    301
    ## 2859      NA   NA         <NA>   1981 munsoth01            BBWAA     401    301
    ## 2860      NA   NA         <NA>   1981 kluszte01            BBWAA     401    301
    ## 2861      NA   NA         <NA>   1981 aparilu01            BBWAA     401    301
    ## 2862      NA   NA         <NA>   1981 burdele01            BBWAA     401    301
    ## 2863      NA   NA         <NA>   1981 mazerbi01            BBWAA     401    301
    ## 2864      NA   NA         <NA>   1981 larsedo01            BBWAA     401    301
    ## 2865      NA   NA         <NA>   1981  facero01            BBWAA     401    301
    ## 2866      NA   NA         <NA>   1981 pinsova01            BBWAA     401    301
    ## 2867      NA   NA         <NA>   1981 perryji01            BBWAA     401    301
    ## 2868      NA   NA         <NA>   1981 mcnalda01            BBWAA     401    301
    ## 2869      NA   NA         <NA>   1981 osteecl01            BBWAA     401    301
    ## 2870      NA   NA         <NA>   1981 beckegl01            BBWAA     401    301
    ## 2871      NA   NA         <NA>   1981 brownga01            BBWAA     401    301
    ## 2872      NA   NA         <NA>   1981 cardele01            BBWAA     401    301
    ## 2873      NA   NA         <NA>   1981 mcdanli01            BBWAA     401    301
    ## 2874      NA   NA         <NA>   1981 northji01            BBWAA     401    301
    ## 2875      NA   NA         <NA>   1981 siebeso01            BBWAA     401    301
    ## 2876      NA   NA         <NA>   1981 berryke01            BBWAA     401    301
    ## 2877      NA   NA         <NA>   1981 briggjo02            BBWAA     401    301
    ## 2878      NA   NA         <NA>   1981 handsbi01            BBWAA     401    301
    ## 2879      NA   NA         <NA>   1981 lockebo01            BBWAA     401    301
    ## 2880      NA   NA         <NA>   1981 maxvida01            BBWAA     401    301
    ## 2881      NA   NA         <NA>   1981 mcauldi01            BBWAA     401    301
    ## 2882      NA   NA         <NA>   1981 mcdowsa01            BBWAA     401    301
    ## 2883      NA   NA         <NA>   1981 fosteru99         Veterans      NA     NA
    ## 2884      NA   NA         <NA>   1981  mizejo01         Veterans      NA     NA
    ## 2781      NA   NA         <NA>   1980 kalinal01            BBWAA     385    289
    ## 2782      NA   NA         <NA>   1980 snidedu01            BBWAA     385    289
    ## 2783      NA   NA         <NA>   1980 drysddo01            BBWAA     385    289
    ## 2784      NA   NA         <NA>   1980 hodgegi01            BBWAA     385    289
    ## 2785      NA   NA         <NA>   1980 wilheho01            BBWAA     385    289
    ## 2786      NA   NA         <NA>   1980 bunniji01            BBWAA     385    289
    ## 2787      NA   NA         <NA>   1980 schoere01            BBWAA     385    289
    ## 2788      NA   NA         <NA>   1980   foxne01            BBWAA     385    289
    ## 2789      NA   NA         <NA>   1980 willsma01            BBWAA     385    289
    ## 2790      NA   NA         <NA>   1980 ashburi01            BBWAA     385    289
    ## 2791      NA   NA         <NA>   1980 aparilu01            BBWAA     385    289
    ## 2792      NA   NA         <NA>   1980 marisro01            BBWAA     385    289
    ## 2793      NA   NA         <NA>   1980 vernomi01            BBWAA     385    289
    ## 2794      NA   NA         <NA>   1980 kuennha01            BBWAA     385    289
    ## 2795      NA   NA         <NA>   1980 burdele01            BBWAA     385    289
    ## 2796      NA   NA         <NA>   1980 newcodo01            BBWAA     385    289
    ## 2797      NA   NA         <NA>   1980 kluszte01            BBWAA     385    289
    ## 2798      NA   NA         <NA>   1980 cepedor01            BBWAA     385    289
    ## 2799      NA   NA         <NA>   1980  darkal01            BBWAA     385    289
    ## 2800      NA   NA         <NA>   1980 mazerbi01            BBWAA     385    289
    ## 2801      NA   NA         <NA>   1980 larsedo01            BBWAA     385    289
    ## 2802      NA   NA         <NA>   1980 howarel01            BBWAA     385    289
    ## 2803      NA   NA         <NA>   1980  facero01            BBWAA     385    289
    ## 2804      NA   NA         <NA>   1980 santoro01            BBWAA     385    289
    ## 2805      NA   NA         <NA>   1980  cashno01            BBWAA     385    289
    ## 2806      NA   NA         <NA>   1980  alouma01            BBWAA     385    289
    ## 2807      NA   NA         <NA>   1980  aloufe01            BBWAA     385    289
    ## 2808      NA   NA         <NA>   1980 stottme01            BBWAA     385    289
    ## 2809      NA   NA         <NA>   1980 blassst01            BBWAA     385    289
    ## 2810      NA   NA         <NA>   1980 hickmji02            BBWAA     385    289
    ## 2811      NA   NA         <NA>   1980 jacksso01            BBWAA     385    289
    ## 2812      NA   NA         <NA>   1980 mcmahdo02            BBWAA     385    289
    ## 2813      NA   NA         <NA>   1980  akerja01            BBWAA     385    289
    ## 2814      NA   NA         <NA>   1980 barbest01            BBWAA     385    289
    ## 2815      NA   NA         <NA>   1980 bartobo01            BBWAA     385    289
    ## 2816      NA   NA         <NA>   1980 boccajo01            BBWAA     385    289
    ## 2817      NA   NA         <NA>   1980 brownla01            BBWAA     385    289
    ## 2818      NA   NA         <NA>   1980 cannich01            BBWAA     385    289
    ## 2819      NA   NA         <NA>   1980 casanpa01            BBWAA     385    289
    ## 2820      NA   NA         <NA>   1980 clarkho01            BBWAA     385    289
    ## 2821      NA   NA         <NA>   1980 edwarjo01            BBWAA     385    289
    ## 2822      NA   NA         <NA>   1980 gagliph01            BBWAA     385    289
    ## 2823      NA   NA         <NA>   1980 gosgeji01            BBWAA     385    289
    ## 2824      NA   NA         <NA>   1980  huntro01            BBWAA     385    289
    ## 2825      NA   NA         <NA>   1980 kennejo03            BBWAA     385    289
    ## 2826      NA   NA         <NA>   1980 koscoan01            BBWAA     385    289
    ## 2827      NA   NA         <NA>   1980 krausle02            BBWAA     385    289
    ## 2828      NA   NA         <NA>   1980 linzyfr01            BBWAA     385    289
    ## 2829      NA   NA         <NA>   1980 menkede01            BBWAA     385    289
    ## 2830      NA   NA         <NA>   1980 millebo04            BBWAA     385    289
    ## 2831      NA   NA         <NA>   1980 milleno01            BBWAA     385    289
    ## 2832      NA   NA         <NA>   1980 murreiv01            BBWAA     385    289
    ## 2833      NA   NA         <NA>   1980 pizarju01            BBWAA     385    289
    ## 2834      NA   NA         <NA>   1980   rayji01            BBWAA     385    289
    ## 2835      NA   NA         <NA>   1980 reichri01            BBWAA     385    289
    ## 2836      NA   NA         <NA>   1980 richepe01            BBWAA     385    289
    ## 2837      NA   NA         <NA>   1980  ryanmi02            BBWAA     385    289
    ## 2838      NA   NA         <NA>   1980 schaapa01            BBWAA     385    289
    ## 2839      NA   NA         <NA>   1980 selmadi01            BBWAA     385    289
    ## 2840      NA   NA         <NA>   1980  simsdu01            BBWAA     385    289
    ## 2841      NA   NA         <NA>   1980 vealebo01            BBWAA     385    289
    ## 2842      NA   NA         <NA>   1980 kleinch01         Veterans      NA     NA
    ## 2843      NA   NA         <NA>   1980 yawketo99         Veterans      NA     NA
    ## 2725      NA   NA         <NA>   1979  mayswi01            BBWAA     432    324
    ## 2726      NA   NA         <NA>   1979 snidedu01            BBWAA     432    324
    ## 2727      NA   NA         <NA>   1979 slaugen01            BBWAA     432    324
    ## 2728      NA   NA         <NA>   1979 hodgegi01            BBWAA     432    324
    ## 2729      NA   NA         <NA>   1979 drysddo01            BBWAA     432    324
    ## 2730      NA   NA         <NA>   1979   foxne01            BBWAA     432    324
    ## 2731      NA   NA         <NA>   1979 wilheho01            BBWAA     432    324
    ## 2732      NA   NA         <NA>   1979 willsma01            BBWAA     432    324
    ## 2733      NA   NA         <NA>   1979 schoere01            BBWAA     432    324
    ## 2734      NA   NA         <NA>   1979 bunniji01            BBWAA     432    324
    ## 2735      NA   NA         <NA>   1979 ashburi01            BBWAA     432    324
    ## 2736      NA   NA         <NA>   1979 marisro01            BBWAA     432    324
    ## 2737      NA   NA         <NA>   1979 aparilu01            BBWAA     432    324
    ## 2738      NA   NA         <NA>   1979 vernomi01            BBWAA     432    324
    ## 2739      NA   NA         <NA>   1979  darkal01            BBWAA     432    324
    ## 2740      NA   NA         <NA>   1979 kuennha01            BBWAA     432    324
    ## 2741      NA   NA         <NA>   1979 kluszte01            BBWAA     432    324
    ## 2742      NA   NA         <NA>   1979 burdele01            BBWAA     432    324
    ## 2743      NA   NA         <NA>   1979 larsedo01            BBWAA     432    324
    ## 2744      NA   NA         <NA>   1979 newcodo01            BBWAA     432    324
    ## 2745      NA   NA         <NA>   1979 mazerbi01            BBWAA     432    324
    ## 2746      NA   NA         <NA>   1979  facero01            BBWAA     432    324
    ## 2747      NA   NA         <NA>   1979 howarel01            BBWAA     432    324
    ## 2748      NA   NA         <NA>   1979 boyerke01            BBWAA     432    324
    ## 2749      NA   NA         <NA>   1979 floodcu01            BBWAA     432    324
    ## 2750      NA   NA         <NA>   1979 thomsbo01            BBWAA     432    324
    ## 2751      NA   NA         <NA>   1979 crandde01            BBWAA     432    324
    ## 2752      NA   NA         <NA>   1979   lawve01            BBWAA     432    324
    ## 2753      NA   NA         <NA>   1979 haddiha01            BBWAA     432    324
    ## 2754      NA   NA         <NA>   1979 howarfr01            BBWAA     432    324
    ## 2755      NA   NA         <NA>   1979 perraro01            BBWAA     432    324
    ## 2756      NA   NA         <NA>   1979 pappami01            BBWAA     432    324
    ## 2757      NA   NA         <NA>   1979 boyercl02            BBWAA     432    324
    ## 2758      NA   NA         <NA>   1979 mclaide01            BBWAA     432    324
    ## 2759      NA   NA         <NA>   1979 malonji01            BBWAA     432    324
    ## 2760      NA   NA         <NA>   1979 callijo01            BBWAA     432    324
    ## 2761      NA   NA         <NA>   1979 lanieha01            BBWAA     432    324
    ## 2762      NA   NA         <NA>   1979 shortch02            BBWAA     432    324
    ## 2763      NA   NA         <NA>   1979  ageeto01            BBWAA     432    324
    ## 2764      NA   NA         <NA>   1979 allenbe01            BBWAA     432    324
    ## 2765      NA   NA         <NA>   1979 alleyge01            BBWAA     432    324
    ## 2766      NA   NA         <NA>   1979 beaucji01            BBWAA     432    324
    ## 2767      NA   NA         <NA>   1979 bolinbo01            BBWAA     432    324
    ## 2768      NA   NA         <NA>   1979  culpra01            BBWAA     432    324
    ## 2769      NA   NA         <NA>   1979 fisheed02            BBWAA     432    324
    ## 2770      NA   NA         <NA>   1979 gladdfr01            BBWAA     432    324
    ## 2771      NA   NA         <NA>   1979   mayje01            BBWAA     432    324
    ## 2772      NA   NA         <NA>   1979 paganjo01            BBWAA     432    324
    ## 2773      NA   NA         <NA>   1979 pepitjo01            BBWAA     432    324
    ## 2774      NA   NA         <NA>   1979 reeseri01            BBWAA     432    324
    ## 2775      NA   NA         <NA>   1979 stahlla01            BBWAA     432    324
    ## 2776      NA   NA         <NA>   1979 stephjo03            BBWAA     432    324
    ## 2777      NA   NA         <NA>   1979 stewaji01            BBWAA     432    324
    ## 2778      NA   NA         <NA>   1979 torboje01            BBWAA     432    324
    ## 2779      NA   NA         <NA>   1979 gileswa99         Veterans      NA     NA
    ## 2780      NA   NA         <NA>   1979 wilsoha01         Veterans      NA     NA
    ## 2686      NA   NA         <NA>   1978 matheed01            BBWAA     379    285
    ## 2687      NA   NA         <NA>   1978 slaugen01            BBWAA     379    285
    ## 2688      NA   NA         <NA>   1978 snidedu01            BBWAA     379    285
    ## 2689      NA   NA         <NA>   1978 hodgegi01            BBWAA     379    285
    ## 2690      NA   NA         <NA>   1978 drysddo01            BBWAA     379    285
    ## 2691      NA   NA         <NA>   1978 bunniji01            BBWAA     379    285
    ## 2692      NA   NA         <NA>   1978 reesepe01            BBWAA     379    285
    ## 2693      NA   NA         <NA>   1978 ashburi01            BBWAA     379    285
    ## 2694      NA   NA         <NA>   1978 wilheho01            BBWAA     379    285
    ## 2695      NA   NA         <NA>   1978   foxne01            BBWAA     379    285
    ## 2696      NA   NA         <NA>   1978 schoere01            BBWAA     379    285
    ## 2697      NA   NA         <NA>   1978 willsma01            BBWAA     379    285
    ## 2698      NA   NA         <NA>   1978 marisro01            BBWAA     379    285
    ## 2699      NA   NA         <NA>   1978 burdele01            BBWAA     379    285
    ## 2700      NA   NA         <NA>   1978 vernomi01            BBWAA     379    285
    ## 2701      NA   NA         <NA>   1978  darkal01            BBWAA     379    285
    ## 2702      NA   NA         <NA>   1978 kuennha01            BBWAA     379    285
    ## 2703      NA   NA         <NA>   1978 kluszte01            BBWAA     379    285
    ## 2704      NA   NA         <NA>   1978 newcodo01            BBWAA     379    285
    ## 2705      NA   NA         <NA>   1978 howarel01            BBWAA     379    285
    ## 2706      NA   NA         <NA>   1978 larsedo01            BBWAA     379    285
    ## 2707      NA   NA         <NA>   1978  facero01            BBWAA     379    285
    ## 2708      NA   NA         <NA>   1978 mazerbi01            BBWAA     379    285
    ## 2709      NA   NA         <NA>   1978 boyerke01            BBWAA     379    285
    ## 2710      NA   NA         <NA>   1978 floodcu01            BBWAA     379    285
    ## 2711      NA   NA         <NA>   1978 haddiha01            BBWAA     379    285
    ## 2712      NA   NA         <NA>   1978 crandde01            BBWAA     379    285
    ## 2713      NA   NA         <NA>   1978   lawve01            BBWAA     379    285
    ## 2714      NA   NA         <NA>   1978 thomsbo01            BBWAA     379    285
    ## 2715      NA   NA         <NA>   1978 wertzvi01            BBWAA     379    285
    ## 2716      NA   NA         <NA>   1978 groatdi01            BBWAA     379    285
    ## 2717      NA   NA         <NA>   1978 malonji01            BBWAA     379    285
    ## 2718      NA   NA         <NA>   1978 boyercl02            BBWAA     379    285
    ## 2719      NA   NA         <NA>   1978 mclaide01            BBWAA     379    285
    ## 2720      NA   NA         <NA>   1978 pascuca02            BBWAA     379    285
    ## 2721      NA   NA         <NA>   1978 grantmu01            BBWAA     379    285
    ## 2722      NA   NA         <NA>   1978 ramospe01            BBWAA     379    285
    ## 2723      NA   NA         <NA>   1978  jossad01         Veterans      NA     NA
    ## 2724      NA   NA         <NA>   1978 macphla99         Veterans      NA     NA
    ## 2647      NA   NA         <NA>   1977 bankser01            BBWAA     383    288
    ## 2648      NA   NA         <NA>   1977 matheed01            BBWAA     383    288
    ## 2649      NA   NA         <NA>   1977 hodgegi01            BBWAA     383    288
    ## 2650      NA   NA         <NA>   1977 slaugen01            BBWAA     383    288
    ## 2651      NA   NA         <NA>   1977 snidedu01            BBWAA     383    288
    ## 2652      NA   NA         <NA>   1977 drysddo01            BBWAA     383    288
    ## 2653      NA   NA         <NA>   1977 reesepe01            BBWAA     383    288
    ## 2654      NA   NA         <NA>   1977   foxne01            BBWAA     383    288
    ## 2655      NA   NA         <NA>   1977 bunniji01            BBWAA     383    288
    ## 2656      NA   NA         <NA>   1977  kellge01            BBWAA     383    288
    ## 2657      NA   NA         <NA>   1977 ashburi01            BBWAA     383    288
    ## 2658      NA   NA         <NA>   1977 schoere01            BBWAA     383    288
    ## 2659      NA   NA         <NA>   1977 burdele01            BBWAA     383    288
    ## 2660      NA   NA         <NA>   1977 marisro01            BBWAA     383    288
    ## 2661      NA   NA         <NA>   1977  darkal01            BBWAA     383    288
    ## 2662      NA   NA         <NA>   1977 kuennha01            BBWAA     383    288
    ## 2663      NA   NA         <NA>   1977 kluszte01            BBWAA     383    288
    ## 2664      NA   NA         <NA>   1977 vernomi01            BBWAA     383    288
    ## 2665      NA   NA         <NA>   1977 coopewa01            BBWAA     383    288
    ## 2666      NA   NA         <NA>   1977 howarel01            BBWAA     383    288
    ## 2667      NA   NA         <NA>   1977 newcodo01            BBWAA     383    288
    ## 2668      NA   NA         <NA>   1977 larsedo01            BBWAA     383    288
    ## 2669      NA   NA         <NA>   1977  facero01            BBWAA     383    288
    ## 2670      NA   NA         <NA>   1977 floodcu01            BBWAA     383    288
    ## 2671      NA   NA         <NA>   1977 boyerke01            BBWAA     383    288
    ## 2672      NA   NA         <NA>   1977 thomsbo01            BBWAA     383    288
    ## 2673      NA   NA         <NA>   1977 crandde01            BBWAA     383    288
    ## 2674      NA   NA         <NA>   1977 haddiha01            BBWAA     383    288
    ## 2675      NA   NA         <NA>   1977   lawve01            BBWAA     383    288
    ## 2676      NA   NA         <NA>   1977 groatdi01            BBWAA     383    288
    ## 2677      NA   NA         <NA>   1977 wertzvi01            BBWAA     383    288
    ## 2678      NA   NA         <NA>   1977 whitewi01            BBWAA     383    288
    ## 2679      NA   NA         <NA>   1977 pascuca02            BBWAA     383    288
    ## 2680      NA   NA         <NA>   1977 podrejo01            BBWAA     383    288
    ## 2681      NA   NA         <NA>   1977 dihigma99     Negro League      NA     NA
    ## 2682      NA   NA         <NA>   1977 lloydpo99     Negro League      NA     NA
    ## 2683      NA   NA         <NA>   1977 lopezal01         Veterans      NA     NA
    ## 2684      NA   NA         <NA>   1977 rusieam01         Veterans      NA     NA
    ## 2685      NA   NA         <NA>   1977 seweljo01         Veterans      NA     NA
    ## 2611      NA   NA         <NA>   1976 roberro01            BBWAA     388    291
    ## 2612      NA   NA         <NA>   1976 lemonbo01            BBWAA     388    291
    ## 2613      NA   NA         <NA>   1976 hodgegi01            BBWAA     388    291
    ## 2614      NA   NA         <NA>   1976 slaugen01            BBWAA     388    291
    ## 2615      NA   NA         <NA>   1976 matheed01            BBWAA     388    291
    ## 2616      NA   NA         <NA>   1976 reesepe01            BBWAA     388    291
    ## 2617      NA   NA         <NA>   1976   foxne01            BBWAA     388    291
    ## 2618      NA   NA         <NA>   1976 snidedu01            BBWAA     388    291
    ## 2619      NA   NA         <NA>   1976 rizzuph01            BBWAA     388    291
    ## 2620      NA   NA         <NA>   1976  kellge01            BBWAA     388    291
    ## 2621      NA   NA         <NA>   1976 schoere01            BBWAA     388    291
    ## 2622      NA   NA         <NA>   1976 drysddo01            BBWAA     388    291
    ## 2623      NA   NA         <NA>   1976 marisro01            BBWAA     388    291
    ## 2624      NA   NA         <NA>   1976 ashburi01            BBWAA     388    291
    ## 2625      NA   NA         <NA>   1976  darkal01            BBWAA     388    291
    ## 2626      NA   NA         <NA>   1976 coopewa01            BBWAA     388    291
    ## 2627      NA   NA         <NA>   1976 howarel01            BBWAA     388    291
    ## 2628      NA   NA         <NA>   1976 vernomi01            BBWAA     388    291
    ## 2629      NA   NA         <NA>   1976 kluszte01            BBWAA     388    291
    ## 2630      NA   NA         <NA>   1976 larsedo01            BBWAA     388    291
    ## 2631      NA   NA         <NA>   1976  facero01            BBWAA     388    291
    ## 2632      NA   NA         <NA>   1976 burdele01            BBWAA     388    291
    ## 2633      NA   NA         <NA>   1976 newcodo01            BBWAA     388    291
    ## 2634      NA   NA         <NA>   1976 boyerke01            BBWAA     388    291
    ## 2635      NA   NA         <NA>   1976 crandde01            BBWAA     388    291
    ## 2636      NA   NA         <NA>   1976   lawve01            BBWAA     388    291
    ## 2637      NA   NA         <NA>   1976 thomsbo01            BBWAA     388    291
    ## 2638      NA   NA         <NA>   1976 haddiha01            BBWAA     388    291
    ## 2639      NA   NA         <NA>   1976 groatdi01            BBWAA     388    291
    ## 2640      NA   NA         <NA>   1976 whitewi01            BBWAA     388    291
    ## 2641      NA   NA         <NA>   1976 wertzvi01            BBWAA     388    291
    ## 2642      NA   NA         <NA>   1976 podrejo01            BBWAA     388    291
    ## 2643      NA   NA         <NA>   1976 charlos99     Negro League      NA     NA
    ## 2644      NA   NA         <NA>   1976 connoro01         Veterans      NA     NA
    ## 2645      NA   NA         <NA>   1976 hubbaca99         Veterans      NA     NA
    ## 2646      NA   NA         <NA>   1976 lindsfr01         Veterans      NA     NA
    ## 2570      NA   NA         <NA>   1975 kinerra01            BBWAA     362    272
    ## 2571      NA   NA         <NA>   1975 roberro01            BBWAA     362    272
    ## 2572      NA   NA         <NA>   1975 lemonbo01            BBWAA     362    272
    ## 2573      NA   NA         <NA>   1975 hodgegi01            BBWAA     362    272
    ## 2574      NA   NA         <NA>   1975 slaugen01            BBWAA     362    272
    ## 2575      NA   NA         <NA>   1975 newhoha01            BBWAA     362    272
    ## 2576      NA   NA         <NA>   1975 reesepe01            BBWAA     362    272
    ## 2577      NA   NA         <NA>   1975 matheed01            BBWAA     362    272
    ## 2578      NA   NA         <NA>   1975 cavarph01            BBWAA     362    272
    ## 2579      NA   NA         <NA>   1975 snidedu01            BBWAA     362    272
    ## 2580      NA   NA         <NA>   1975  sainjo01            BBWAA     362    272
    ## 2581      NA   NA         <NA>   1975 rizzuph01            BBWAA     362    272
    ## 2582      NA   NA         <NA>   1975  kellge01            BBWAA     362    272
    ## 2583      NA   NA         <NA>   1975 schoere01            BBWAA     362    272
    ## 2584      NA   NA         <NA>   1975 ashburi01            BBWAA     362    272
    ## 2585      NA   NA         <NA>   1975 drysddo01            BBWAA     362    272
    ## 2586      NA   NA         <NA>   1975   foxne01            BBWAA     362    272
    ## 2587      NA   NA         <NA>   1975 marisro01            BBWAA     362    272
    ## 2588      NA   NA         <NA>   1975  darkal01            BBWAA     362    272
    ## 2589      NA   NA         <NA>   1975 raschvi01            BBWAA     362    272
    ## 2590      NA   NA         <NA>   1975 kluszte01            BBWAA     362    272
    ## 2591      NA   NA         <NA>   1975 howarel01            BBWAA     362    272
    ## 2592      NA   NA         <NA>   1975 larsedo01            BBWAA     362    272
    ## 2593      NA   NA         <NA>   1975 vernomi01            BBWAA     362    272
    ## 2594      NA   NA         <NA>   1975 coopewa01            BBWAA     362    272
    ## 2595      NA   NA         <NA>   1975 burdele01            BBWAA     362    272
    ## 2596      NA   NA         <NA>   1975 newcodo01            BBWAA     362    272
    ## 2597      NA   NA         <NA>   1975 thomsbo01            BBWAA     362    272
    ## 2598      NA   NA         <NA>   1975 boyerke01            BBWAA     362    272
    ## 2599      NA   NA         <NA>   1975 haddiha01            BBWAA     362    272
    ## 2600      NA   NA         <NA>   1975 whitewi01            BBWAA     362    272
    ## 2601      NA   NA         <NA>   1975   lawve01            BBWAA     362    272
    ## 2602      NA   NA         <NA>   1975 wertzvi01            BBWAA     362    272
    ## 2603      NA   NA         <NA>   1975 groatdi01            BBWAA     362    272
    ## 2604      NA   NA         <NA>   1975 podrejo01            BBWAA     362    272
    ## 2605      NA   NA         <NA>   1975 colavro01            BBWAA     362    272
    ## 2606      NA   NA         <NA>   1975 virdobi01            BBWAA     362    272
    ## 2607      NA   NA         <NA>   1975 averiea01         Veterans      NA     NA
    ## 2608      NA   NA         <NA>   1975 harribu01         Veterans      NA     NA
    ## 2609      NA   NA         <NA>   1975 hermabi01         Veterans      NA     NA
    ## 2610      NA   NA         <NA>   1975 johnsju99     Negro League      NA     NA
    ## 2522      NA   NA         <NA>   1974 mantlmi01            BBWAA     365    274
    ## 2523      NA   NA         <NA>   1974  fordwh01            BBWAA     365    274
    ## 2524      NA   NA         <NA>   1974 roberro01            BBWAA     365    274
    ## 2525      NA   NA         <NA>   1974 kinerra01            BBWAA     365    274
    ## 2526      NA   NA         <NA>   1974 hodgegi01            BBWAA     365    274
    ## 2527      NA   NA         <NA>   1974 lemonbo01            BBWAA     365    274
    ## 2528      NA   NA         <NA>   1974 slaugen01            BBWAA     365    274
    ## 2529      NA   NA         <NA>   1974 reesepe01            BBWAA     365    274
    ## 2530      NA   NA         <NA>   1974 matheed01            BBWAA     365    274
    ## 2531      NA   NA         <NA>   1974 rizzuph01            BBWAA     365    274
    ## 2532      NA   NA         <NA>   1974 snidedu01            BBWAA     365    274
    ## 2533      NA   NA         <NA>   1974 schoere01            BBWAA     365    274
    ## 2534      NA   NA         <NA>   1974 reynoal01            BBWAA     365    274
    ## 2535      NA   NA         <NA>   1974  kellge01            BBWAA     365    274
    ## 2536      NA   NA         <NA>   1974   foxne01            BBWAA     365    274
    ## 2537      NA   NA         <NA>   1974 marisro01            BBWAA     365    274
    ## 2538      NA   NA         <NA>   1974 newhoha01            BBWAA     365    274
    ## 2539      NA   NA         <NA>   1974 cavarph01            BBWAA     365    274
    ## 2540      NA   NA         <NA>   1974 ashburi01            BBWAA     365    274
    ## 2541      NA   NA         <NA>   1974  darkal01            BBWAA     365    274
    ## 2542      NA   NA         <NA>   1974  sainjo01            BBWAA     365    274
    ## 2543      NA   NA         <NA>   1974 larsedo01            BBWAA     365    274
    ## 2544      NA   NA         <NA>   1974 kluszte01            BBWAA     365    274
    ## 2545      NA   NA         <NA>   1974 vernomi01            BBWAA     365    274
    ## 2546      NA   NA         <NA>   1974 howarel01            BBWAA     365    274
    ## 2547      NA   NA         <NA>   1974 erskica01            BBWAA     365    274
    ## 2548      NA   NA         <NA>   1974 coopewa01            BBWAA     365    274
    ## 2549      NA   NA         <NA>   1974 haddiha01            BBWAA     365    274
    ## 2550      NA   NA         <NA>   1974 burdele01            BBWAA     365    274
    ## 2551      NA   NA         <NA>   1974 newcodo01            BBWAA     365    274
    ## 2552      NA   NA         <NA>   1974 thomsbo01            BBWAA     365    274
    ## 2553      NA   NA         <NA>   1974   lawve01            BBWAA     365    274
    ## 2554      NA   NA         <NA>   1974 richabo01            BBWAA     365    274
    ## 2555      NA   NA         <NA>   1974 groatdi01            BBWAA     365    274
    ## 2556      NA   NA         <NA>   1974 mcmilro01            BBWAA     365    274
    ## 2557      NA   NA         <NA>   1974 piercbi02            BBWAA     365    274
    ## 2558      NA   NA         <NA>   1974 mcdougi01            BBWAA     365    274
    ## 2559      NA   NA         <NA>   1974 raschvi01            BBWAA     365    274
    ## 2560      NA   NA         <NA>   1974 shantbo01            BBWAA     365    274
    ## 2561      NA   NA         <NA>   1974 simmocu01            BBWAA     365    274
    ## 2562      NA   NA         <NA>   1974 virdobi01            BBWAA     365    274
    ## 2563      NA   NA         <NA>   1974 burgesm01            BBWAA     365    274
    ## 2564      NA   NA         <NA>   1974 colavro01            BBWAA     365    274
    ## 2565      NA   NA         <NA>   1974 wertzvi01            BBWAA     365    274
    ## 2566      NA   NA         <NA>   1974  bellco99     Negro League      NA     NA
    ## 2567      NA   NA         <NA>   1974 bottoji01         Veterans      NA     NA
    ## 2568      NA   NA         <NA>   1974 conlajo01         Veterans      NA     NA
    ## 2569      NA   NA         <NA>   1974 thompsa01         Veterans      NA     NA
    ## 2473      NA   NA         <NA>   1973 spahnwa01            BBWAA     380    285
    ## 2474      NA   NA         <NA>   1973  fordwh01            BBWAA     380    285
    ## 2475      NA   NA         <NA>   1973 kinerra01            BBWAA     380    285
    ## 2476      NA   NA         <NA>   1973 hodgegi01            BBWAA     380    285
    ## 2477      NA   NA         <NA>   1973 roberro01            BBWAA     380    285
    ## 2478      NA   NA         <NA>   1973 lemonbo01            BBWAA     380    285
    ## 2479      NA   NA         <NA>   1973  mizejo01            BBWAA     380    285
    ## 2480      NA   NA         <NA>   1973 slaugen01            BBWAA     380    285
    ## 2481      NA   NA         <NA>   1973 marioma01            BBWAA     380    285
    ## 2482      NA   NA         <NA>   1973 reesepe01            BBWAA     380    285
    ## 2483      NA   NA         <NA>   1973  kellge01            BBWAA     380    285
    ## 2484      NA   NA         <NA>   1973 rizzuph01            BBWAA     380    285
    ## 2485      NA   NA         <NA>   1973 snidedu01            BBWAA     380    285
    ## 2486      NA   NA         <NA>   1973 schoere01            BBWAA     380    285
    ## 2487      NA   NA         <NA>   1973 reynoal01            BBWAA     380    285
    ## 2488      NA   NA         <NA>   1973 newhoha01            BBWAA     380    285
    ## 2489      NA   NA         <NA>   1973 cavarph01            BBWAA     380    285
    ## 2490      NA   NA         <NA>   1973   foxne01            BBWAA     380    285
    ## 2491      NA   NA         <NA>   1973  darkal01            BBWAA     380    285
    ## 2492      NA   NA         <NA>   1973  sainjo01            BBWAA     380    285
    ## 2493      NA   NA         <NA>   1973 dimagdo01            BBWAA     380    285
    ## 2494      NA   NA         <NA>   1973 newsobo01            BBWAA     380    285
    ## 2495      NA   NA         <NA>   1973 ashburi01            BBWAA     380    285
    ## 2496      NA   NA         <NA>   1973 vernomi01            BBWAA     380    285
    ## 2497      NA   NA         <NA>   1973 kluszte01            BBWAA     380    285
    ## 2498      NA   NA         <NA>   1973 burdele01            BBWAA     380    285
    ## 2499      NA   NA         <NA>   1973 newcodo01            BBWAA     380    285
    ## 2500      NA   NA         <NA>   1973   lawve01            BBWAA     380    285
    ## 2501      NA   NA         <NA>   1973 coopewa01            BBWAA     380    285
    ## 2502      NA   NA         <NA>   1973 groatdi01            BBWAA     380    285
    ## 2503      NA   NA         <NA>   1973 raschvi01            BBWAA     380    285
    ## 2504      NA   NA         <NA>   1973 leonadu02            BBWAA     380    285
    ## 2505      NA   NA         <NA>   1973 mcmilro01            BBWAA     380    285
    ## 2506      NA   NA         <NA>   1973 shantbo01            BBWAA     380    285
    ## 2507      NA   NA         <NA>   1973 simmocu01            BBWAA     380    285
    ## 2508      NA   NA         <NA>   1973 erskica01            BBWAA     380    285
    ## 2509      NA   NA         <NA>   1973 piercbi02            BBWAA     380    285
    ## 2510      NA   NA         <NA>   1973 brechha01            BBWAA     380    285
    ## 2511      NA   NA         <NA>   1973 thomsbo01            BBWAA     380    285
    ## 2512      NA   NA         <NA>   1973 mcdougi01            BBWAA     380    285
    ## 2513      NA   NA         <NA>   1973 richabo01            BBWAA     380    285
    ## 2514      NA   NA         <NA>   1973 wertzvi01            BBWAA     380    285
    ## 2515      NA   NA         <NA>   1973 burgesm01            BBWAA     380    285
    ## 2516      NA   NA         <NA>   1973 haddiha01            BBWAA     380    285
    ## 2517      NA   NA         <NA>   1973 clemero01 Special Election      NA     NA
    ## 2518      NA   NA         <NA>   1973 evansbi99         Veterans      NA     NA
    ## 2519      NA   NA         <NA>   1973 irvinmo01     Negro League      NA     NA
    ## 2520      NA   NA         <NA>   1973 kellyge01         Veterans      NA     NA
    ## 2521      NA   NA         <NA>   1973 welchmi01         Veterans      NA     NA
    ## 2422      NA   NA         <NA>   1972 koufasa01            BBWAA     396    297
    ## 2423      NA   NA         <NA>   1972 berrayo01            BBWAA     396    297
    ## 2424      NA   NA         <NA>   1972  wynnea01            BBWAA     396    297
    ## 2425      NA   NA         <NA>   1972 kinerra01            BBWAA     396    297
    ## 2426      NA   NA         <NA>   1972 hodgegi01            BBWAA     396    297
    ## 2427      NA   NA         <NA>   1972  mizejo01            BBWAA     396    297
    ## 2428      NA   NA         <NA>   1972 slaugen01            BBWAA     396    297
    ## 2429      NA   NA         <NA>   1972 reesepe01            BBWAA     396    297
    ## 2430      NA   NA         <NA>   1972 marioma01            BBWAA     396    297
    ## 2431      NA   NA         <NA>   1972 lemonbo01            BBWAA     396    297
    ## 2432      NA   NA         <NA>   1972  kellge01            BBWAA     396    297
    ## 2433      NA   NA         <NA>   1972 reynoal01            BBWAA     396    297
    ## 2434      NA   NA         <NA>   1972 schoere01            BBWAA     396    297
    ## 2435      NA   NA         <NA>   1972 rizzuph01            BBWAA     396    297
    ## 2436      NA   NA         <NA>   1972 newhoha01            BBWAA     396    297
    ## 2437      NA   NA         <NA>   1972 snidedu01            BBWAA     396    297
    ## 2438      NA   NA         <NA>   1972   foxne01            BBWAA     396    297
    ## 2439      NA   NA         <NA>   1972 cavarph01            BBWAA     396    297
    ## 2440      NA   NA         <NA>   1972  darkal01            BBWAA     396    297
    ## 2441      NA   NA         <NA>   1972 dimagdo01            BBWAA     396    297
    ## 2442      NA   NA         <NA>   1972 newsobo01            BBWAA     396    297
    ## 2443      NA   NA         <NA>   1972 kellech01            BBWAA     396    297
    ## 2444      NA   NA         <NA>   1972  sainjo01            BBWAA     396    297
    ## 2445      NA   NA         <NA>   1972 vernomi01            BBWAA     396    297
    ## 2446      NA   NA         <NA>   1972 ashburi01            BBWAA     396    297
    ## 2447      NA   NA         <NA>   1972 kluszte01            BBWAA     396    297
    ## 2448      NA   NA         <NA>   1972 thomsbo01            BBWAA     396    297
    ## 2449      NA   NA         <NA>   1972 haddiha01            BBWAA     396    297
    ## 2450      NA   NA         <NA>   1972 mcmilro01            BBWAA     396    297
    ## 2451      NA   NA         <NA>   1972 shantbo01            BBWAA     396    297
    ## 2452      NA   NA         <NA>   1972 coopewa01            BBWAA     396    297
    ## 2453      NA   NA         <NA>   1972 richabo01            BBWAA     396    297
    ## 2454      NA   NA         <NA>   1972 newcodo01            BBWAA     396    297
    ## 2455      NA   NA         <NA>   1972 brechha01            BBWAA     396    297
    ## 2456      NA   NA         <NA>   1972 leonadu02            BBWAA     396    297
    ## 2457      NA   NA         <NA>   1972 erskica01            BBWAA     396    297
    ## 2458      NA   NA         <NA>   1972 mcdougi01            BBWAA     396    297
    ## 2459      NA   NA         <NA>   1972 piercbi02            BBWAA     396    297
    ## 2460      NA   NA         <NA>   1972 raschvi01            BBWAA     396    297
    ## 2461      NA   NA         <NA>   1972 wertzvi01            BBWAA     396    297
    ## 2462      NA   NA         <NA>   1972 powervi01            BBWAA     396    297
    ## 2463      NA   NA         <NA>   1972 sievero01            BBWAA     396    297
    ## 2464      NA   NA         <NA>   1972 furilca01            BBWAA     396    297
    ## 2465      NA   NA         <NA>   1972 lopated01            BBWAA     396    297
    ## 2466      NA   NA         <NA>   1972   roepr01            BBWAA     396    297
    ## 2467      NA   NA         <NA>   1972 jenseja01            BBWAA     396    297
    ## 2468      NA   NA         <NA>   1972 gibsojo99     Negro League      NA     NA
    ## 2469      NA   NA         <NA>   1972 gomezle01         Veterans      NA     NA
    ## 2470      NA   NA         <NA>   1972 harriwi99         Veterans      NA     NA
    ## 2471      NA   NA         <NA>   1972 leonabu99     Negro League      NA     NA
    ## 2472      NA   NA         <NA>   1972 youngro01         Veterans      NA     NA
    ## 2366      NA   NA         <NA>   1971 berrayo01            BBWAA     360    270
    ## 2367      NA   NA         <NA>   1971  wynnea01            BBWAA     360    270
    ## 2368      NA   NA         <NA>   1971 kinerra01            BBWAA     360    270
    ## 2369      NA   NA         <NA>   1971 hodgegi01            BBWAA     360    270
    ## 2370      NA   NA         <NA>   1971 slaugen01            BBWAA     360    270
    ## 2371      NA   NA         <NA>   1971  mizejo01            BBWAA     360    270
    ## 2372      NA   NA         <NA>   1971 reesepe01            BBWAA     360    270
    ## 2373      NA   NA         <NA>   1971 marioma01            BBWAA     360    270
    ## 2374      NA   NA         <NA>   1971 schoere01            BBWAA     360    270
    ## 2375      NA   NA         <NA>   1971 reynoal01            BBWAA     360    270
    ## 2376      NA   NA         <NA>   1971  kellge01            BBWAA     360    270
    ## 2377      NA   NA         <NA>   1971 vandejo01            BBWAA     360    270
    ## 2378      NA   NA         <NA>   1971 newhoha01            BBWAA     360    270
    ## 2379      NA   NA         <NA>   1971 rizzuph01            BBWAA     360    270
    ## 2380      NA   NA         <NA>   1971 lemonbo01            BBWAA     360    270
    ## 2381      NA   NA         <NA>   1971 snidedu01            BBWAA     360    270
    ## 2382      NA   NA         <NA>   1971 cavarph01            BBWAA     360    270
    ## 2383      NA   NA         <NA>   1971 doerrbo01            BBWAA     360    270
    ## 2384      NA   NA         <NA>   1971  darkal01            BBWAA     360    270
    ## 2385      NA   NA         <NA>   1971   foxne01            BBWAA     360    270
    ## 2386      NA   NA         <NA>   1971 newsobo01            BBWAA     360    270
    ## 2387      NA   NA         <NA>   1971 dimagdo01            BBWAA     360    270
    ## 2388      NA   NA         <NA>   1971 kellech01            BBWAA     360    270
    ## 2389      NA   NA         <NA>   1971 vernomi01            BBWAA     360    270
    ## 2390      NA   NA         <NA>   1971  sainjo01            BBWAA     360    270
    ## 2391      NA   NA         <NA>   1971 ashburi01            BBWAA     360    270
    ## 2392      NA   NA         <NA>   1971 haddiha01            BBWAA     360    270
    ## 2393      NA   NA         <NA>   1971 kluszte01            BBWAA     360    270
    ## 2394      NA   NA         <NA>   1971 newcodo01            BBWAA     360    270
    ## 2395      NA   NA         <NA>   1971 brechha01            BBWAA     360    270
    ## 2396      NA   NA         <NA>   1971 coopewa01            BBWAA     360    270
    ## 2397      NA   NA         <NA>   1971 moseswa01            BBWAA     360    270
    ## 2398      NA   NA         <NA>   1971 piercbi02            BBWAA     360    270
    ## 2399      NA   NA         <NA>   1971 furilca01            BBWAA     360    270
    ## 2400      NA   NA         <NA>   1971 shantbo01            BBWAA     360    270
    ## 2401      NA   NA         <NA>   1971 lopated01            BBWAA     360    270
    ## 2402      NA   NA         <NA>   1971 mcdougi01            BBWAA     360    270
    ## 2403      NA   NA         <NA>   1971 sievero01            BBWAA     360    270
    ## 2404      NA   NA         <NA>   1971 thomsbo01            BBWAA     360    270
    ## 2405      NA   NA         <NA>   1971 erskica01            BBWAA     360    270
    ## 2406      NA   NA         <NA>   1971 leonadu02            BBWAA     360    270
    ## 2407      NA   NA         <NA>   1971   roepr01            BBWAA     360    270
    ## 2408      NA   NA         <NA>   1971 jenseja01            BBWAA     360    270
    ## 2409      NA   NA         <NA>   1971  moonwa01            BBWAA     360    270
    ## 2410      NA   NA         <NA>   1971 powervi01            BBWAA     360    270
    ## 2411      NA   NA         <NA>   1971 raschvi01            BBWAA     360    270
    ## 2412      NA   NA         <NA>   1971 wertzvi01            BBWAA     360    270
    ## 2413      NA   NA         <NA>   1971 brutobi01            BBWAA     360    270
    ## 2414      NA   NA         <NA>   1971 bancrda01         Veterans      NA     NA
    ## 2415      NA   NA         <NA>   1971 becklja01         Veterans      NA     NA
    ## 2416      NA   NA         <NA>   1971 hafeych01         Veterans      NA     NA
    ## 2417      NA   NA         <NA>   1971 hoopeha01         Veterans      NA     NA
    ## 2418      NA   NA         <NA>   1971 kellejo01         Veterans      NA     NA
    ## 2419      NA   NA         <NA>   1971 marquru01         Veterans      NA     NA
    ## 2420      NA   NA         <NA>   1971 paigesa01     Negro League      NA     NA
    ## 2421      NA   NA         <NA>   1971 weissge99         Veterans      NA     NA
    ## 2317      NA   NA         <NA>   1970 boudrlo01            BBWAA     300    225
    ## 2318      NA   NA         <NA>   1970 kinerra01            BBWAA     300    225
    ## 2319      NA   NA         <NA>   1970 hodgegi01            BBWAA     300    225
    ## 2320      NA   NA         <NA>   1970  wynnea01            BBWAA     300    225
    ## 2321      NA   NA         <NA>   1970 slaugen01            BBWAA     300    225
    ## 2322      NA   NA         <NA>   1970  mizejo01            BBWAA     300    225
    ## 2323      NA   NA         <NA>   1970 marioma01            BBWAA     300    225
    ## 2324      NA   NA         <NA>   1970 reesepe01            BBWAA     300    225
    ## 2325      NA   NA         <NA>   1970 schoere01            BBWAA     300    225
    ## 2326      NA   NA         <NA>   1970  kellge01            BBWAA     300    225
    ## 2327      NA   NA         <NA>   1970 reynoal01            BBWAA     300    225
    ## 2328      NA   NA         <NA>   1970 vandejo01            BBWAA     300    225
    ## 2329      NA   NA         <NA>   1970 newhoha01            BBWAA     300    225
    ## 2330      NA   NA         <NA>   1970 gordojo01            BBWAA     300    225
    ## 2331      NA   NA         <NA>   1970 rizzuph01            BBWAA     300    225
    ## 2332      NA   NA         <NA>   1970 doerrbo01            BBWAA     300    225
    ## 2333      NA   NA         <NA>   1970 lemonbo01            BBWAA     300    225
    ## 2334      NA   NA         <NA>   1970 henrito01            BBWAA     300    225
    ## 2335      NA   NA         <NA>   1970  darkal01            BBWAA     300    225
    ## 2336      NA   NA         <NA>   1970 cavarph01            BBWAA     300    225
    ## 2337      NA   NA         <NA>   1970 snidedu01            BBWAA     300    225
    ## 2338      NA   NA         <NA>   1970 waltebu01            BBWAA     300    225
    ## 2339      NA   NA         <NA>   1970 dimagdo01            BBWAA     300    225
    ## 2340      NA   NA         <NA>   1970 blackew01            BBWAA     300    225
    ## 2341      NA   NA         <NA>   1970 newsobo01            BBWAA     300    225
    ## 2342      NA   NA         <NA>   1970 ashburi01            BBWAA     300    225
    ## 2343      NA   NA         <NA>   1970 vernomi01            BBWAA     300    225
    ## 2344      NA   NA         <NA>   1970 coopewa01            BBWAA     300    225
    ## 2345      NA   NA         <NA>   1970  sainjo01            BBWAA     300    225
    ## 2346      NA   NA         <NA>   1970 kluszte01            BBWAA     300    225
    ## 2347      NA   NA         <NA>   1970 kellech01            BBWAA     300    225
    ## 2348      NA   NA         <NA>   1970 shantbo01            BBWAA     300    225
    ## 2349      NA   NA         <NA>   1970 leonadu02            BBWAA     300    225
    ## 2350      NA   NA         <NA>   1970 moseswa01            BBWAA     300    225
    ##      votes inducted          category needed_note
    ## 1       NA     <NA>              <NA>        <NA>
    ## 2       NA     <NA>              <NA>        <NA>
    ## 3       NA     <NA>              <NA>        <NA>
    ## 4       NA     <NA>              <NA>        <NA>
    ## 5       NA     <NA>              <NA>        <NA>
    ## 6       NA     <NA>              <NA>        <NA>
    ## 7       NA     <NA>              <NA>        <NA>
    ## 8       NA     <NA>              <NA>        <NA>
    ## 9       NA     <NA>              <NA>        <NA>
    ## 10      NA     <NA>              <NA>        <NA>
    ## 11      NA     <NA>              <NA>        <NA>
    ## 12      NA     <NA>              <NA>        <NA>
    ## 13      NA     <NA>              <NA>        <NA>
    ## 14      NA     <NA>              <NA>        <NA>
    ## 15      NA     <NA>              <NA>        <NA>
    ## 16      NA     <NA>              <NA>        <NA>
    ## 17      NA     <NA>              <NA>        <NA>
    ## 18      NA     <NA>              <NA>        <NA>
    ## 19      NA     <NA>              <NA>        <NA>
    ## 20      NA     <NA>              <NA>        <NA>
    ## 21      NA     <NA>              <NA>        <NA>
    ## 22      NA     <NA>              <NA>        <NA>
    ## 23      NA     <NA>              <NA>        <NA>
    ## 24      NA     <NA>              <NA>        <NA>
    ## 4312   307        Y            Player        <NA>
    ## 4313   260        N            Player        <NA>
    ## 4314   257        N            Player        <NA>
    ## 4315   249        N            Player        <NA>
    ## 4316   231        N            Player        <NA>
    ## 4317   205        N            Player        <NA>
    ## 4318   201        N            Player        <NA>
    ## 4319   163        N            Player        <NA>
    ## 4320   160        N            Player        <NA>
    ## 4321   135        N            Player        <NA>
    ## 4322   129        N            Player        <NA>
    ## 4323   114        N            Player        <NA>
    ## 4324    94        N            Player        <NA>
    ## 4325    73        N            Player        <NA>
    ## 4326    42        N            Player        <NA>
    ## 4327    37        N            Player        <NA>
    ## 4328    34        N            Player        <NA>
    ## 4329    23        N            Player        <NA>
    ## 4330    21        N            Player        <NA>
    ## 4331    17        N            Player        <NA>
    ## 4332    12        N            Player        <NA>
    ## 4333     9        N            Player        <NA>
    ## 4334     8        N            Player        <NA>
    ## 4335     6        N            Player        <NA>
    ## 4336     5        N            Player        <NA>
    ## 4337     5        N            Player        <NA>
    ## 4338     2        N            Player        <NA>
    ## 4339     2        N            Player        <NA>
    ## 4340     0        N            Player        <NA>
    ## 4341     0        N            Player        <NA>
    ## 4342    NA        Y Pioneer/Executive        <NA>
    ## 4343    NA        Y            Player        <NA>
    ## 4344    NA        Y            Player        <NA>
    ## 4345    NA        Y            Player        <NA>
    ## 4346    NA        Y Pioneer/Executive        <NA>
    ## 4347    NA        Y            Player        <NA>
    ## 4287   285        N            Player        <NA>
    ## 4288   248        N            Player        <NA>
    ## 4289   247        N            Player        <NA>
    ## 4290   212        N            Player        <NA>
    ## 4291   197        N            Player        <NA>
    ## 4292   186        N            Player        <NA>
    ## 4293   180        N            Player        <NA>
    ## 4294   163        N            Player        <NA>
    ## 4295   136        N            Player        <NA>
    ## 4296   130        N            Player        <NA>
    ## 4297   113        N            Player        <NA>
    ## 4298    68        N            Player        <NA>
    ## 4299    55        N            Player        <NA>
    ## 4300    44        N            Player        <NA>
    ## 4301    38        N            Player        <NA>
    ## 4302    35        N            Player        <NA>
    ## 4303    21        N            Player        <NA>
    ## 4304     4        N            Player        <NA>
    ## 4305     2        N            Player        <NA>
    ## 4306     1        N            Player        <NA>
    ## 4307     0        N            Player        <NA>
    ## 4308     0        N            Player        <NA>
    ## 4309     0        N            Player        <NA>
    ## 4310     0        N            Player        <NA>
    ## 4311     0        N            Player        <NA>
    ## 4253   396        Y            Player        <NA>
    ## 4254   304        Y            Player        <NA>
    ## 4255   278        N            Player        <NA>
    ## 4256   242        N            Player        <NA>
    ## 4257   241        N            Player        <NA>
    ## 4258   209        N            Player        <NA>
    ## 4259   140        N            Player        <NA>
    ## 4260   126        N            Player        <NA>
    ## 4261   121        N            Player        <NA>
    ## 4262   116        N            Player        <NA>
    ## 4263   112        N            Player        <NA>
    ## 4264   109        N            Player        <NA>
    ## 4265    77        N            Player        <NA>
    ## 4266    55        N            Player        <NA>
    ## 4267    45        N            Player        <NA>
    ## 4268    22        N            Player        <NA>
    ## 4269    10        N            Player        <NA>
    ## 4270     6        N            Player        <NA>
    ## 4271     6        N            Player        <NA>
    ## 4272     2        N            Player        <NA>
    ## 4273     2        N            Player        <NA>
    ## 4274     1        N            Player        <NA>
    ## 4275     1        N            Player        <NA>
    ## 4276     1        N            Player        <NA>
    ## 4277     1        N            Player        <NA>
    ## 4278     0        N            Player        <NA>
    ## 4279     0        N            Player        <NA>
    ## 4280     0        N            Player        <NA>
    ## 4281     0        N            Player        <NA>
    ## 4282     0        N            Player        <NA>
    ## 4283     0        N            Player        <NA>
    ## 4284     0        N            Player        <NA>
    ## 4285    NA        Y Pioneer/Executive        <NA>
    ## 4286    NA        Y            Player        <NA>
    ## 4216   425        Y            Player        <NA>
    ## 4217   363        Y            Player        <NA>
    ## 4218   363        Y            Player        <NA>
    ## 4219   326        Y            Player        <NA>
    ## 4220   259        N            Player        <NA>
    ## 4221   253        N            Player        <NA>
    ## 4222   251        N            Player        <NA>
    ## 4223   232        N            Player        <NA>
    ## 4224   182        N            Player        <NA>
    ## 4225   169        N            Player        <NA>
    ## 4226    97        N            Player        <NA>
    ## 4227    77        N            Player        <NA>
    ## 4228    73        N            Player        <NA>
    ## 4229    71        N            Player        <NA>
    ## 4230    70        N            Player        <NA>
    ## 4231    58        N            Player        <NA>
    ## 4232    42        N            Player        <NA>
    ## 4233    36        N            Player        <NA>
    ## 4234    32        N            Player        <NA>
    ## 4235     9        N            Player        <NA>
    ## 4236     5        N            Player        <NA>
    ## 4237     5        N            Player        <NA>
    ## 4238     4        N            Player        <NA>
    ## 4239     2        N            Player        <NA>
    ## 4240     0        N            Player        <NA>
    ## 4241     0        N            Player        <NA>
    ## 4242     0        N            Player        <NA>
    ## 4243     0        N            Player        <NA>
    ## 4244     0        N            Player        <NA>
    ## 4245     0        N            Player        <NA>
    ## 4246     0        N            Player        <NA>
    ## 4247     0        N            Player        <NA>
    ## 4248     0        N            Player        <NA>
    ## 4249     0        N            Player        <NA>
    ## 4250     0        N            Player        <NA>
    ## 4251    NA        Y            Player        <NA>
    ## 4252    NA        Y            Player        <NA>
    ## 4181   410        Y            Player        <NA>
    ## 4182   392        Y            Player        <NA>
    ## 4183   379        Y            Player        <NA>
    ## 4184   337        Y            Player        <NA>
    ## 4185   297        N            Player        <NA>
    ## 4186   268        N            Player        <NA>
    ## 4187   242        N            Player        <NA>
    ## 4188   238        N            Player        <NA>
    ## 4189   216        N            Player        <NA>
    ## 4190   156        N            Player        <NA>
    ## 4191   144        N            Player        <NA>
    ## 4192    98        N            Player        <NA>
    ## 4193    93        N            Player        <NA>
    ## 4194    61        N            Player        <NA>
    ## 4195    47        N            Player        <NA>
    ## 4196    47        N            Player        <NA>
    ## 4197    43        N            Player        <NA>
    ## 4198    33        N            Player        <NA>
    ## 4199    31        N            Player        <NA>
    ## 4200    10        N            Player        <NA>
    ## 4201    10        N            Player        <NA>
    ## 4202     8        N            Player        <NA>
    ## 4203     4        N            Player        <NA>
    ## 4204     2        N            Player        <NA>
    ## 4205     2        N            Player        <NA>
    ## 4206     1        N            Player        <NA>
    ## 4207     1        N            Player        <NA>
    ## 4208     0        N            Player        <NA>
    ## 4209     0        N            Player        <NA>
    ## 4210     0        N            Player        <NA>
    ## 4211     0        N            Player        <NA>
    ## 4212     0        N            Player        <NA>
    ## 4213     0        N            Player        <NA>
    ## 4214    NA        Y            Player        <NA>
    ## 4215    NA        Y            Player        <NA>
    ## 4145   381        Y            Player        <NA>
    ## 4146   380        Y            Player        <NA>
    ## 4147   336        Y            Player        <NA>
    ## 4148   327        N            Player        <NA>
    ## 4149   317        N            Player        <NA>
    ## 4150   259        N            Player        <NA>
    ## 4151   239        N            Player        <NA>
    ## 4152   238        N            Player        <NA>
    ## 4153   229        N            Player        <NA>
    ## 4154   199        N            Player        <NA>
    ## 4155   151        N            Player        <NA>
    ## 4156   105        N            Player        <NA>
    ## 4157    97        N            Player        <NA>
    ## 4158    96        N            Player        <NA>
    ## 4159    74        N            Player        <NA>
    ## 4160    59        N            Player        <NA>
    ## 4161    45        N            Player        <NA>
    ## 4162    38        N            Player        <NA>
    ## 4163    17        N            Player        <NA>
    ## 4164     3        N            Player        <NA>
    ## 4165     2        N            Player        <NA>
    ## 4166     2        N            Player        <NA>
    ## 4167     1        N            Player        <NA>
    ## 4168     0        N            Player        <NA>
    ## 4169     0        N            Player        <NA>
    ## 4170     0        N            Player        <NA>
    ## 4171     0        N            Player        <NA>
    ## 4172     0        N            Player        <NA>
    ## 4173     0        N            Player        <NA>
    ## 4174     0        N            Player        <NA>
    ## 4175     0        N            Player        <NA>
    ## 4176     0        N            Player        <NA>
    ## 4177     0        N            Player        <NA>
    ## 4178     0        N            Player        <NA>
    ## 4179    NA        Y Pioneer/Executive        <NA>
    ## 4180    NA        Y Pioneer/Executive        <NA>
    ## 4113   437        Y            Player        <NA>
    ## 4114   365        Y            Player        <NA>
    ## 4115   315        N            Player        <NA>
    ## 4116   307        N            Player        <NA>
    ## 4117   296        N            Player        <NA>
    ## 4118   230        N            Player        <NA>
    ## 4119   199        N            Player        <NA>
    ## 4120   195        N            Player        <NA>
    ## 4121   191        N            Player        <NA>
    ## 4122   189        N            Player        <NA>
    ## 4123   180        N            Player        <NA>
    ## 4124   150        N            Player        <NA>
    ## 4125    92        N            Player        <NA>
    ## 4126    73        N            Player        <NA>
    ## 4127    68        N            Player        <NA>
    ## 4128    54        N            Player        <NA>
    ## 4129    51        N            Player        <NA>
    ## 4130    46        N            Player        <NA>
    ## 4131    31        N            Player        <NA>
    ## 4132    11        N            Player        <NA>
    ## 4133     8        N            Player        <NA>
    ## 4134     3        N            Player        <NA>
    ## 4135     2        N            Player        <NA>
    ## 4136     2        N            Player        <NA>
    ## 4137     1        N            Player        <NA>
    ## 4138     0        N            Player        <NA>
    ## 4139     0        N            Player        <NA>
    ## 4140     0        N            Player        <NA>
    ## 4141     0        N            Player        <NA>
    ## 4142     0        N            Player        <NA>
    ## 4143     0        N            Player        <NA>
    ## 4144     0        N            Player        <NA>
    ## 4079   534        Y            Player        <NA>
    ## 4080   500        Y            Player        <NA>
    ## 4081   455        Y            Player        <NA>
    ## 4082   454        Y            Player        <NA>
    ## 4083   384        N            Player        <NA>
    ## 4084   306        N            Player        <NA>
    ## 4085   302        N            Player        <NA>
    ## 4086   215        N            Player        <NA>
    ## 4087   206        N            Player        <NA>
    ## 4088   202        N            Player        <NA>
    ## 4089   166        N            Player        <NA>
    ## 4090   148        N            Player        <NA>
    ## 4091   138        N            Player        <NA>
    ## 4092   135        N            Player        <NA>
    ## 4093    77        N            Player        <NA>
    ## 4094    71        N            Player        <NA>
    ## 4095    65        N            Player        <NA>
    ## 4096    64        N            Player        <NA>
    ## 4097    55        N            Player        <NA>
    ## 4098    50        N            Player        <NA>
    ## 4099    36        N            Player        <NA>
    ## 4100    30        N            Player        <NA>
    ## 4101    21        N            Player        <NA>
    ## 4102     4        N            Player        <NA>
    ## 4103     2        N            Player        <NA>
    ## 4104     2        N            Player        <NA>
    ## 4105     1        N            Player        <NA>
    ## 4106     0        N            Player        <NA>
    ## 4107     0        N            Player        <NA>
    ## 4108     0        N            Player        <NA>
    ## 4109     0        N            Player        <NA>
    ## 4110     0        N            Player        <NA>
    ## 4111     0        N            Player        <NA>
    ## 4112     0        N            Player        <NA>
    ## 4040    NA        Y           Manager        <NA>
    ## 4041    NA        Y           Manager        <NA>
    ## 4042    NA        Y           Manager        <NA>
    ## 4043   555        Y            Player        <NA>
    ## 4044   525        Y            Player        <NA>
    ## 4045   478        Y            Player        <NA>
    ## 4046   427        N            Player        <NA>
    ## 4047   355        N            Player        <NA>
    ## 4048   351        N            Player        <NA>
    ## 4049   310        N            Player        <NA>
    ## 4050   263        N            Player        <NA>
    ## 4051   202        N            Player        <NA>
    ## 4052   198        N            Player        <NA>
    ## 4053   171        N            Player        <NA>
    ## 4054   167        N            Player        <NA>
    ## 4055   144        N            Player        <NA>
    ## 4056   119        N            Player        <NA>
    ## 4057   116        N            Player        <NA>
    ## 4058    87        N            Player        <NA>
    ## 4059    67        N            Player        <NA>
    ## 4060    63        N            Player        <NA>
    ## 4061    58        N            Player        <NA>
    ## 4062    47        N            Player        <NA>
    ## 4063    41        N            Player        <NA>
    ## 4064    25        N            Player        <NA>
    ## 4065     6        N            Player        <NA>
    ## 4066     6        N            Player        <NA>
    ## 4067     5        N            Player        <NA>
    ## 4068     2        N            Player        <NA>
    ## 4069     2        N            Player        <NA>
    ## 4070     1        N            Player        <NA>
    ## 4071     1        N            Player        <NA>
    ## 4072     1        N            Player        <NA>
    ## 4073     0        N            Player        <NA>
    ## 4074     0        N            Player        <NA>
    ## 4075     0        N            Player        <NA>
    ## 4076     0        N            Player        <NA>
    ## 4077     0        N            Player        <NA>
    ## 4078     0        N            Player        <NA>
    ## 4000   388        N            Player        <NA>
    ## 4001   385        N            Player        <NA>
    ## 4002   339        N            Player        <NA>
    ## 4003   329        N            Player        <NA>
    ## 4004   297        N            Player        <NA>
    ## 4005   272        N            Player        <NA>
    ## 4006   221        N            Player        <NA>
    ## 4007   214        N            Player        <NA>
    ## 4008   206        N            Player        <NA>
    ## 4009   204        N            Player        <NA>
    ## 4010   191        N            Player        <NA>
    ## 4011   123        N            Player        <NA>
    ## 4012   118        N            Player        <NA>
    ## 4013   106        N            Player        <NA>
    ## 4014    96        N            Player        <NA>
    ## 4015    75        N            Player        <NA>
    ## 4016    71        N            Player        <NA>
    ## 4017    50        N            Player        <NA>
    ## 4018    19        N            Player        <NA>
    ## 4019    18        N            Player        <NA>
    ## 4020    16        N            Player        <NA>
    ## 4021     6        N            Player        <NA>
    ## 4022     5        N            Player        <NA>
    ## 4023     4        N            Player        <NA>
    ## 4024     2        N            Player        <NA>
    ## 4025     1        N            Player        <NA>
    ## 4026     0        N            Player        <NA>
    ## 4027     0        N            Player        <NA>
    ## 4028     0        N            Player        <NA>
    ## 4029     0        N            Player        <NA>
    ## 4030     0        N            Player        <NA>
    ## 4031     0        N            Player        <NA>
    ## 4032     0        N            Player        <NA>
    ## 4033     0        N            Player        <NA>
    ## 4034     0        N            Player        <NA>
    ## 4035     0        N            Player        <NA>
    ## 4036     0        N            Player        <NA>
    ## 4037    NA        Y Pioneer/Executive        <NA>
    ## 4038    NA        Y            Umpire        <NA>
    ## 4039    NA        Y            Player        <NA>
    ## 3972   495        Y            Player        <NA>
    ## 3973   382        N            Player        <NA>
    ## 3974   321        N            Player        <NA>
    ## 3975   290        N            Player        <NA>
    ## 3976   279        N            Player        <NA>
    ## 3977   211        N            Player        <NA>
    ## 3978   209        N            Player        <NA>
    ## 3979   137        N            Player        <NA>
    ## 3980   131        N            Player        <NA>
    ## 3981   112        N            Player        <NA>
    ## 3982   102        N            Player        <NA>
    ## 3983    83        N            Player        <NA>
    ## 3984    72        N            Player        <NA>
    ## 3985    55        N            Player        <NA>
    ## 3986    23        N            Player        <NA>
    ## 3987     6        N            Player        <NA>
    ## 3988     5        N            Player        <NA>
    ## 3989     4        N            Player        <NA>
    ## 3990     2        N            Player        <NA>
    ## 3991     1        N            Player        <NA>
    ## 3992     1        N            Player        <NA>
    ## 3993     0        N            Player        <NA>
    ## 3994     0        N            Player        <NA>
    ## 3995     0        N            Player        <NA>
    ## 3996     0        N            Player        <NA>
    ## 3997     0        N            Player        <NA>
    ## 3998     0        N            Player        <NA>
    ## 3999    NA        Y            Player        <NA>
    ## 3938   523        Y            Player        <NA>
    ## 3939   463        Y            Player        <NA>
    ## 3940   361        N            Player        <NA>
    ## 3941   311        N            Player        <NA>
    ## 3942   263        N            Player        <NA>
    ## 3943   242        N            Player        <NA>
    ## 3944   218        N            Player        <NA>
    ## 3945   191        N            Player        <NA>
    ## 3946   141        N            Player        <NA>
    ## 3947   118        N            Player        <NA>
    ## 3948   115        N            Player        <NA>
    ## 3949   104        N            Player        <NA>
    ## 3950    89        N            Player        <NA>
    ## 3951    79        N            Player        <NA>
    ## 3952    73        N            Player        <NA>
    ## 3953    64        N            Player        <NA>
    ## 3954    30        N            Player        <NA>
    ## 3955    28        N            Player        <NA>
    ## 3956    27        N            Player        <NA>
    ## 3957    12        N            Player        <NA>
    ## 3958     6        N            Player        <NA>
    ## 3959     4        N            Player        <NA>
    ## 3960     4        N            Player        <NA>
    ## 3961     4        N            Player        <NA>
    ## 3962     2        N            Player        <NA>
    ## 3963     1        N            Player        <NA>
    ## 3964     1        N            Player        <NA>
    ## 3965     0        N            Player        <NA>
    ## 3966     0        N            Player        <NA>
    ## 3967     0        N            Player        <NA>
    ## 3968     0        N            Player        <NA>
    ## 3969     0        N            Player        <NA>
    ## 3970     0        N            Player        <NA>
    ## 3971    NA        Y Pioneer/Executive        <NA>
    ## 3910   420        Y            Player        <NA>
    ## 3911   400        N            Player        <NA>
    ## 3912   397        N            Player        <NA>
    ## 3913   282        N            Player        <NA>
    ## 3914   278        N            Player        <NA>
    ## 3915   255        N            Player        <NA>
    ## 3916   195        N            Player        <NA>
    ## 3917   164        N            Player        <NA>
    ## 3918   128        N            Player        <NA>
    ## 3919   121        N            Player        <NA>
    ## 3920   116        N            Player        <NA>
    ## 3921    87        N            Player        <NA>
    ## 3922    82        N            Player        <NA>
    ## 3923    63        N            Player        <NA>
    ## 3924    33        N            Player        <NA>
    ## 3925    22        N            Player        <NA>
    ## 3926     7        N            Player        <NA>
    ## 3927     2        N            Player        <NA>
    ## 3928     2        N            Player        <NA>
    ## 3929     1        N            Player        <NA>
    ## 3930     1        N            Player        <NA>
    ## 3931     1        N            Player        <NA>
    ## 3932     0        N            Player        <NA>
    ## 3933     0        N            Player        <NA>
    ## 3934     0        N            Player        <NA>
    ## 3935     0        N            Player        <NA>
    ## 3936    NA        Y           Manager        <NA>
    ## 3937    NA        Y            Umpire        <NA>
    ## 3886   511        Y            Player        <NA>
    ## 3887   412        Y            Player        <NA>
    ## 3888   361        N            Player        <NA>
    ## 3889   338        N            Player        <NA>
    ## 3890   240        N            Player        <NA>
    ## 3891   237        N            Player        <NA>
    ## 3892   171        N            Player        <NA>
    ## 3893   122        N            Player        <NA>
    ## 3894   118        N            Player        <NA>
    ## 3895    94        N            Player        <NA>
    ## 3896    81        N            Player        <NA>
    ## 3897    64        N            Player        <NA>
    ## 3898    62        N            Player        <NA>
    ## 3899    32        N            Player        <NA>
    ## 3900    22        N            Player        <NA>
    ## 3901    21        N            Player        <NA>
    ## 3902     7        N            Player        <NA>
    ## 3903     6        N            Player        <NA>
    ## 3904     2        N            Player        <NA>
    ## 3905     1        N            Player        <NA>
    ## 3906     0        N            Player        <NA>
    ## 3907     0        N            Player        <NA>
    ## 3908     0        N            Player        <NA>
    ## 3909    NA        Y            Player        <NA>
    ## 3856   466        Y            Player        <NA>
    ## 3857   392        N            Player        <NA>
    ## 3858   358        N            Player        <NA>
    ## 3859   336        N            Player        <NA>
    ## 3860   235        N            Player        <NA>
    ## 3861   233        N            Player        <NA>
    ## 3862   158        N            Player        <NA>
    ## 3863   132        N            Player        <NA>
    ## 3864   128        N            Player        <NA>
    ## 3865    99        N            Player        <NA>
    ## 3866    88        N            Player        <NA>
    ## 3867    86        N            Player        <NA>
    ## 3868    82        N            Player        <NA>
    ## 3869    75        N            Player        <NA>
    ## 3870    28        N            Player        <NA>
    ## 3871     2        N            Player        <NA>
    ## 3872     2        N            Player        <NA>
    ## 3873     2        N            Player        <NA>
    ## 3874     1        N            Player        <NA>
    ## 3875     1        N            Player        <NA>
    ## 3876     1        N            Player        <NA>
    ## 3877     1        N            Player        <NA>
    ## 3878     1        N            Player        <NA>
    ## 3879     0        N            Player        <NA>
    ## 3880     0        N            Player        <NA>
    ## 3881    NA        Y Pioneer/Executive        <NA>
    ## 3882    NA        Y Pioneer/Executive        <NA>
    ## 3883    NA        Y Pioneer/Executive        <NA>
    ## 3884    NA        Y           Manager        <NA>
    ## 3885    NA        Y           Manager        <NA>
    ## 3824   537        Y            Player        <NA>
    ## 3825   532        Y            Player        <NA>
    ## 3826   388        N            Player        <NA>
    ## 3827   346        N            Player        <NA>
    ## 3828   309        N            Player        <NA>
    ## 3829   260        N            Player        <NA>
    ## 3830   217        N            Player        <NA>
    ## 3831   202        N            Player        <NA>
    ## 3832   128        N            Player        <NA>
    ## 3833   125        N            Player        <NA>
    ## 3834   115        N            Player        <NA>
    ## 3835    74        N            Player        <NA>
    ## 3836    73        N            Player        <NA>
    ## 3837    62        N            Player        <NA>
    ## 3838    54        N            Player        <NA>
    ## 3839    50        N            Player        <NA>
    ## 3840    29        N            Player        <NA>
    ## 3841    24        N            Player        <NA>
    ## 3842    19        N            Player        <NA>
    ## 3843    12        N            Player        <NA>
    ## 3844     7        N            Player        <NA>
    ## 3845     6        N            Player        <NA>
    ## 3846     4        N            Player        <NA>
    ## 3847     3        N            Player        <NA>
    ## 3848     3        N            Player        <NA>
    ## 3849     2        N            Player        <NA>
    ## 3850     2        N            Player        <NA>
    ## 3851     1        N            Player        <NA>
    ## 3852     0        N            Player        <NA>
    ## 3853     0        N            Player        <NA>
    ## 3854     0        N            Player        <NA>
    ## 3855     0        N            Player        <NA>
    ## 3778   400        Y            Player        <NA>
    ## 3779   337        N            Player        <NA>
    ## 3780   336        N            Player        <NA>
    ## 3781   317        N            Player        <NA>
    ## 3782   277        N            Player        <NA>
    ## 3783   234        N            Player        <NA>
    ## 3784   214        N            Player        <NA>
    ## 3785   154        N            Player        <NA>
    ## 3786   135        N            Player        <NA>
    ## 3787    92        N            Player        <NA>
    ## 3788    76        N            Player        <NA>
    ## 3789    65        N            Player        <NA>
    ## 3790    64        N            Player        <NA>
    ## 3791    58        N            Player        <NA>
    ## 3792    56        N            Player        <NA>
    ## 3793    40        N            Player        <NA>
    ## 3794    23        N            Player        <NA>
    ## 3795    17        N            Player        <NA>
    ## 3796    12        N            Player        <NA>
    ## 3797     5        N            Player        <NA>
    ## 3798     5        N            Player        <NA>
    ## 3799     4        N            Player        <NA>
    ## 3800     4        N            Player        <NA>
    ## 3801     3        N            Player        <NA>
    ## 3802     2        N            Player        <NA>
    ## 3803     2        N            Player        <NA>
    ## 3804     1        N            Player        <NA>
    ## 3805     0        N            Player        <NA>
    ## 3806     0        N            Player        <NA>
    ## 3807    NA        Y            Player        <NA>
    ## 3808    NA        Y            Player        <NA>
    ## 3809    NA        Y            Player        <NA>
    ## 3810    NA        Y            Player        <NA>
    ## 3811    NA        Y            Player        <NA>
    ## 3812    NA        Y            Player        <NA>
    ## 3813    NA        Y Pioneer/Executive        <NA>
    ## 3814    NA        Y            Player        <NA>
    ## 3815    NA        Y Pioneer/Executive        <NA>
    ## 3816    NA        Y Pioneer/Executive        <NA>
    ## 3817    NA        Y            Player        <NA>
    ## 3818    NA        Y            Player        <NA>
    ## 3819    NA        Y            Player        <NA>
    ## 3820    NA        Y            Player        <NA>
    ## 3821    NA        Y Pioneer/Executive        <NA>
    ## 3822    NA        Y Pioneer/Executive        <NA>
    ## 3823    NA        Y            Player        <NA>
    ## 3751   474        Y            Player        <NA>
    ## 3752   393        Y            Player        <NA>
    ## 3753   344        N            Player        <NA>
    ## 3754   307        N            Player        <NA>
    ## 3755   285        N            Player        <NA>
    ## 3756   270        N            Player        <NA>
    ## 3757   211        N            Player        <NA>
    ## 3758   200        N            Player        <NA>
    ## 3759   172        N            Player        <NA>
    ## 3760   123        N            Player        <NA>
    ## 3761   106        N            Player        <NA>
    ## 3762    87        N            Player        <NA>
    ## 3763    65        N            Player        <NA>
    ## 3764    59        N            Player        <NA>
    ## 3765    55        N            Player        <NA>
    ## 3766    54        N            Player        <NA>
    ## 3767    26        N            Player        <NA>
    ## 3768    13        N            Player        <NA>
    ## 3769     6        N            Player        <NA>
    ## 3770     4        N            Player        <NA>
    ## 3771     3        N            Player        <NA>
    ## 3772     2        N            Player        <NA>
    ## 3773     2        N            Player        <NA>
    ## 3774     1        N            Player        <NA>
    ## 3775     1        N            Player        <NA>
    ## 3776     0        N            Player        <NA>
    ## 3777     0        N            Player        <NA>
    ## 3719   431        Y            Player        <NA>
    ## 3720   421        Y            Player        <NA>
    ## 3721   309        N            Player        <NA>
    ## 3722   301        N            Player        <NA>
    ## 3723   276        N            Player        <NA>
    ## 3724   253        N            Player        <NA>
    ## 3725   206        N            Player        <NA>
    ## 3726   185        N            Player        <NA>
    ## 3727   179        N            Player        <NA>
    ## 3728   133        N            Player        <NA>
    ## 3729   123        N            Player        <NA>
    ## 3730   111        N            Player        <NA>
    ## 3731    70        N            Player        <NA>
    ## 3732    65        N            Player        <NA>
    ## 3733    57        N            Player        <NA>
    ## 3734    53        N            Player        <NA>
    ## 3735    43        N            Player        <NA>
    ## 3736    22        N            Player        <NA>
    ## 3737    19        N            Player        <NA>
    ## 3738    19        N            Player        <NA>
    ## 3739    16        N            Player        <NA>
    ## 3740     7        N            Player        <NA>
    ## 3741     3        N            Player        <NA>
    ## 3742     3        N            Player        <NA>
    ## 3743     2        N            Player        <NA>
    ## 3744     2        N            Player        <NA>
    ## 3745     2        N            Player        <NA>
    ## 3746     1        N            Player        <NA>
    ## 3747     1        N            Player        <NA>
    ## 3748     1        N            Player        <NA>
    ## 3749     0        N            Player        <NA>
    ## 3750     0        N            Player        <NA>
    ## 3686   423        Y            Player        <NA>
    ## 3687   387        Y            Player        <NA>
    ## 3688   266        N            Player        <NA>
    ## 3689   259        N            Player        <NA>
    ## 3690   248        N            Player        <NA>
    ## 3691   244        N            Player        <NA>
    ## 3692   210        N            Player        <NA>
    ## 3693   209        N            Player        <NA>
    ## 3694   145        N            Player        <NA>
    ## 3695   138        N            Player        <NA>
    ## 3696   130        N            Player        <NA>
    ## 3697   116        N            Player        <NA>
    ## 3698   113        N            Player        <NA>
    ## 3699    70        N            Player        <NA>
    ## 3700    68        N            Player        <NA>
    ## 3701    58        N            Player        <NA>
    ## 3702    55        N            Player        <NA>
    ## 3703    51        N            Player        <NA>
    ## 3704    31        N            Player        <NA>
    ## 3705    30        N            Player        <NA>
    ## 3706     7        N            Player        <NA>
    ## 3707     3        N            Player        <NA>
    ## 3708     2        N            Player        <NA>
    ## 3709     2        N            Player        <NA>
    ## 3710     2        N            Player        <NA>
    ## 3711     2        N            Player        <NA>
    ## 3712     1        N            Player        <NA>
    ## 3713     1        N            Player        <NA>
    ## 3714     1        N            Player        <NA>
    ## 3715     0        N            Player        <NA>
    ## 3716     0        N            Player        <NA>
    ## 3717     0        N            Player        <NA>
    ## 3718     0        N            Player        <NA>
    ## 3658   433        Y            Player        <NA>
    ## 3659   343        N            Player        <NA>
    ## 3660   260        N            Player        <NA>
    ## 3661   238        N            Player        <NA>
    ## 3662   214        N            Player        <NA>
    ## 3663   203        N            Player        <NA>
    ## 3664   134        N            Player        <NA>
    ## 3665   127        N            Player        <NA>
    ## 3666   124        N            Player        <NA>
    ## 3667   109        N            Player        <NA>
    ## 3668    97        N            Player        <NA>
    ## 3669    96        N            Player        <NA>
    ## 3670    85        N            Player        <NA>
    ## 3671    74        N            Player        <NA>
    ## 3672    70        N            Player        <NA>
    ## 3673    66        N            Player        <NA>
    ## 3674    56        N            Player        <NA>
    ## 3675    29        N            Player        <NA>
    ## 3676    23        N            Player        <NA>
    ## 3677    23        N            Player        <NA>
    ## 3678     2        N            Player        <NA>
    ## 3679     2        N            Player        <NA>
    ## 3680     1        N            Player        <NA>
    ## 3681     1        N            Player        <NA>
    ## 3682     0        N            Player        <NA>
    ## 3683     0        N            Player        <NA>
    ## 3684     0        N            Player        <NA>
    ## 3685     0        N            Player        <NA>
    ## 3624   435        Y            Player        <NA>
    ## 3625   423        Y            Player        <NA>
    ## 3626   334        N            Player        <NA>
    ## 3627   298        N            Player        <NA>
    ## 3628   245        N            Player        <NA>
    ## 3629   228        N            Player        <NA>
    ## 3630   176        N            Player        <NA>
    ## 3631   146        N            Player        <NA>
    ## 3632   145        N            Player        <NA>
    ## 3633   135        N            Player        <NA>
    ## 3634   121        N            Player        <NA>
    ## 3635   101        N            Player        <NA>
    ## 3636    93        N            Player        <NA>
    ## 3637    84        N            Player        <NA>
    ## 3638    74        N            Player        <NA>
    ## 3639    63        N            Player        <NA>
    ## 3640    41        N            Player        <NA>
    ## 3641    38        N            Player        <NA>
    ## 3642    27        N            Player        <NA>
    ## 3643    15        N            Player        <NA>
    ## 3644    13        N            Player        <NA>
    ## 3645     9        N            Player        <NA>
    ## 3646     6        N            Player        <NA>
    ## 3647     2        N            Player        <NA>
    ## 3648     1        N            Player        <NA>
    ## 3649     1        N            Player        <NA>
    ## 3650     1        N            Player        <NA>
    ## 3651     1        N            Player        <NA>
    ## 3652     1        N            Player        <NA>
    ## 3653     1        N            Player        <NA>
    ## 3654     0        N            Player        <NA>
    ## 3655     0        N            Player        <NA>
    ## 3656    NA        Y            Player        <NA>
    ## 3657    NA        Y            Player        <NA>
    ## 3591   397        Y            Player        <NA>
    ## 3592   385        Y            Player        <NA>
    ## 3593   257        N            Player        <NA>
    ## 3594   248        N            Player        <NA>
    ## 3595   192        N            Player        <NA>
    ## 3596   166        N            Player        <NA>
    ## 3597   160        N            Player        <NA>
    ## 3598   135        N            Player        <NA>
    ## 3599   125        N            Player        <NA>
    ## 3600   116        N            Player        <NA>
    ## 3601   111        N            Player        <NA>
    ## 3602   104        N            Player        <NA>
    ## 3603    87        N            Player        <NA>
    ## 3604    86        N            Player        <NA>
    ## 3605    67        N            Player        <NA>
    ## 3606    52        N            Player        <NA>
    ## 3607    44        N            Player        <NA>
    ## 3608    24        N            Player        <NA>
    ## 3609    21        N            Player        <NA>
    ## 3610    10        N            Player        <NA>
    ## 3611     9        N            Player        <NA>
    ## 3612     5        N            Player        <NA>
    ## 3613     4        N            Player        <NA>
    ## 3614     2        N            Player        <NA>
    ## 3615     2        N            Player        <NA>
    ## 3616     1        N            Player        <NA>
    ## 3617     1        N            Player        <NA>
    ## 3618     1        N            Player        <NA>
    ## 3619     1        N            Player        <NA>
    ## 3620     0        N            Player        <NA>
    ## 3621    NA        Y           Manager        <NA>
    ## 3622    NA        Y            Player        <NA>
    ## 3623    NA        Y            Player        <NA>
    ## 3559   491        Y            Player        <NA>
    ## 3560   488        Y            Player        <NA>
    ## 3561   385        Y            Player        <NA>
    ## 3562   330        N            Player        <NA>
    ## 3563   302        N            Player        <NA>
    ## 3564   168        N            Player        <NA>
    ## 3565   150        N            Player        <NA>
    ## 3566   146        N            Player        <NA>
    ## 3567   121        N            Player        <NA>
    ## 3568   100        N            Player        <NA>
    ## 3569    96        N            Player        <NA>
    ## 3570    93        N            Player        <NA>
    ## 3571    80        N            Player        <NA>
    ## 3572    73        N            Player        <NA>
    ## 3573    70        N            Player        <NA>
    ## 3574    59        N            Player        <NA>
    ## 3575    53        N            Player        <NA>
    ## 3576    34        N            Player        <NA>
    ## 3577    31        N            Player        <NA>
    ## 3578    27        N            Player        <NA>
    ## 3579    26        N            Player        <NA>
    ## 3580    18        N            Player        <NA>
    ## 3581     6        N            Player        <NA>
    ## 3582     1        N            Player        <NA>
    ## 3583     0        N            Player        <NA>
    ## 3584     0        N            Player        <NA>
    ## 3585     0        N            Player        <NA>
    ## 3586     0        N            Player        <NA>
    ## 3587    NA        Y            Player        <NA>
    ## 3588    NA        Y            Umpire        <NA>
    ## 3589    NA        Y           Manager        <NA>
    ## 3590    NA        Y            Player        <NA>
    ## 3529   386        Y            Player        <NA>
    ## 3530   321        N            Player        <NA>
    ## 3531   204        N            Player        <NA>
    ## 3532   203        N            Player        <NA>
    ## 3533   200        N            Player        <NA>
    ## 3534   195        N            Player        <NA>
    ## 3535   147        N            Player        <NA>
    ## 3536   129        N            Player        <NA>
    ## 3537   129        N            Player        <NA>
    ## 3538   116        N            Player        <NA>
    ## 3539    83        N            Player        <NA>
    ## 3540    80        N            Player        <NA>
    ## 3541    76        N            Player        <NA>
    ## 3542    62        N            Player        <NA>
    ## 3543    51        N            Player        <NA>
    ## 3544    49        N            Player        <NA>
    ## 3545    39        N            Player        <NA>
    ## 3546    37        N            Player        <NA>
    ## 3547    26        N            Player        <NA>
    ## 3548     7        N            Player        <NA>
    ## 3549     6        N            Player        <NA>
    ## 3550     5        N            Player        <NA>
    ## 3551     3        N            Player        <NA>
    ## 3552     2        N            Player        <NA>
    ## 3553     2        N            Player        <NA>
    ## 3554     1        N            Player        <NA>
    ## 3555    NA        Y            Player        <NA>
    ## 3556    NA        Y            Player        <NA>
    ## 3557    NA        Y Pioneer/Executive        <NA>
    ## 3558    NA        Y            Player        <NA>
    ## 3496   380        Y            Player        <NA>
    ## 3497   346        N            Player        <NA>
    ## 3498   312        N            Player        <NA>
    ## 3499   186        N            Player        <NA>
    ## 3500   178        N            Player        <NA>
    ## 3501   167        N            Player        <NA>
    ## 3502   130        N            Player        <NA>
    ## 3503   107        N            Player        <NA>
    ## 3504   105        N            Player        <NA>
    ## 3505    97        N            Player        <NA>
    ## 3506    84        N            Player        <NA>
    ## 3507    83        N            Player        <NA>
    ## 3508    79        N            Player        <NA>
    ## 3509    60        N            Player        <NA>
    ## 3510    53        N            Player        <NA>
    ## 3511    45        N            Player        <NA>
    ## 3512    34        N            Player        <NA>
    ## 3513    31        N            Player        <NA>
    ## 3514    28        N            Player        <NA>
    ## 3515    28        N            Player        <NA>
    ## 3516    22        N            Player        <NA>
    ## 3517    22        N            Player        <NA>
    ## 3518    22        N            Player        <NA>
    ## 3519    20        N            Player        <NA>
    ## 3520    18        N            Player        <NA>
    ## 3521     2        N            Player        <NA>
    ## 3522     2        N            Player        <NA>
    ## 3523     2        N            Player        <NA>
    ## 3524     1        N            Player        <NA>
    ## 3525     1        N            Player        <NA>
    ## 3526    NA        Y            Player        <NA>
    ## 3527    NA        Y           Manager        <NA>
    ## 3528    NA        Y            Player        <NA>
    ## 3457   321        N            Player        <NA>
    ## 3458   309        N            Player        <NA>
    ## 3459   300        N            Player        <NA>
    ## 3460   175        N            Player        <NA>
    ## 3461   174        N            Player        <NA>
    ## 3462   170        N            Player        <NA>
    ## 3463   166        N            Player        <NA>
    ## 3464   137        N            Player        <NA>
    ## 3465   102        N            Player        <NA>
    ## 3466    91        N            Player        <NA>
    ## 3467    89        N            Player        <NA>
    ## 3468    71        N            Player        <NA>
    ## 3469    64        N            Player        <NA>
    ## 3470    63        N            Player        <NA>
    ## 3471    62        N            Player        <NA>
    ## 3472    51        N            Player        <NA>
    ## 3473    50        N            Player        <NA>
    ## 3474    37        N            Player        <NA>
    ## 3475    37        N            Player        <NA>
    ## 3476    36        N            Player        <NA>
    ## 3477    33        N            Player        <NA>
    ## 3478    26        N            Player        <NA>
    ## 3479    24        N            Player        <NA>
    ## 3480    24        N            Player        <NA>
    ## 3481    24        N            Player        <NA>
    ## 3482    18        N            Player        <NA>
    ## 3483    18        N            Player        <NA>
    ## 3484    10        N            Player        <NA>
    ## 3485     2        N            Player        <NA>
    ## 3486     2        N            Player        <NA>
    ## 3487     1        N            Player        <NA>
    ## 3488     0        N            Player        <NA>
    ## 3489     0        N            Player        <NA>
    ## 3490     0        N            Player        <NA>
    ## 3491     0        N            Player        <NA>
    ## 3492    NA        Y            Player        <NA>
    ## 3493    NA        Y            Player        <NA>
    ## 3494    NA        Y           Manager        <NA>
    ## 3495    NA        Y           Manager        <NA>
    ## 3414   444        Y            Player        <NA>
    ## 3415   286        N            Player        <NA>
    ## 3416   264        N            Player        <NA>
    ## 3417   259        N            Player        <NA>
    ## 3418   196        N            Player        <NA>
    ## 3419   149        N            Player        <NA>
    ## 3420   139        N            Player        <NA>
    ## 3421   137        N            Player        <NA>
    ## 3422   137        N            Player        <NA>
    ## 3423   100        N            Player        <NA>
    ## 3424    98        N            Player        <NA>
    ## 3425    72        N            Player        <NA>
    ## 3426    66        N            Player        <NA>
    ## 3427    59        N            Player        <NA>
    ## 3428    50        N            Player        <NA>
    ## 3429    45        N            Player        <NA>
    ## 3430    43        N            Player        <NA>
    ## 3431    35        N            Player        <NA>
    ## 3432    32        N            Player        <NA>
    ## 3433    30        N            Player        <NA>
    ## 3434    28        N            Player        <NA>
    ## 3435    26        N            Player        <NA>
    ## 3436    26        N            Player        <NA>
    ## 3437    25        N            Player        <NA>
    ## 3438    23        N            Player        <NA>
    ## 3439    19        N            Player        <NA>
    ## 3440    12        N            Player        <NA>
    ## 3441     8        N            Player        <NA>
    ## 3442     8        N            Player        <NA>
    ## 3443     6        N            Player        <NA>
    ## 3444     2        N            Player        <NA>
    ## 3445     2        N            Player        <NA>
    ## 3446     1        N            Player        <NA>
    ## 3447     1        N            Player        <NA>
    ## 3448     1        N            Player        <NA>
    ## 3449     0        N            Player        <NA>
    ## 3450     0        N            Player        <NA>
    ## 3451     0        N            Player        <NA>
    ## 3452     0        N            Player        <NA>
    ## 3453    NA        Y            Player        <NA>
    ## 3454    NA        Y            Player        <NA>
    ## 3455    NA        Y Pioneer/Executive        <NA>
    ## 3456    NA        Y            Player        <NA>
    ## 3373   436        Y            Player        <NA>
    ## 3374   335        N            Player        <NA>
    ## 3375   273        N            Player        <NA>
    ## 3376   263        N            Player        <NA>
    ## 3377   259        N            Player        <NA>
    ## 3378   166        N            Player        <NA>
    ## 3379   158        N            Player        <NA>
    ## 3380   150        N            Player        <NA>
    ## 3381   109        N            Player        <NA>
    ## 3382    98        N            Player        <NA>
    ## 3383    66        N            Player        <NA>
    ## 3384    54        N            Player        <NA>
    ## 3385    53        N            Player        <NA>
    ## 3386    46        N            Player        <NA>
    ## 3387    45        N            Player        <NA>
    ## 3388    42        N            Player        <NA>
    ## 3389    39        N            Player        <NA>
    ## 3390    38        N            Player        <NA>
    ## 3391    37        N            Player        <NA>
    ## 3392    36        N            Player        <NA>
    ## 3393    31        N            Player        <NA>
    ## 3394    31        N            Player        <NA>
    ## 3395    24        N            Player        <NA>
    ## 3396    23        N            Player        <NA>
    ## 3397    19        N            Player        <NA>
    ## 3398    17        N            Player        <NA>
    ## 3399    16        N            Player        <NA>
    ## 3400    14        N            Player        <NA>
    ## 3401    12        N            Player        <NA>
    ## 3402     6        N            Player        <NA>
    ## 3403     2        N            Player        <NA>
    ## 3404     2        N            Player        <NA>
    ## 3405     2        N            Player        <NA>
    ## 3406     1        N            Player        <NA>
    ## 3407     0        N            Player        <NA>
    ## 3408     0        N            Player        <NA>
    ## 3409     0        N            Player        <NA>
    ## 3410     0        N            Player        <NA>
    ## 3411     0        N            Player        <NA>
    ## 3412    NA        Y           Manager        <NA>
    ## 3413    NA        Y            Player        <NA>
    ## 3340   396        Y            Player        <NA>
    ## 3341   278        N            Player        <NA>
    ## 3342   252        N            Player        <NA>
    ## 3343   233        N            Player        <NA>
    ## 3344   176        N            Player        <NA>
    ## 3345   157        N            Player        <NA>
    ## 3346   155        N            Player        <NA>
    ## 3347   125        N            Player        <NA>
    ## 3348    70        N            Player        <NA>
    ## 3349    69        N            Player        <NA>
    ## 3350    67        N            Player        <NA>
    ## 3351    63        N            Player        <NA>
    ## 3352    62        N            Player        <NA>
    ## 3353    45        N            Player        <NA>
    ## 3354    43        N            Player        <NA>
    ## 3355    40        N            Player        <NA>
    ## 3356    38        N            Player        <NA>
    ## 3357    37        N            Player        <NA>
    ## 3358    36        N            Player        <NA>
    ## 3359    32        N            Player        <NA>
    ## 3360    29        N            Player        <NA>
    ## 3361    19        N            Player        <NA>
    ## 3362    14        N            Player        <NA>
    ## 3363     8        N            Player        <NA>
    ## 3364     2        N            Player        <NA>
    ## 3365     2        N            Player        <NA>
    ## 3366     2        N            Player        <NA>
    ## 3367     1        N            Player        <NA>
    ## 3368     0        N            Player        <NA>
    ## 3369     0        N            Player        <NA>
    ## 3370     0        N            Player        <NA>
    ## 3371     0        N            Player        <NA>
    ## 3372     0        N            Player        <NA>
    ## 3301   425        Y            Player        <NA>
    ## 3302   349        Y            Player        <NA>
    ## 3303   246        N            Player        <NA>
    ## 3304   215        N            Player        <NA>
    ## 3305   182        N            Player        <NA>
    ## 3306   175        N            Player        <NA>
    ## 3307   136        N            Player        <NA>
    ## 3308   114        N            Player        <NA>
    ## 3309   110        N            Player        <NA>
    ## 3310    71        N            Player        <NA>
    ## 3311    69        N            Player        <NA>
    ## 3312    69        N            Player        <NA>
    ## 3313    62        N            Player        <NA>
    ## 3314    50        N            Player        <NA>
    ## 3315    45        N            Player        <NA>
    ## 3316    42        N            Player        <NA>
    ## 3317    41        N            Player        <NA>
    ## 3318    40        N            Player        <NA>
    ## 3319    36        N            Player        <NA>
    ## 3320    32        N            Player        <NA>
    ## 3321    26        N            Player        <NA>
    ## 3322    24        N            Player        <NA>
    ## 3323    23        N            Player        <NA>
    ## 3324    11        N            Player        <NA>
    ## 3325     4        N            Player        <NA>
    ## 3326     3        N            Player        <NA>
    ## 3327     3        N            Player        <NA>
    ## 3328     2        N            Player        <NA>
    ## 3329     2        N            Player        <NA>
    ## 3330     1        N            Player        <NA>
    ## 3331     1        N            Player        <NA>
    ## 3332     0        N            Player        <NA>
    ## 3333     0        N            Player        <NA>
    ## 3334     0        N            Player        <NA>
    ## 3335     0        N            Player        <NA>
    ## 3336     0        N            Player        <NA>
    ## 3337     0        N            Player        <NA>
    ## 3338    NA        Y            Umpire        <NA>
    ## 3339    NA        Y            Player        <NA>
    ## 3254   401        Y            Player        <NA>
    ## 3255   342        Y            Player        <NA>
    ## 3256   334        Y            Player        <NA>
    ## 3257   291        N            Player        <NA>
    ## 3258   282        N            Player        <NA>
    ## 3259   192        N            Player        <NA>
    ## 3260   160        N            Player        <NA>
    ## 3261   142        N            Player        <NA>
    ## 3262   116        N            Player        <NA>
    ## 3263   100        N            Player        <NA>
    ## 3264    62        N            Player        <NA>
    ## 3265    61        N            Player        <NA>
    ## 3266    59        N            Player        <NA>
    ## 3267    58        N            Player        <NA>
    ## 3268    41        N            Player        <NA>
    ## 3269    39        N            Player        <NA>
    ## 3270    38        N            Player        <NA>
    ## 3271    33        N            Player        <NA>
    ## 3272    32        N            Player        <NA>
    ## 3273    30        N            Player        <NA>
    ## 3274    28        N            Player        <NA>
    ## 3275    28        N            Player        <NA>
    ## 3276    23        N            Player        <NA>
    ## 3277    19        N            Player        <NA>
    ## 3278    15        N            Player        <NA>
    ## 3279    11        N            Player        <NA>
    ## 3280     4        N            Player        <NA>
    ## 3281     1        N            Player        <NA>
    ## 3282     1        N            Player        <NA>
    ## 3283     1        N            Player        <NA>
    ## 3284     1        N            Player        <NA>
    ## 3285     1        N            Player        <NA>
    ## 3286     1        N            Player        <NA>
    ## 3287     1        N            Player        <NA>
    ## 3288     0        N            Player        <NA>
    ## 3289     0        N            Player        <NA>
    ## 3290     0        N            Player        <NA>
    ## 3291     0        N            Player        <NA>
    ## 3292     0        N            Player        <NA>
    ## 3293     0        N            Player        <NA>
    ## 3294     0        N            Player        <NA>
    ## 3295     0        N            Player        <NA>
    ## 3296     0        N            Player        <NA>
    ## 3297     0        N            Player        <NA>
    ## 3298     0        N            Player        <NA>
    ## 3299    NA        Y            Player        <NA>
    ## 3300    NA        Y Pioneer/Executive        <NA>
    ## 3210   411        Y            Player        <NA>
    ## 3211   363        Y            Player        <NA>
    ## 3212   320        N            Player        <NA>
    ## 3213   296        N            Player        <NA>
    ## 3214   257        N            Player        <NA>
    ## 3215   211        N            Player        <NA>
    ## 3216   142        N            Player        <NA>
    ## 3217   131        N            Player        <NA>
    ## 3218   107        N            Player        <NA>
    ## 3219    96        N            Player        <NA>
    ## 3220    95        N            Player        <NA>
    ## 3221    79        N            Player        <NA>
    ## 3222    78        N            Player        <NA>
    ## 3223    58        N            Player        <NA>
    ## 3224    55        N            Player        <NA>
    ## 3225    51        N            Player        <NA>
    ## 3226    50        N            Player        <NA>
    ## 3227    42        N            Player        <NA>
    ## 3228    36        N            Player        <NA>
    ## 3229    35        N            Player        <NA>
    ## 3230    33        N            Player        <NA>
    ## 3231    30        N            Player        <NA>
    ## 3232    27        N            Player        <NA>
    ## 3233    25        N            Player        <NA>
    ## 3234     6        N            Player        <NA>
    ## 3235     3        N            Player        <NA>
    ## 3236     3        N            Player        <NA>
    ## 3237     2        N            Player        <NA>
    ## 3238     2        N            Player        <NA>
    ## 3239     2        N            Player        <NA>
    ## 3240     1        N            Player        <NA>
    ## 3241     1        N            Player        <NA>
    ## 3242     1        N            Player        <NA>
    ## 3243     1        N            Player        <NA>
    ## 3244     0        N            Player        <NA>
    ## 3245     0        N            Player        <NA>
    ## 3246     0        N            Player        <NA>
    ## 3247     0        N            Player        <NA>
    ## 3248     0        N            Player        <NA>
    ## 3249     0        N            Player        <NA>
    ## 3250     0        N            Player        <NA>
    ## 3251     0        N            Player        <NA>
    ## 3252     0        N            Player        <NA>
    ## 3253     0        N            Player        <NA>
    ## 3167   431        Y            Player        <NA>
    ## 3168   423        Y            Player        <NA>
    ## 3169   304        N            Player        <NA>
    ## 3170   283        N            Player        <NA>
    ## 3171   234        N            Player        <NA>
    ## 3172   176        N            Player        <NA>
    ## 3173   135        N            Player        <NA>
    ## 3174   134        N            Player        <NA>
    ## 3175   115        N            Player        <NA>
    ## 3176    95        N            Player        <NA>
    ## 3177    87        N            Player        <NA>
    ## 3178    75        N            Player        <NA>
    ## 3179    62        N            Player        <NA>
    ## 3180    59        N            Player        <NA>
    ## 3181    47        N            Player        <NA>
    ## 3182    47        N            Player        <NA>
    ## 3183    47        N            Player        <NA>
    ## 3184    40        N            Player        <NA>
    ## 3185    35        N            Player        <NA>
    ## 3186    33        N            Player        <NA>
    ## 3187    31        N            Player        <NA>
    ## 3188    29        N            Player        <NA>
    ## 3189    27        N            Player        <NA>
    ## 3190    25        N            Player        <NA>
    ## 3191    14        N            Player        <NA>
    ## 3192    14        N            Player        <NA>
    ## 3193     9        N            Player        <NA>
    ## 3194     3        N            Player        <NA>
    ## 3195     1        N            Player        <NA>
    ## 3196     1        N            Player        <NA>
    ## 3197     0        N            Player        <NA>
    ## 3198     0        N            Player        <NA>
    ## 3199     0        N            Player        <NA>
    ## 3200     0        N            Player        <NA>
    ## 3201     0        N            Player        <NA>
    ## 3202     0        N            Player        <NA>
    ## 3203     0        N            Player        <NA>
    ## 3204     0        N            Player        <NA>
    ## 3205     0        N            Player        <NA>
    ## 3206     0        N            Player        <NA>
    ## 3207     0        N            Player        <NA>
    ## 3208    NA        Y            Umpire        <NA>
    ## 3209    NA        Y            Player        <NA>
    ## 3123   352        Y            Player        <NA>
    ## 3124   317        N            Player        <NA>
    ## 3125   202        N            Player        <NA>
    ## 3126   199        N            Player        <NA>
    ## 3127   184        N            Player        <NA>
    ## 3128   168        N            Player        <NA>
    ## 3129   143        N            Player        <NA>
    ## 3130   132        N            Player        <NA>
    ## 3131   127        N            Player        <NA>
    ## 3132   109        N            Player        <NA>
    ## 3133   109        N            Player        <NA>
    ## 3134   108        N            Player        <NA>
    ## 3135    90        N            Player        <NA>
    ## 3136    79        N            Player        <NA>
    ## 3137    67        N            Player        <NA>
    ## 3138    60        N            Player        <NA>
    ## 3139    56        N            Player        <NA>
    ## 3140    53        N            Player        <NA>
    ## 3141    52        N            Player        <NA>
    ## 3142    48        N            Player        <NA>
    ## 3143    32        N            Player        <NA>
    ## 3144    31        N            Player        <NA>
    ## 3145    30        N            Player        <NA>
    ## 3146    27        N            Player        <NA>
    ## 3147    18        N            Player        <NA>
    ## 3148    16        N            Player        <NA>
    ## 3149     3        N            Player        <NA>
    ## 3150     3        N            Player        <NA>
    ## 3151     3        N            Player        <NA>
    ## 3152     1        N            Player        <NA>
    ## 3153     0        N            Player        <NA>
    ## 3154     0        N            Player        <NA>
    ## 3155     0        N            Player        <NA>
    ## 3156     0        N            Player        <NA>
    ## 3157     0        N            Player        <NA>
    ## 3158     0        N            Player        <NA>
    ## 3159     0        N            Player        <NA>
    ## 3160     0        N            Player        <NA>
    ## 3161     0        N            Player        <NA>
    ## 3162     0        N            Player        <NA>
    ## 3163     0        N            Player        <NA>
    ## 3164     0        N            Player        <NA>
    ## 3165     0        N            Player        <NA>
    ## 3166     0        N            Player        <NA>
    ## 3094   354        Y            Player        <NA>
    ## 3095   315        Y            Player        <NA>
    ## 3096   289        N            Player        <NA>
    ## 3097   179        N            Player        <NA>
    ## 3098   176        N            Player        <NA>
    ## 3099   160        N            Player        <NA>
    ## 3100   144        N            Player        <NA>
    ## 3101   125        N            Player        <NA>
    ## 3102   113        N            Player        <NA>
    ## 3103    96        N            Player        <NA>
    ## 3104    96        N            Player        <NA>
    ## 3105    84        N            Player        <NA>
    ## 3106    82        N            Player        <NA>
    ## 3107    78        N            Player        <NA>
    ## 3108    78        N            Player        <NA>
    ## 3109    55        N            Player        <NA>
    ## 3110    50        N            Player        <NA>
    ## 3111    48        N            Player        <NA>
    ## 3112    47        N            Player        <NA>
    ## 3113    44        N            Player        <NA>
    ## 3114    30        N            Player        <NA>
    ## 3115    28        N            Player        <NA>
    ## 3116    26        N            Player        <NA>
    ## 3117    24        N            Player        <NA>
    ## 3118     6        N            Player        <NA>
    ## 3119     3        N            Player        <NA>
    ## 3120     0        N            Player        <NA>
    ## 3121     0        N            Player        <NA>
    ## 3122    NA        Y            Player        <NA>
    ## 3051   346        Y            Player        <NA>
    ## 3052   315        N            Player        <NA>
    ## 3053   289        N            Player        <NA>
    ## 3054   279        N            Player        <NA>
    ## 3055   177        N            Player        <NA>
    ## 3056   154        N            Player        <NA>
    ## 3057   152        N            Player        <NA>
    ## 3058   144        N            Player        <NA>
    ## 3059   124        N            Player        <NA>
    ## 3060   100        N            Player        <NA>
    ## 3061    96        N            Player        <NA>
    ## 3062    95        N            Player        <NA>
    ## 3063    89        N            Player        <NA>
    ## 3064    86        N            Player        <NA>
    ## 3065    74        N            Player        <NA>
    ## 3066    64        N            Player        <NA>
    ## 3067    60        N            Player        <NA>
    ## 3068    51        N            Player        <NA>
    ## 3069    45        N            Player        <NA>
    ## 3070    43        N            Player        <NA>
    ## 3071    41        N            Player        <NA>
    ## 3072    35        N            Player        <NA>
    ## 3073    33        N            Player        <NA>
    ## 3074    23        N            Player        <NA>
    ## 3075    16        N            Player        <NA>
    ## 3076    12        N            Player        <NA>
    ## 3077    11        N            Player        <NA>
    ## 3078     8        N            Player        <NA>
    ## 3079     7        N            Player        <NA>
    ## 3080     5        N            Player        <NA>
    ## 3081     4        N            Player        <NA>
    ## 3082     3        N            Player        <NA>
    ## 3083     3        N            Player        <NA>
    ## 3084     2        N            Player        <NA>
    ## 3085     2        N            Player        <NA>
    ## 3086     1        N            Player        <NA>
    ## 3087     1        N            Player        <NA>
    ## 3088     1        N            Player        <NA>
    ## 3089     1        N            Player        <NA>
    ## 3090     0        N            Player        <NA>
    ## 3091     0        N            Player        <NA>
    ## 3092    NA        Y            Player        <NA>
    ## 3093    NA        Y            Player        <NA>
    ## 3008   331        Y            Player        <NA>
    ## 3009   315        Y            Player        <NA>
    ## 3010   295        N            Player        <NA>
    ## 3011   252        N            Player        <NA>
    ## 3012   214        N            Player        <NA>
    ## 3013   212        N            Player        <NA>
    ## 3014   128        N            Player        <NA>
    ## 3015   125        N            Player        <NA>
    ## 3016   114        N            Player        <NA>
    ## 3017   114        N            Player        <NA>
    ## 3018    93        N            Player        <NA>
    ## 3019    87        N            Player        <NA>
    ## 3020    82        N            Player        <NA>
    ## 3021    78        N            Player        <NA>
    ## 3022    68        N            Player        <NA>
    ## 3023    62        N            Player        <NA>
    ## 3024    54        N            Player        <NA>
    ## 3025    53        N            Player        <NA>
    ## 3026    44        N            Player        <NA>
    ## 3027    32        N            Player        <NA>
    ## 3028    32        N            Player        <NA>
    ## 3029    28        N            Player        <NA>
    ## 3030    28        N            Player        <NA>
    ## 3031    19        N            Player        <NA>
    ## 3032    16        N            Player        <NA>
    ## 3033    15        N            Player        <NA>
    ## 3034     7        N            Player        <NA>
    ## 3035     4        N            Player        <NA>
    ## 3036     3        N            Player        <NA>
    ## 3037     3        N            Player        <NA>
    ## 3038     3        N            Player        <NA>
    ## 3039     2        N            Player        <NA>
    ## 3040     2        N            Player        <NA>
    ## 3041     1        N            Player        <NA>
    ## 3042     1        N            Player        <NA>
    ## 3043     1        N            Player        <NA>
    ## 3044     0        N            Player        <NA>
    ## 3045     0        N            Player        <NA>
    ## 3046     0        N            Player        <NA>
    ## 3047     0        N            Player        <NA>
    ## 3048     0        N            Player        <NA>
    ## 3049    NA        Y            Player        <NA>
    ## 3050    NA        Y            Player        <NA>
    ## 2977   341        Y            Player        <NA>
    ## 2978   335        Y            Player        <NA>
    ## 2979   316        Y            Player        <NA>
    ## 2980   290        N            Player        <NA>
    ## 2981   246        N            Player        <NA>
    ## 2982   202        N            Player        <NA>
    ## 2983   201        N            Player        <NA>
    ## 2984   124        N            Player        <NA>
    ## 2985   124        N            Player        <NA>
    ## 2986   107        N            Player        <NA>
    ## 2987   106        N            Player        <NA>
    ## 2988   104        N            Player        <NA>
    ## 2989    97        N            Player        <NA>
    ## 2990    74        N            Player        <NA>
    ## 2991    65        N            Player        <NA>
    ## 2992    45        N            Player        <NA>
    ## 2993    45        N            Player        <NA>
    ## 2994    29        N            Player        <NA>
    ## 2995    25        N            Player        <NA>
    ## 2996    14        N            Player        <NA>
    ## 2997     4        N            Player        <NA>
    ## 2998     3        N            Player        <NA>
    ## 2999     3        N            Player        <NA>
    ## 3000     2        N            Player        <NA>
    ## 3001     1        N            Player        <NA>
    ## 3002     1        N            Player        <NA>
    ## 3003     1        N            Player        <NA>
    ## 3004     0        N            Player        <NA>
    ## 3005     0        N            Player        <NA>
    ## 3006    NA        Y            Player        <NA>
    ## 3007    NA        Y            Player        <NA>
    ## 2929   344        Y            Player        <NA>
    ## 2930   313        Y            Player        <NA>
    ## 2931   269        N            Player        <NA>
    ## 2932   252        N            Player        <NA>
    ## 2933   243        N            Player        <NA>
    ## 2934   242        N            Player        <NA>
    ## 2935   237        N            Player        <NA>
    ## 2936   173        N            Player        <NA>
    ## 2937   153        N            Player        <NA>
    ## 2938   146        N            Player        <NA>
    ## 2939   138        N            Player        <NA>
    ## 2940    77        N            Player        <NA>
    ## 2941    77        N            Player        <NA>
    ## 2942    75        N            Player        <NA>
    ## 2943    69        N            Player        <NA>
    ## 2944    59        N            Player        <NA>
    ## 2945    48        N            Player        <NA>
    ## 2946    43        N            Player        <NA>
    ## 2947    32        N            Player        <NA>
    ## 2948    32        N            Player        <NA>
    ## 2949    22        N            Player        <NA>
    ## 2950    20        N            Player        <NA>
    ## 2951    18        N            Player        <NA>
    ## 2952    14        N            Player        <NA>
    ## 2953    12        N            Player        <NA>
    ## 2954     7        N            Player        <NA>
    ## 2955     5        N            Player        <NA>
    ## 2956     2        N            Player        <NA>
    ## 2957     1        N            Player        <NA>
    ## 2958     1        N            Player        <NA>
    ## 2959     1        N            Player        <NA>
    ## 2960     0        N            Player        <NA>
    ## 2961     0        N            Player        <NA>
    ## 2962     0        N            Player        <NA>
    ## 2963     0        N            Player        <NA>
    ## 2964     0        N            Player        <NA>
    ## 2965     0        N            Player        <NA>
    ## 2966     0        N            Player        <NA>
    ## 2967     0        N            Player        <NA>
    ## 2968     0        N            Player        <NA>
    ## 2969     0        N            Player        <NA>
    ## 2970     0        N            Player        <NA>
    ## 2971     0        N            Player        <NA>
    ## 2972     0        N            Player        <NA>
    ## 2973     0        N            Player        <NA>
    ## 2974     0        N            Player        <NA>
    ## 2975    NA        Y           Manager        <NA>
    ## 2976    NA        Y            Player        <NA>
    ## 2885   406        Y            Player        <NA>
    ## 2886   370        Y            Player        <NA>
    ## 2887   305        N            Player        <NA>
    ## 2888   246        N            Player        <NA>
    ## 2889   236        N            Player        <NA>
    ## 2890   233        N            Player        <NA>
    ## 2891   205        N            Player        <NA>
    ## 2892   174        N            Player        <NA>
    ## 2893   138        N            Player        <NA>
    ## 2894   135        N            Player        <NA>
    ## 2895   127        N            Player        <NA>
    ## 2896   126        N            Player        <NA>
    ## 2897    97        N            Player        <NA>
    ## 2898    91        N            Player        <NA>
    ## 2899    69        N            Player        <NA>
    ## 2900    63        N            Player        <NA>
    ## 2901    62        N            Player        <NA>
    ## 2902    43        N            Player        <NA>
    ## 2903    42        N            Player        <NA>
    ## 2904    40        N            Player        <NA>
    ## 2905    32        N            Player        <NA>
    ## 2906    28        N            Player        <NA>
    ## 2907    26        N            Player        <NA>
    ## 2908    22        N            Player        <NA>
    ## 2909     6        N            Player        <NA>
    ## 2910     5        N            Player        <NA>
    ## 2911     5        N            Player        <NA>
    ## 2912     3        N            Player        <NA>
    ## 2913     3        N            Player        <NA>
    ## 2914     2        N            Player        <NA>
    ## 2915     2        N            Player        <NA>
    ## 2916     1        N            Player        <NA>
    ## 2917     1        N            Player        <NA>
    ## 2918     0        N            Player        <NA>
    ## 2919     0        N            Player        <NA>
    ## 2920     0        N            Player        <NA>
    ## 2921     0        N            Player        <NA>
    ## 2922     0        N            Player        <NA>
    ## 2923     0        N            Player        <NA>
    ## 2924     0        N            Player        <NA>
    ## 2925     0        N            Player        <NA>
    ## 2926     0        N            Player        <NA>
    ## 2927    NA        Y Pioneer/Executive        <NA>
    ## 2928    NA        Y            Player        <NA>
    ## 2844   337        Y            Player        <NA>
    ## 2845   243        N            Player        <NA>
    ## 2846   241        N            Player        <NA>
    ## 2847   239        N            Player        <NA>
    ## 2848   238        N            Player        <NA>
    ## 2849   233        N            Player        <NA>
    ## 2850   168        N            Player        <NA>
    ## 2851   166        N            Player        <NA>
    ## 2852   164        N            Player        <NA>
    ## 2853   163        N            Player        <NA>
    ## 2854   142        N            Player        <NA>
    ## 2855    94        N            Player        <NA>
    ## 2856    93        N            Player        <NA>
    ## 2857    83        N            Player        <NA>
    ## 2858    77        N            Player        <NA>
    ## 2859    62        N            Player        <NA>
    ## 2860    56        N            Player        <NA>
    ## 2861   148        N            Player        <NA>
    ## 2862    48        N            Player        <NA>
    ## 2863    38        N            Player        <NA>
    ## 2864    33        N            Player        <NA>
    ## 2865    23        N            Player        <NA>
    ## 2866    18        N            Player        <NA>
    ## 2867     6        N            Player        <NA>
    ## 2868     5        N            Player        <NA>
    ## 2869     2        N            Player        <NA>
    ## 2870     1        N            Player        <NA>
    ## 2871     1        N            Player        <NA>
    ## 2872     1        N            Player        <NA>
    ## 2873     1        N            Player        <NA>
    ## 2874     1        N            Player        <NA>
    ## 2875     1        N            Player        <NA>
    ## 2876     0        N            Player        <NA>
    ## 2877     0        N            Player        <NA>
    ## 2878     0        N            Player        <NA>
    ## 2879     0        N            Player        <NA>
    ## 2880     0        N            Player        <NA>
    ## 2881     0        N            Player        <NA>
    ## 2882     0        N            Player        <NA>
    ## 2883    NA        Y           Manager        <NA>
    ## 2884    NA        Y            Player        <NA>
    ## 2781   340        Y            Player        <NA>
    ## 2782   333        Y            Player        <NA>
    ## 2783   238        N            Player        <NA>
    ## 2784   230        N            Player        <NA>
    ## 2785   209        N            Player        <NA>
    ## 2786   177        N            Player        <NA>
    ## 2787   164        N            Player        <NA>
    ## 2788   161        N            Player        <NA>
    ## 2789   146        N            Player        <NA>
    ## 2790   134        N            Player        <NA>
    ## 2791   124        N            Player        <NA>
    ## 2792   111        N            Player        <NA>
    ## 2793    96        N            Player        <NA>
    ## 2794    83        N            Player        <NA>
    ## 2795    66        N            Player        <NA>
    ## 2796    59        N            Player        <NA>
    ## 2797    50        N            Player        <NA>
    ## 2798    48        N            Player        <NA>
    ## 2799    43        N            Player        <NA>
    ## 2800    33        N            Player        <NA>
    ## 2801    31        N            Player        <NA>
    ## 2802    29        N            Player        <NA>
    ## 2803    21        N            Player        <NA>
    ## 2804    15        N            Player        <NA>
    ## 2805     6        N            Player        <NA>
    ## 2806     5        N            Player        <NA>
    ## 2807     3        N            Player        <NA>
    ## 2808     3        N            Player        <NA>
    ## 2809     2        N            Player        <NA>
    ## 2810     1        N            Player        <NA>
    ## 2811     1        N            Player        <NA>
    ## 2812     1        N            Player        <NA>
    ## 2813     0        N            Player        <NA>
    ## 2814     0        N            Player        <NA>
    ## 2815     0        N            Player        <NA>
    ## 2816     0        N            Player        <NA>
    ## 2817     0        N            Player        <NA>
    ## 2818     0        N            Player        <NA>
    ## 2819     0        N            Player        <NA>
    ## 2820     0        N            Player        <NA>
    ## 2821     0        N            Player        <NA>
    ## 2822     0        N            Player        <NA>
    ## 2823     0        N            Player        <NA>
    ## 2824     0        N            Player        <NA>
    ## 2825     0        N            Player        <NA>
    ## 2826     0        N            Player        <NA>
    ## 2827     0        N            Player        <NA>
    ## 2828     0        N            Player        <NA>
    ## 2829     0        N            Player        <NA>
    ## 2830     0        N            Player        <NA>
    ## 2831     0        N            Player        <NA>
    ## 2832     0        N            Player        <NA>
    ## 2833     0        N            Player        <NA>
    ## 2834     0        N            Player        <NA>
    ## 2835     0        N            Player        <NA>
    ## 2836     0        N            Player        <NA>
    ## 2837     0        N            Player        <NA>
    ## 2838     0        N            Player        <NA>
    ## 2839     0        N            Player        <NA>
    ## 2840     0        N            Player        <NA>
    ## 2841     0        N            Player        <NA>
    ## 2842    NA        Y            Player        <NA>
    ## 2843    NA        Y Pioneer/Executive        <NA>
    ## 2725   409        Y            Player        <NA>
    ## 2726   308        N            Player        <NA>
    ## 2727   297        N            Player        <NA>
    ## 2728   242        N            Player        <NA>
    ## 2729   233        N            Player        <NA>
    ## 2730   174        N            Player        <NA>
    ## 2731   168        N            Player        <NA>
    ## 2732   166        N            Player        <NA>
    ## 2733   159        N            Player        <NA>
    ## 2734   147        N            Player        <NA>
    ## 2735   130        N            Player        <NA>
    ## 2736   127        N            Player        <NA>
    ## 2737   120        N            Player        <NA>
    ## 2738    88        N            Player        <NA>
    ## 2739    80        N            Player        <NA>
    ## 2740    63        N            Player        <NA>
    ## 2741    58        N            Player        <NA>
    ## 2742    53        N            Player        <NA>
    ## 2743    53        N            Player        <NA>
    ## 2744    52        N            Player        <NA>
    ## 2745    36        N            Player        <NA>
    ## 2746    35        N            Player        <NA>
    ## 2747    30        N            Player        <NA>
    ## 2748    20        N            Player        <NA>
    ## 2749    14        N            Player        <NA>
    ## 2750    11        N            Player        <NA>
    ## 2751     9        N            Player        <NA>
    ## 2752     9        N            Player        <NA>
    ## 2753     8        N            Player        <NA>
    ## 2754     6        N            Player        <NA>
    ## 2755     6        N            Player        <NA>
    ## 2756     5        N            Player        <NA>
    ## 2757     3        N            Player        <NA>
    ## 2758     3        N            Player        <NA>
    ## 2759     2        N            Player        <NA>
    ## 2760     1        N            Player        <NA>
    ## 2761     1        N            Player        <NA>
    ## 2762     1        N            Player        <NA>
    ## 2763     0        N            Player        <NA>
    ## 2764     0        N            Player        <NA>
    ## 2765     0        N            Player        <NA>
    ## 2766     0        N            Player        <NA>
    ## 2767     0        N            Player        <NA>
    ## 2768     0        N            Player        <NA>
    ## 2769     0        N            Player        <NA>
    ## 2770     0        N            Player        <NA>
    ## 2771     0        N            Player        <NA>
    ## 2772     0        N            Player        <NA>
    ## 2773     0        N            Player        <NA>
    ## 2774     0        N            Player        <NA>
    ## 2775     0        N            Player        <NA>
    ## 2776     0        N            Player        <NA>
    ## 2777     0        N            Player        <NA>
    ## 2778     0        N            Player        <NA>
    ## 2779    NA        Y Pioneer/Executive        <NA>
    ## 2780    NA        Y            Player        <NA>
    ## 2686   301        Y            Player        <NA>
    ## 2687   261        N            Player        <NA>
    ## 2688   254        N            Player        <NA>
    ## 2689   226        N            Player        <NA>
    ## 2690   219        N            Player        <NA>
    ## 2691   181        N            Player        <NA>
    ## 2692   169        N            Player        <NA>
    ## 2693   158        N            Player        <NA>
    ## 2694   158        N            Player        <NA>
    ## 2695   149        N            Player        <NA>
    ## 2696   130        N            Player        <NA>
    ## 2697   115        N            Player        <NA>
    ## 2698    83        N            Player        <NA>
    ## 2699    76        N            Player        <NA>
    ## 2700    66        N            Player        <NA>
    ## 2701    60        N            Player        <NA>
    ## 2702    58        N            Player        <NA>
    ## 2703    51        N            Player        <NA>
    ## 2704    48        N            Player        <NA>
    ## 2705    41        N            Player        <NA>
    ## 2706    32        N            Player        <NA>
    ## 2707    27        N            Player        <NA>
    ## 2708    23        N            Player        <NA>
    ## 2709    18        N            Player        <NA>
    ## 2710     8        N            Player        <NA>
    ## 2711     7        N            Player        <NA>
    ## 2712     6        N            Player        <NA>
    ## 2713     6        N            Player        <NA>
    ## 2714     5        N            Player        <NA>
    ## 2715     4        N            Player        <NA>
    ## 2716     3        N            Player        <NA>
    ## 2717     2        N            Player        <NA>
    ## 2718     1        N            Player        <NA>
    ## 2719     1        N            Player        <NA>
    ## 2720     1        N            Player        <NA>
    ## 2721     0        N            Player        <NA>
    ## 2722     0        N            Player        <NA>
    ## 2723    NA        Y            Player        <NA>
    ## 2724    NA        Y Pioneer/Executive        <NA>
    ## 2647   321        Y            Player        <NA>
    ## 2648   239        N            Player        <NA>
    ## 2649   224        N            Player        <NA>
    ## 2650   222        N            Player        <NA>
    ## 2651   212        N            Player        <NA>
    ## 2652   197        N            Player        <NA>
    ## 2653   163        N            Player        <NA>
    ## 2654   152        N            Player        <NA>
    ## 2655   146        N            Player        <NA>
    ## 2656   141        N            Player        <NA>
    ## 2657   139        N            Player        <NA>
    ## 2658   105        N            Player        <NA>
    ## 2659    85        N            Player        <NA>
    ## 2660    82        N            Player        <NA>
    ## 2661    66        N            Player        <NA>
    ## 2662    57        N            Player        <NA>
    ## 2663    55        N            Player        <NA>
    ## 2664    52        N            Player        <NA>
    ## 2665    45        N            Player        <NA>
    ## 2666    43        N            Player        <NA>
    ## 2667    43        N            Player        <NA>
    ## 2668    39        N            Player        <NA>
    ## 2669    33        N            Player        <NA>
    ## 2670    16        N            Player        <NA>
    ## 2671    14        N            Player        <NA>
    ## 2672    10        N            Player        <NA>
    ## 2673     8        N            Player        <NA>
    ## 2674     7        N            Player        <NA>
    ## 2675     5        N            Player        <NA>
    ## 2676     4        N            Player        <NA>
    ## 2677     4        N            Player        <NA>
    ## 2678     4        N            Player        <NA>
    ## 2679     3        N            Player        <NA>
    ## 2680     3        N            Player        <NA>
    ## 2681    NA        Y            Player        <NA>
    ## 2682    NA        Y            Player        <NA>
    ## 2683    NA        Y           Manager        <NA>
    ## 2684    NA        Y            Player        <NA>
    ## 2685    NA        Y            Player        <NA>
    ## 2611   337        Y            Player        <NA>
    ## 2612   305        Y            Player        <NA>
    ## 2613   233        N            Player        <NA>
    ## 2614   197        N            Player        <NA>
    ## 2615   189        N            Player        <NA>
    ## 2616   186        N            Player        <NA>
    ## 2617   174        N            Player        <NA>
    ## 2618   159        N            Player        <NA>
    ## 2619   149        N            Player        <NA>
    ## 2620   129        N            Player        <NA>
    ## 2621   129        N            Player        <NA>
    ## 2622   114        N            Player        <NA>
    ## 2623    87        N            Player        <NA>
    ## 2624    85        N            Player        <NA>
    ## 2625    62        N            Player        <NA>
    ## 2626    56        N            Player        <NA>
    ## 2627    55        N            Player        <NA>
    ## 2628    52        N            Player        <NA>
    ## 2629    50        N            Player        <NA>
    ## 2630    47        N            Player        <NA>
    ## 2631    23        N            Player        <NA>
    ## 2632    21        N            Player        <NA>
    ## 2633    21        N            Player        <NA>
    ## 2634    15        N            Player        <NA>
    ## 2635    15        N            Player        <NA>
    ## 2636     9        N            Player        <NA>
    ## 2637     9        N            Player        <NA>
    ## 2638     8        N            Player        <NA>
    ## 2639     7        N            Player        <NA>
    ## 2640     7        N            Player        <NA>
    ## 2641     5        N            Player        <NA>
    ## 2642     2        N            Player        <NA>
    ## 2643    NA        Y            Player        <NA>
    ## 2644    NA        Y            Player        <NA>
    ## 2645    NA        Y            Umpire        <NA>
    ## 2646    NA        Y            Player        <NA>
    ## 2570   273        Y            Player        <NA>
    ## 2571   263        N            Player        <NA>
    ## 2572   233        N            Player        <NA>
    ## 2573   188        N            Player        <NA>
    ## 2574   177        N            Player        <NA>
    ## 2575   155        N            Player        <NA>
    ## 2576   154        N            Player        <NA>
    ## 2577   148        N            Player        <NA>
    ## 2578   129        N            Player        <NA>
    ## 2579   129        N            Player        <NA>
    ## 2580   123        N            Player        <NA>
    ## 2581   117        N            Player        <NA>
    ## 2582   114        N            Player        <NA>
    ## 2583    94        N            Player        <NA>
    ## 2584    76        N            Player        <NA>
    ## 2585    76        N            Player        <NA>
    ## 2586    76        N            Player        <NA>
    ## 2587    70        N            Player        <NA>
    ## 2588    48        N            Player        <NA>
    ## 2589    37        N            Player        <NA>
    ## 2590    33        N            Player        <NA>
    ## 2591    23        N            Player        <NA>
    ## 2592    23        N            Player        <NA>
    ## 2593    22        N            Player        <NA>
    ## 2594    13        N            Player        <NA>
    ## 2595    11        N            Player        <NA>
    ## 2596    11        N            Player        <NA>
    ## 2597    10        N            Player        <NA>
    ## 2598     9        N            Player        <NA>
    ## 2599     8        N            Player        <NA>
    ## 2600     7        N            Player        <NA>
    ## 2601     6        N            Player        <NA>
    ## 2602     5        N            Player        <NA>
    ## 2603     4        N            Player        <NA>
    ## 2604     3        N            Player        <NA>
    ## 2605     1        N            Player        <NA>
    ## 2606     1        N            Player        <NA>
    ## 2607    NA        Y            Player        <NA>
    ## 2608    NA        Y           Manager        <NA>
    ## 2609    NA        Y            Player        <NA>
    ## 2610    NA        Y            Player        <NA>
    ## 2522   322        Y            Player        <NA>
    ## 2523   284        Y            Player        <NA>
    ## 2524   224        N            Player        <NA>
    ## 2525   215        N            Player        <NA>
    ## 2526   198        N            Player        <NA>
    ## 2527   190        N            Player        <NA>
    ## 2528   145        N            Player        <NA>
    ## 2529   141        N            Player        <NA>
    ## 2530   118        N            Player        <NA>
    ## 2531   111        N            Player        <NA>
    ## 2532   111        N            Player        <NA>
    ## 2533   110        N            Player        <NA>
    ## 2534   101        N            Player        <NA>
    ## 2535    94        N            Player        <NA>
    ## 2536    79        N            Player        <NA>
    ## 2537    78        N            Player        <NA>
    ## 2538    73        N            Player        <NA>
    ## 2539    61        N            Player        <NA>
    ## 2540    56        N            Player        <NA>
    ## 2541    54        N            Player        <NA>
    ## 2542    51        N            Player        <NA>
    ## 2543    29        N            Player        <NA>
    ## 2544    28        N            Player        <NA>
    ## 2545    27        N            Player        <NA>
    ## 2546    19        N            Player        <NA>
    ## 2547    11        N            Player        <NA>
    ## 2548     9        N            Player        <NA>
    ## 2549     8        N            Player        <NA>
    ## 2550     7        N            Player        <NA>
    ## 2551     7        N            Player        <NA>
    ## 2552     6        N            Player        <NA>
    ## 2553     5        N            Player        <NA>
    ## 2554     5        N            Player        <NA>
    ## 2555     4        N            Player        <NA>
    ## 2556     4        N            Player        <NA>
    ## 2557     4        N            Player        <NA>
    ## 2558     3        N            Player        <NA>
    ## 2559     3        N            Player        <NA>
    ## 2560     3        N            Player        <NA>
    ## 2561     3        N            Player        <NA>
    ## 2562     3        N            Player        <NA>
    ## 2563     2        N            Player        <NA>
    ## 2564     2        N            Player        <NA>
    ## 2565     2        N            Player        <NA>
    ## 2566    NA        Y            Player        <NA>
    ## 2567    NA        Y            Player        <NA>
    ## 2568    NA        Y            Umpire        <NA>
    ## 2569    NA        Y            Player        <NA>
    ## 2473   316        Y            Player        <NA>
    ## 2474   255        N            Player        <NA>
    ## 2475   235        N            Player        <NA>
    ## 2476   218        N            Player        <NA>
    ## 2477   213        N            Player        <NA>
    ## 2478   177        N            Player        <NA>
    ## 2479   157        N            Player        <NA>
    ## 2480   145        N            Player        <NA>
    ## 2481   127        N            Player        <NA>
    ## 2482   126        N            Player        <NA>
    ## 2483   114        N            Player        <NA>
    ## 2484   111        N            Player        <NA>
    ## 2485   101        N            Player        <NA>
    ## 2486    96        N            Player        <NA>
    ## 2487    93        N            Player        <NA>
    ## 2488    79        N            Player        <NA>
    ## 2489    73        N            Player        <NA>
    ## 2490    73        N            Player        <NA>
    ## 2491    53        N            Player        <NA>
    ## 2492    47        N            Player        <NA>
    ## 2493    43        N            Player        <NA>
    ## 2494    33        N            Player        <NA>
    ## 2495    25        N            Player        <NA>
    ## 2496    23        N            Player        <NA>
    ## 2497    14        N            Player        <NA>
    ## 2498    12        N            Player        <NA>
    ## 2499    11        N            Player        <NA>
    ## 2500     9        N            Player        <NA>
    ## 2501     8        N            Player        <NA>
    ## 2502     7        N            Player        <NA>
    ## 2503     7        N            Player        <NA>
    ## 2504     6        N            Player        <NA>
    ## 2505     5        N            Player        <NA>
    ## 2506     5        N            Player        <NA>
    ## 2507     5        N            Player        <NA>
    ## 2508     4        N            Player        <NA>
    ## 2509     4        N            Player        <NA>
    ## 2510     3        N            Player        <NA>
    ## 2511     3        N            Player        <NA>
    ## 2512     2        N            Player        <NA>
    ## 2513     2        N            Player        <NA>
    ## 2514     2        N            Player        <NA>
    ## 2515     1        N            Player        <NA>
    ## 2516     1        N            Player        <NA>
    ## 2517    NA        Y            Player        <NA>
    ## 2518    NA        Y            Umpire        <NA>
    ## 2519    NA        Y            Player        <NA>
    ## 2520    NA        Y            Player        <NA>
    ## 2521    NA        Y            Player        <NA>
    ## 2422   344        Y            Player        <NA>
    ## 2423   339        Y            Player        <NA>
    ## 2424   301        Y            Player        <NA>
    ## 2425   235        N            Player        <NA>
    ## 2426   161        N            Player        <NA>
    ## 2427   157        N            Player        <NA>
    ## 2428   149        N            Player        <NA>
    ## 2429   129        N            Player        <NA>
    ## 2430   120        N            Player        <NA>
    ## 2431   117        N            Player        <NA>
    ## 2432   115        N            Player        <NA>
    ## 2433   105        N            Player        <NA>
    ## 2434   104        N            Player        <NA>
    ## 2435   103        N            Player        <NA>
    ## 2436    92        N            Player        <NA>
    ## 2437    84        N            Player        <NA>
    ## 2438    64        N            Player        <NA>
    ## 2439    61        N            Player        <NA>
    ## 2440    55        N            Player        <NA>
    ## 2441    36        N            Player        <NA>
    ## 2442    31        N            Player        <NA>
    ## 2443    24        N            Player        <NA>
    ## 2444    21        N            Player        <NA>
    ## 2445    12        N            Player        <NA>
    ## 2446    11        N            Player        <NA>
    ## 2447    10        N            Player        <NA>
    ## 2448    10        N            Player        <NA>
    ## 2449     9        N            Player        <NA>
    ## 2450     9        N            Player        <NA>
    ## 2451     9        N            Player        <NA>
    ## 2452     8        N            Player        <NA>
    ## 2453     8        N            Player        <NA>
    ## 2454     7        N            Player        <NA>
    ## 2455     5        N            Player        <NA>
    ## 2456     5        N            Player        <NA>
    ## 2457     4        N            Player        <NA>
    ## 2458     4        N            Player        <NA>
    ## 2459     4        N            Player        <NA>
    ## 2460     4        N            Player        <NA>
    ## 2461     4        N            Player        <NA>
    ## 2462     3        N            Player        <NA>
    ## 2463     3        N            Player        <NA>
    ## 2464     2        N            Player        <NA>
    ## 2465     2        N            Player        <NA>
    ## 2466     2        N            Player        <NA>
    ## 2467     1        N            Player        <NA>
    ## 2468    NA        Y            Player        <NA>
    ## 2469    NA        Y            Player        <NA>
    ## 2470    NA        Y Pioneer/Executive        <NA>
    ## 2471    NA        Y            Player        <NA>
    ## 2472    NA        Y            Player        <NA>
    ## 2366   242        N            Player        <NA>
    ## 2367   240        N            Player        <NA>
    ## 2368   212        N            Player        <NA>
    ## 2369   180        N            Player        <NA>
    ## 2370   165        N            Player        <NA>
    ## 2371   157        N            Player        <NA>
    ## 2372   127        N            Player        <NA>
    ## 2373   123        N            Player        <NA>
    ## 2374   123        N            Player        <NA>
    ## 2375   110        N            Player        <NA>
    ## 2376   105        N            Player        <NA>
    ## 2377    98        N            Player        <NA>
    ## 2378    94        N            Player        <NA>
    ## 2379    92        N            Player        <NA>
    ## 2380    90        N            Player        <NA>
    ## 2381    89        N            Player        <NA>
    ## 2382    83        N            Player        <NA>
    ## 2383    78        N            Player        <NA>
    ## 2384    54        N            Player        <NA>
    ## 2385    39        N            Player        <NA>
    ## 2386    17        N            Player        <NA>
    ## 2387    15        N            Player        <NA>
    ## 2388    14        N            Player        <NA>
    ## 2389    12        N            Player        <NA>
    ## 2390    11        N            Player        <NA>
    ## 2391    10        N            Player        <NA>
    ## 2392    10        N            Player        <NA>
    ## 2393     9        N            Player        <NA>
    ## 2394     8        N            Player        <NA>
    ## 2395     7        N            Player        <NA>
    ## 2396     7        N            Player        <NA>
    ## 2397     7        N            Player        <NA>
    ## 2398     7        N            Player        <NA>
    ## 2399     5        N            Player        <NA>
    ## 2400     5        N            Player        <NA>
    ## 2401     4        N            Player        <NA>
    ## 2402     4        N            Player        <NA>
    ## 2403     4        N            Player        <NA>
    ## 2404     4        N            Player        <NA>
    ## 2405     3        N            Player        <NA>
    ## 2406     3        N            Player        <NA>
    ## 2407     3        N            Player        <NA>
    ## 2408     2        N            Player        <NA>
    ## 2409     2        N            Player        <NA>
    ## 2410     2        N            Player        <NA>
    ## 2411     2        N            Player        <NA>
    ## 2412     2        N            Player        <NA>
    ## 2413     1        N            Player        <NA>
    ## 2414    NA        Y            Player        <NA>
    ## 2415    NA        Y            Player        <NA>
    ## 2416    NA        Y            Player        <NA>
    ## 2417    NA        Y            Player        <NA>
    ## 2418    NA        Y            Player        <NA>
    ## 2419    NA        Y            Player        <NA>
    ## 2420    NA        Y            Player        <NA>
    ## 2421    NA        Y Pioneer/Executive        <NA>
    ## 2317   232        Y            Player        <NA>
    ## 2318   167        N            Player        <NA>
    ## 2319   145        N            Player        <NA>
    ## 2320   140        N            Player        <NA>
    ## 2321   133        N            Player        <NA>
    ## 2322   126        N            Player        <NA>
    ## 2323   120        N            Player        <NA>
    ## 2324    97        N            Player        <NA>
    ## 2325    97        N            Player        <NA>
    ## 2326    90        N            Player        <NA>
    ## 2327    89        N            Player        <NA>
    ## 2328    88        N            Player        <NA>
    ## 2329    80        N            Player        <NA>
    ## 2330    79        N            Player        <NA>
    ## 2331    79        N            Player        <NA>
    ## 2332    75        N            Player        <NA>
    ## 2333    75        N            Player        <NA>
    ## 2334    62        N            Player        <NA>
    ## 2335    55        N            Player        <NA>
    ## 2336    51        N            Player        <NA>
    ## 2337    51        N            Player        <NA>
    ## 2338    29        N            Player        <NA>
    ## 2339    15        N            Player        <NA>
    ## 2340    14        N            Player        <NA>
    ## 2341    12        N            Player        <NA>
    ## 2342    11        N            Player        <NA>
    ## 2343    10        N            Player        <NA>
    ## 2344     9        N            Player        <NA>
    ## 2345     9        N            Player        <NA>
    ## 2346     8        N            Player        <NA>
    ## 2347     7        N            Player        <NA>
    ## 2348     7        N            Player        <NA>
    ## 2349     5        N            Player        <NA>
    ## 2350     5        N            Player        <NA>
    ##  [ reached 'max' / getOption("max.print") -- omitted 2307 rows ]

``` r
# creating csv
write.csv(new_hof, file="HallOfFame.csv", row.names = FALSE)
```
