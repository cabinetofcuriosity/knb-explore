Location
================

Size and count data is seperated by season so I want to do the same thing to PISCO data.

``` r
# first add a column to PISCO data
allLocationNDate <- mutate(allLocationNDate, season = c(0), year = c(0))
for (i in 1:nrow(allLocationNDate)) {
  curDate <- (dateToNum(allLocationNDate$end[i]) + dateToNum(allLocationNDate$begin[i])) / 2
  curSeason <- curDate %% 1
  curYear <- curDate - curSeason
  # curDate <- dateToNum('0000-09-06')
  rt <- 0
  if (curSeason >= dateToNum('0000-12-01') & curSeason <= dateToNum('0000-02-29')) {
    rt <- 4 # Winter
  } else if (curSeason >= dateToNum('0000-03-01') & curSeason <= dateToNum('0000-05-31')) {
    rt <- 1 # Spring
  } else if (curSeason >= dateToNum('0000-06-01') & curSeason <= dateToNum('0000-08-31')) {
    rt <- 2 # Summer
  } else {
    rt <- 3 # Fall
  }
  allLocationNDate$season[i] <- rt
  allLocationNDate$year[i] <- curYear
}
```

``` r
unique(allLocationNDate$year)
```

    ##  [1] 2001 2005 2002 2004 2003 2006 2000 2007 2008 2009 1999 2010 1950 2011
    ## [15] 1949   11

``` r
seastarkat <- read.csv("../data/seastarkat_size_count_totals_download.csv", stringsAsFactors = FALSE)
unique(seastarkat$state_province)
```

We first examine the species data inside California and later to all the data.

``` r
seastarkat_ca <- seastarkat[seastarkat$state_province == 'California', ]
seastarkat_ca_g <- seastarkat_ca %>% 
  group_by(season_sequence, marine_common_year, target_assemblage, latitude, longitude) %>%
  summarise(total_sum = sum(total))
seastarkat_ca_g[1,]$season_sequence
```

season\_sequence == 1

``` r
nearPisWSeason <- function(i, sea_star_dt, pisco_dt) {
  ss <- sea_star_dt[i, ]
  ss_loc <- c(ss$longitude, ss$latitude)
  ss_year <- ss$marine_common_year
  ss_season <- ss$season_sequence
  
  min_dis <- 100000
  which_pis <- -1
  for (j in 1:nrow(pisco_dt)) {
    cur_pisco <- pisco_dt[j, ]
    cur_pisco_year <- cur_pisco$year
    cur_pisco_season <- cur_pisco$season
    if (ss_year != cur_pisco_year | ss_season != cur_pisco_season) {
      next
    }
    
    cur_pisco_loc <- c(cur_pisco$longitude, cur_pisco$latitude)
    dis <- (ss_loc[1] - cur_pisco_loc[1])**2 + (ss_loc[2] - cur_pisco_loc[2])**2
    if (dis < min_dis) {
      min_dis <- dis
      which_pis <- j
    }
  }
  if (min_dis > 1) {
    return(-1)
  }
  
  return(which_pis)
}
```

``` r
ca_which_pis <- c()
for (m in 1:nrow(seastarkat_ca_g)) {
  ca_which_pis <- c(ca_which_pis, nearPisWSeason(m, seastarkat_ca_g, allLocationNDate))
}
write.csv(ca_which_pis, '../data/scca_seastar_vs_pis.csv')
```

``` r
seastarkat_ca_g$pis_ind <- ca_which_pis
write.csv(seastarkat_ca_g, '../data/scca_seastarka.csv')
```
