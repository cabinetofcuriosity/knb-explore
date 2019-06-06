Count and size vs PISCO
================

Size and count data is seperated by season so I want to do the same thing to PISCO data, putting a season for each row with the function `dataToNum` defined before.

``` r
# first add a column to PISCO data
allLocationNDateS <- mutate(allLocationNDate, season = c(0), year = c(0))
for (i in 1:nrow(allLocationNDateS)) {
  curDate <- (dateToNum(allLocationNDateS$end[i]) + dateToNum(allLocationNDateS$begin[i])) / 2
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
  allLocationNDateS$season[i] <- rt
  allLocationNDateS$year[i] <- curYear
}
```

``` r
write.csv(allLocationNDateS, '../data/PISCOwSeason.csv')
```

``` r
seastarkat <- read.csv("../data/seastarkat_size_count_totals_download.csv", stringsAsFactors = FALSE)
```

We first examine the species data inside California and later to all the data.

``` r
seastarkat_ca <- seastarkat[seastarkat$state_province == 'California', ]
seastarkat_ca_g <- seastarkat_ca %>% 
  group_by(season_sequence, marine_common_year, target_assemblage, latitude, longitude) %>%
  summarise(total_sum = sum(total))
```

For each size and count data entry, I find a nearest PISCO location(latitude and longitude), using the Euclidean distance, in the same season and year. If the minimum Euclidean distance is bigger than 1 or the seasons and the years do not match, this size and count data entry doesn't have a corresponding PISCO dataset.

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

Call the function defined above on size and count data to obtain the indicies of all corresponding PISCO datasets.

``` r
ca_which_pis <- c()
for (m in 1:nrow(seastarkat_ca_g)) {
  ca_which_pis <- c(ca_which_pis, nearPisWSeason(m, seastarkat_ca_g, allLocationNDate))
}
```

Add a column of these indicies to size and count data frame and write the new data frame into a csv file.

``` r
seastarkat_ca_g$pis_ind <- ca_which_pis
write.csv(seastarkat_ca_g, '../data/scca_seastarka.csv')
```

We can see that more than half of size and count rows are discarded because of wrong season and year or wrong location and this can be shown with two plots below.

``` r
nrow(filter(seastarkat_ca_g, pis_ind == -1))
```

    ## [1] 560

``` r
#, season_sequence, marine_common_year
ggplot() +
  geom_point(data = filter(seastarkat_ca_g, pis_ind == -1), aes(x = latitude, y = longitude, color = "Discarded size and count")) +
  geom_point(data = allLocationNDate, aes(x = latitude, y = longitude, color = "PISCO")) + 
  theme_minimal() +
  labs(title = "Locations of discarded size and count data entries", color = "Datasets\n") +
  scale_colour_manual("", 
                      breaks = c("Discarded size and count", "PISCO"),
                      values = c("light green", "light blue")) +
  ylim(-125, -115)
```

    ## Warning: Removed 7 rows containing missing values (geom_point).

![](knb-seak_files/figure-markdown_github/unnamed-chunk-9-1.png)

``` r
ggplot() +
  geom_point(data = filter(seastarkat_ca_g, pis_ind == -1), aes(x = season_sequence, y = marine_common_year, color = "Discarded size and count"), alpha = 0.1) +
  geom_point(data = allLocationNDate, aes(x = season, y = year, color = "PISCO"), alpha = 0.1) + 
  theme_minimal() +
  labs(title = "Season and year of discarded size and count data entries", color = "Datasets\n") +
  scale_colour_manual("", 
                      breaks = c("Discarded size and count", "PISCO"),
                      values = c("light green", "light blue")) +
  ylim(1990, 2020)
```

    ## Warning: Removed 12 rows containing missing values (geom_point).

![](knb-seak_files/figure-markdown_github/unnamed-chunk-10-1.png)


## Adding PISCO datasets with the same locations  
When I find a nearest PISCO location, I only considered one dataset with the closest location(latitude and longitude), restrained by the season and year. However, for each season, year and location, there should be multiple datasets. Thus I should find these datasets and store them in a matrix.  
Also, since I've found that some data entries don't have a corresponding PISCO dataset, I can discard them. So only 421 rows of size and count data are useful for now.
```{r, eval=FALSE}
sk_ca_filtered <- filter(seastarkat_ca_g, pis_ind != -1)
nrow(sk_ca_filtered)
```
Define two matricies and put all related PISCO datasets for each row into the matricies, one for IDs, the other for the row indicies.
```{r, eval=FALSE}
needed_pisID <- matrix(list(), nrow=421, ncol=1) # each row for each size and count data row
needed_pisIND <- matrix(list(), nrow=421, ncol=1)
for (i in 1:nrow(sk_ca_filtered)) {
  cur_pis <- sk_ca_filtered$pis_ind[i] # index of the ith pisco dataset in allLocationNDateS
  cur_pis_dt <- allLocationNDateS[cur_pis, ]
  these_pis <- filter(allLocationNDateS, 
                      latitude == cur_pis_dt$latitude, 
                      longitude == cur_pis_dt$longitude, 
                      season == cur_pis_dt$season, 
                      year == cur_pis_dt$year)
  needed_pisID[[i, 1]] <- these_pis$ID
  needed_pisIND[[i, 1]] <- these_pis$X
}
```

