Location
================

This document extracts location and date information from PISCO, species and sea star wasting syndrome datasets and compare them.


### description

readme average description

Sea Stars
---------

So first we get the [sea stars data](https://www.eeb.ucsc.edu/pacificrockyintertidal/explore-the-data/index.html). We can see the data is inside four states.

``` r
sea_stars <- read.csv("../data/phototranraw_download.csv", stringsAsFactors = FALSE)
unique(sea_stars$state_province)
```

We first examine the species data inside California and later to all the data.

``` r
sea_stars_ca <- sea_stars[sea_stars$state_province == 'California', ]
# add ids to represent each row
sea_stars_ca <- mutate(sea_stars_ca, id = 1:nrow(sea_stars_ca))
head(sea_stars_ca, 100)
colnames(sea_stars_ca)
dim(sea_stars_ca)

sea_stars_ca %>%
  group_by(target_assemblage, marine_common_year) %>%
  count()

sea_stars_ca %>%
  group_by(plot_code, marine_site_name) %>%
  count()

### Mapping species onto ggmap
### Somehow looking at temperature and species distribution. Maybe add the temperature on the map as a color?

## Subset by year first.
## plot each species by plot code and site name --- dots to line shape --- then with pisco ---

sea_stars_ca %>%
  filter(marine_site_name == "Stairs", plot_code == 1, target_assemblage == "endocladia", marine_common_year == 2002) %>%
  ggplot(., aes(survey_date, percent_cover, color = target_assemblage)) +
  geom_point() +
  theme_bw()
```

``` r
if(!requireNamespace("devtools")) install.packages("devtools")
devtools::install_github("dkahle/ggmap", ref = "tidyup", force=TRUE)

# Load ggmap
library("ggmap")

# Set your API Key
ggmap::register_google(key = "AIzaSyC9rkZpbIW8EhtAHjKEHrl7AX84ez0hvYs")
```

We can see plenty of repeating latitudes and longitudes so we can extract the unique pairs of latitude and longitude, ie, unique locations.

``` r
#group_by()
ca_unique_ld <- unique(sea_stars_ca[, c('latitude','longitude', 'survey_date')])
ca_unique_ld
```

``` r
unique(sea_stars_ca[, 'target_assemblage'])
```

PISCO
-----

Some setup for getting the location data.

``` r
library(dataone)
library(XML)

id <- dataset_w_pop[1, 3]
cn <- CNode("PROD")

# get the node of this metadata using `dataOne` package
locations <- resolve(cn, id)
mnId <- locations$data[2, "nodeIdentifier"]
mn <- getMNode(cn, mnId)
```

Define a function which returns location and date information of the ith PISCO datasets.

``` r
getLocationNDate <- function(i) {
  # id of the ith row of the PISCO datasets
  id <- dataset_w_pop[i, 3]

  # download the metadata file to find the data table
  metadata <- rawToChar(getObject(mn, id))
  doc = xmlRoot(xmlTreeParse(metadata, asText=TRUE, trim = TRUE, ignoreBlanks = TRUE))

  # now extract the node that has the data table's location information
  node <- getNodeSet(doc, "//boundingCoordinates")
  curLoc <- head(xmlToDataFrame(node, stringsAsFactors = FALSE), 1)
  west <- as.numeric(curLoc[1, 1])
  east <- as.numeric(curLoc[1, 2])
  north <- as.numeric(curLoc[1, 3])
  south <- as.numeric(curLoc[1, 4])
  
  begin_node <- getNodeSet(doc, "//beginDate")
  begin_date <- xmlToDataFrame(begin_node, stringsAsFactors = FALSE)[1, 1]
  end_node <- getNodeSet(doc, "//endDate")
  end_date <- xmlToDataFrame(end_node, stringsAsFactors = FALSE)[1, 1]
  
  # take the average of west and east, and north and south
  thisLoc <- data.frame(longitude = c((1/2)*(west + east)), latitude = c((1/2)*(north + south)), begin = c(begin_date), end = c(end_date), ID = c(id))
  
  return(thisLoc)
}
```

``` r
getLocationNDate(926)
```

> Error in .local(x, ...) : get() error: READ not allowed on <doi:10.6085/AA/JALXXX_015ADCP015R00_20030529.50.2>

After testing, 926th ID doesn't work

``` r
allLocationNDate <- data.frame()
for (i in 1:925) {
  allLocationNDate <- rbind(allLocationNDate, getLocationNDate(i))
}

for (i in 927:nrow(dataset_w_pop)) {
  allLocationNDate <- rbind(allLocationNDate, getLocationNDate(i))
}
```

We save this data frame into a `.csv` file.

``` r
write.csv(allLocationNDate, '../data/pisco-locations-dates.csv')
```

Find the nearest PISCO site for each species data entry
-------------------------------------------------------

Define a function that takes in index `i`, sea stars data `sea_star_dt` and PISCO data `pisco_dt` and returns the row number of the neareast PISCO data

``` r
head(allLocationNDate, 10)
```

    ##     X longitude latitude      begin        end
    ## 1   1 -122.1579 36.97290 2001-03-12 2001-05-14
    ## 2   2 -121.9448 36.55800 2005-01-06 2005-04-05
    ## 3   3 -120.6358 34.86919 2002-04-23 2002-06-12
    ## 4   4 -119.6960 34.03130 2004-10-11 2004-12-20
    ## 5   5 -122.0804 36.94342 2002-01-24 2002-04-16
    ## 6   6 -124.1273 44.25160 2002-05-15 2002-06-26
    ## 7   7 -122.1579 36.97290 2002-01-15 2002-04-11
    ## 8   8 -124.0255 45.00190 2001-04-20 2001-07-09
    ## 9   9 -120.6358 34.86919 2005-02-26 2005-05-24
    ## 10 10 -122.1585 36.97212 2003-04-29 2003-07-27
    ##                                                   ID
    ## 1  doi:10.6085/AA/SHB001_021ADCP020R00_20010312.50.1
    ## 2  doi:10.6085/AA/SWC001_022ADCP021R00_20050106.50.3
    ## 3  doi:10.6085/AA/PTSXXX_015ADCP015R00_20020423.50.4
    ## 4  doi:10.6085/AA/PELXXX_015ADCP015R00_20041011.50.8
    ## 5  doi:10.6085/AA/TPT001_018ADCP018R00_20020124.50.4
    ## 6  doi:10.6085/AA/SH15CX_015ADCP014R00_20020515.50.3
    ## 7  doi:10.6085/AA/SHB001_021ADCP020R00_20020115.50.1
    ## 8  doi:10.6085/AA/CH15CX_015ADCP014R00_20010420.50.3
    ## 9  doi:10.6085/AA/PTSXXX_015ADCP015R00_20050226.50.3
    ## 10 doi:10.6085/AA/SHB001_021ADCP020R00_20030429.50.2

Since there are begin and end dates in PISCO datasets, we want to transform the dates into numbers so that we can compare the dates and determine if a data entry in the sea star dataset can find a corresponding dataset in PISCO ones. Now we define a function to do so, using method found on [stackoverflow](https://stackoverflow.com/a/8215581/10733819)

``` r
dateToNum <- function(date) {
  rt <- 0
  date <- as.POSIXlt(date, format = "%Y-%m-%d")
  # yearday will always be less than or equal to 366 so we can represent yeardays as decimal numbers, and years as whole numbers
  rt <- 1900 + date$year + date$yday / 366
  return(rt)
}
```

``` r
dateToNum('2001-03-12')
```

    ## [1] 2001.191

``` r
dateToNum('2008-03-12')
```

    ## [1] 2008.194

We put in a data frame of species with unique pairs of locations and dates and find their corresponding pisco datasets

``` r
nearPisWDate <- function(i, sea_star_dt, pisco_dt) {
  ss <- sea_star_dt[i, ]
  ss_loc <- c(ss$longitude, ss$latitude)
  # change the date into a number
  ss_date_n <- dateToNum(ss$survey_date)
  
  min_dis <- 100000
  which_pis <- -1
  for (j in 1:nrow(pisco_dt)) {
    cur_pisco <- pisco_dt[j, ]
    cur_pisco_date <- c(cur_pisco$begin, cur_pisco$end)
    cur_pisco_begin <- dateToNum(cur_pisco_date[1])
    cur_pisco_end <- dateToNum(cur_pisco_date[2])
    if (ss_date_n > cur_pisco_end | ss_date_n < cur_pisco_begin) {
      next
    }
    
    cur_pisco_loc <- c(cur_pisco$longitude, cur_pisco$latitude)
    dis <- (ss_loc[1] - cur_pisco_loc[1])**2 + (ss_loc[2] - cur_pisco_loc[2])**2
    if (dis < min_dis) {
      min_dis <- dis
      which_pis <- j
    }
  }
  if (min_dis > 2) {
    return(-1)
  }
  #result <- list("PISCO" = which_pis, "distance" = min_dis)
  return(which_pis)
}
```

``` r
ca_which_pis <- c()
for (m in 1:nrow(ca_unique_ld)) {
  ca_which_pis <- c(ca_which_pis, nearPisWDate(m, ca_unique_ld, allLocationNDate))
}
write.csv(ca_which_pis, '../data/ca_sea_star_vs_pisco.csv')
```

``` r
ca_unique_ld[163, ]
ca_which_pis$x[163]
```

Next we define a function to combine species datasets and their corrsponding PISCO datasets.

``` r
find_the_pisco <- function(df, data_list) {
  rt <- data.frame()
  for (i in 1:length(data_list)) {
    cur = data_list[i]
    if (cur == -1) {
      next
    }
    cur <- data.frame(latitude = df$latitude[cur], longitude = df$longitude[cur], begin = df$begin[cur], end = df$end[cur], ID = df$ID[cur], species_ind = c(i), pis_ind = c(cur))
    rt <- rbind(rt, cur)
  }
  return(rt)
}
```

``` r
species_pis <- find_the_pisco(allLocationNDate, ca_which_pis$x)
species_pis
```

``` r
write.csv(species_pis, '../data/species_which_pisco.csv')
```

``` r
# get the node of this metadata using `dataOne` package
library(dataone)
library(XML)
locations <- resolve(cn, id)
mnId <- locations$data[2, "nodeIdentifier"]
mn <- getMNode(cn, mnId)


downl <- function(i) {
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
  write.csv(df, file = paste0("../data/downloaded/d", i, ".csv"))
  return(table_id)
}
```

``` r
  id <- 'doi:10.6085/AA/SHB001_021ADCP020R00_20021017.50.1'

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
  write.csv(df, file = paste0("../data/dt", 1, ".csv"))
```

``` r
dt1 <- read.csv("../data/dt1.csv", stringsAsFactors = FALSE)
```

For each data entry of `species_pis`, extract the temprature information from its corresponding PISCO dataset. Define a function to download a PISCO dataset by its index in `dataset_w_pop`.

``` r
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

``` r
get_temp <- function(pisco_df, spe_pis, ) {
  spe_ind <- species_pis$species_ind
  pis_ind <- species_pis$pis_ind
  for (i in 1:nrow(species_pis)) {
    cur_spe <- spe_ind[i]
    cur_pis <- pis_ind[i]
    if (cur_pis >= 926) {
      cur_pis <- cur_pis + 1
    }
    
    pisco_df <- downl_pis(cur_pis)
    ### if you've downloaded all the datasets:
    ### pisco_df <- read.csv(file = paste0("../data/d", cur_pis, ".csv"), stringsAsFactors = FALSE)
    
    # discard if temp == 9999.00, which is equivalent to NA for PISCO datasets
    tem <- filter(pisco_df, date == , temp_c != 9999.00)
  }

  
}
```

``` r
dt1_tem <- filter(dt1, date == '2002-11-03', temp_c != 9999.00)
unique(dt1$date)
ggplot(data = dt1_tem, aes(x=X, y=temp_c)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```

Sea Star Wasting Syndrome
=========================

So first we get the [sea stars data](https://www.eeb.ucsc.edu/pacificrockyintertidal/explore-the-data/index.html). We can see the data is inside four states.

``` r
ssws <- read.csv("../data/sswd_sea_star_observations_2019_0411.csv", stringsAsFactors = FALSE)
```

``` r
nearPisWDateWS <- function(i, sea_star_dt, pisco_dt) {
  ss <- sea_star_dt[i, ]
  ss_loc <- c(ss$longitude, ss$latitude)
  # change the date into a number
  ss_date_n <- dateToNum(ss$sample_date)
  
  min_dis <- 100000
  which_pis <- -1
  for (j in 1:nrow(pisco_dt)) {
    cur_pisco <- pisco_dt[j, ]
    cur_pisco_date <- c(cur_pisco$begin, cur_pisco$end)
    cur_pisco_begin <- dateToNum(cur_pisco_date[1])
    cur_pisco_end <- dateToNum(cur_pisco_date[2])
    if (ss_date_n > cur_pisco_end + 0.5 | ss_date_n < cur_pisco_begin - 0.5) {
      next
    }
    
    cur_pisco_loc <- c(cur_pisco$longitude, cur_pisco$latitude)
    dis <- (ss_loc[1] - cur_pisco_loc[1])**2 + (ss_loc[2] - cur_pisco_loc[2])**2
    if (dis < min_dis) {
      min_dis <- dis
      which_pis <- j
    }
  }
  result <- list("PISCO" = which_pis, "distance" = min_dis)
  return(result)
}
```

``` r
nearPisWDateWS(100, ssws, allLocationNDate)
```

``` r
ssws_which_pis <- c()
for (m in 1:nrow(ssws)) {
  ssws_which_pis <- c(ssws_which_pis, nearPisWDateWS(m, ssws, allLocationNDate))
}
```

``` r
pisco_mean_date <- c()
for (n in 1:length(allLocationNDate$begin)) {
  pisco_mean_date <- c(pisco_mean_date, (dateToNum(allLocationNDate$end[n]) + dateToNum(allLocationNDate$begin[n]))/2)
}
```

``` r
max(pisco_mean_date)
```

``` r
min(pisco_mean_date)
```

``` r
length(ssws_which_pis)
```

``` r
write.csv(ssws_which_pis, '../data/ssws_vs_pisco.csv')
```

``` r
unique(ssws_which_pis)
```

``` r
ssws_date <- c()
for (n in 1:length(ssws$sample_date)) {
  ssws_date <- c(ssws_date, dateToNum(ssws$sample_date[n]))
}
```

``` r
max(ssws_date)
```

``` r
min(ssws_date)
```