What is the maximum number of corresponding PISCO datasets for each row in `needed_pis`? 
```{r}
max(sapply(needed_pis, function(row) {length(row)}))
```


Download PISCO datasets
-----------------------

``` r
library(dataone)
library(XML)
cn <- CNode("PROD")

downl_pis <- function(i) {
  id <- dataset_w_pop[i, 3]

  # download the metadata file to find the data table
  metadata <- rawToChar(getObject(mn, id))
  doc = xmlRoot(xmlTreeParse(metadata, asText=TRUE, trim = TRUE, ignoreBlanks = TRUE))

  # now extract the node that has the data table's information
  node <- getNodeSet(doc, "//entityName")
  table_id <- xmlValue(node[[1]])
  
  if (grepl("\\.(TXT|txt)", table_id)) {
    table_id <- gsub("\\.(TXT|txt)", "", table_id)
  }
  
  # we can see that the ids have the pattern
  dataRaw <- getObject(mn, paste0("doi:10.6085/AA/", table_id))
  dataChar <- rawToChar(dataRaw)
  theData <- textConnection(dataChar)
  df <- read.csv(theData, stringsAsFactors=FALSE, header = TRUE, sep = " ", row.names=NULL)
  return(df)
}
```
Because downloading PISCO datasets is pretty slow, I want to reduce unneccessary downloading as mnuch as possible.  
Ealier I have downloaded plenty of PISCO datasets stored in a hard drive, from the 1st to the 1260th except for some of them, so the indicies of already dowloaded PISCO datasets are:
```{r, eval=FALSE}
dlded_pis <- (1:1260)[!(1:1260) %in% c(7, 8, 9, 15, 23, 28, 64, 77, 93, 105, 106, 110, 113, 121, 132, 133, 135, 186, 201, 223, 257, 301, 340, 345, 359, 360, 414, 418, 430, 503, 519, 537, 552, 562, 573, 580, 617, 653, 655, 673, 676, 688, 692, 694, 697, 701, 710, 727, 778, 786, 798, 814, 817, 836, 851, 863, 870, 878, 879, 885, 900, 912, 915, 927, 953, 991, 1003, 1015, 1082)]
```
Here I define a function to obtain temperature information of one PISCO dataset such that I only download one PISCO dataset once, storing already downloaded datasets.
```{r, eval=FALSE}
# returns temperature info of cur_pis
down_pis_temp <- function(cur_pis) {
  ### if you've downloaded this dataset
  if (cur_pis %in% dlded_pis) { 
    pisco_df <- read.csv(file = paste0("../data/downloaded/d", cur_pis, ".csv"), stringsAsFactors = FALSE)
  } else {
    cur_pisID <- needed_pisID[[i, 1]][j]
    pisco_df <- downl_pis(cur_pisID)
    write.csv(pisco_df, file = paste0("../data/downloaded/d", cur_pis, ".csv"))
    dlded_pis <- c(dlded_pis, cur_pis)
  }
  # discard if temp == 9999.00, which is equivalent to NA for PISCO datasets
  return(filter(pisco_df, temp_c != 9999.00)$temp_c)
}
```
Initialize the mean_temp column with a huge unrealistic value so that it works as `NA`. For each row of size and count data frame, use `try` to obtain the temperature information from the related PISCO datasets and print error information.  
```{r}
seastarkat_ca_g$mean_temp <- c(9999.0)
  
for (i in 1:nrow(needed_pisID)) {
  all_temp <- c()
  # indicies of needed pisco datasets in allLocationNDateS
  these_pis <- needed_pisIND[[i, 1]]
  for (j in 1:length(these_pis)) {
    tryCatch(all_temp <- c(all_temp, down_pis_temp(these_pis[j])), 
             error = function(e) {print(paste("row", i, "PISCO", j, "invalid"));
                                  NaN})
}
    
  seastarkat_ca_g$mean_temp[i] <- mean(all_temp)
}
```

``` r
dt1_tem <- filter(dt1, date == '2002-11-03', temp_c != 9999.00)

ggplot(data = dt1_tem, aes(x=X, y=temp_c)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```
