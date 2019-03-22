---
title: "Understanding and Processing KNB PISCO data"
output: github_document
---

```{r}
# Load some packages and import some data frames
library(ggplot2)
library(dplyr)
d203 <- read.csv("../data/d203.csv", stringsAsFactors = FALSE)
dataset_w_pop <- read.csv("../data/pop_ds.csv", stringsAsFactors = FALSE)
```

Let's look at one dataset, understand its attributes and do some processing. 
```{r}
head(d203, 100)
```

To obtain information of the columns, we need the metadata file. So we download the metadata file for this dataset from KNB. 

```{r, eval = FALSE}
dataset_w_pop <- read.csv("../data/pop_ds.csv", stringsAsFactors = FALSE)
library(XML)
library(dataone)
id <- dataset_w_pop[203, 3]
# get the node of this metadata using `dataOne` package
locations <- resolve(cn, id)
mnId <- locations$data[2, "nodeIdentifier"]
mn <- getMNode(cn, mnId)

# download the metadata file to find the data table
metadata <- rawToChar(getObject(mn, id))
doc = xmlRoot(xmlTreeParse(metadata, asText=TRUE, trim = TRUE, ignoreBlanks = TRUE))
saveXML(doc, file="d203.xml")
```

# Overview of the dataset 
From the metadata file downloaded, we obtain the following information:  
## About the organization  
PISCO is a large-scale marine research program that focuses on understanding the near-shore ecosystems of the U.S. West Coast. An interdisciplinary collaboration of scientists from four universities, PISCO integrates long-term monitoring of ecological and oceanographic processes at dozens of coastal sites with experimental work in the lab and field. We explore how individual organisms, populations, and ecological communities vary over space and time.  Findings are applied to issues of ocean conservation and management, and are shared through our public outreach and student training programs.

## Data Collection  
These data were collected by PISCO to understand the physical processes of the inner continental shelf and their potential effects on marine ecology.

This metadata record describes bottom-mounted ADCP data collected at Fogarty Creek, Oregon, USA, by PISCO. Measurements were collected using an RDI 600 kHz Workhorse Sentinel ADCP beginning 2002-08-02.  The instrument depth was 014 meters, in an overall water depth of 15 meters (both relative to Mean Sea Level, MSL).  The instrument was programmed with a sampling interval of 2.0 minutes and a vertical resolution of 1 meter.

Fogarty Creek (ADCP) 15m: FC15C: The nearshore mooring is located offshore of Fogarty Creek, approximately 4km South West of the onshore Lincoln Beach site, about 3.6km North of Depoe Bay, Oregon, USA.  The base of the mooring is in 15 meters of water below Mean Sea Level (MSL), and it extends up to 4 meters below MSL.  The altitudeMinimum and altitudeMaximum tags in this initial coverage section refer to the ADCP measurement range and are also referenced to MSL.  They do not represent the overall water depth.  Note the nominal range of the ADCP may extend from near-bottom (a depth expressed as a negative altitude) to slightly above MSL (a height expressed as a positive altitude)

<westBoundingCoordinate>-124.074776
<eastBoundingCoordinate>-124.074776
<northBoundingCoordinate>44.841800
<southBoundingCoordinate>44.841800
<boundingAltitudes>
  <altitudeMinimum>-014
  <altitudeMaximum>9
  <altitudeUnits>meter

Methods for PISCO bottom-mounted ADCP data collection and quality-control are available online, see the protocol citation.  Data were collected using an RDI 600 kHz Workhorse Sentinel ADCP set to 40 pings per 2.0-minute ensemble, with a vertical resolution of 1 meter.


## Data Attributes
- date: Greenwich Mean Time calendar date of each ADCP measurement record, using ISO 8601 standard format YYYY-MM-DD

- time: Greenwich Mean Time of each ADCP measurement record, to the 1/100th of a second, using ISO 8601 standard format hh:mm:ss.ssZ.  The &apos;Z&apos; indicates &apos;Zero Meridian&apos; (or Greenwich Mean Time) and must be included. [00z: https://www.weather.gov/jetstream/time]

- yearday: Greenwich Mean Time of each ADCP measurement record, expressed as fractional decimal days since Jan. 1 of the year measurement was made.  For example, 12 noon GMT on Jan. 2 is represented by yearday 1.5, NOT yearday 2.5.

- height: The height of the velocity, temperature, or pressure measurement, in meters above the sea bottom.

- depth: The depth of the velocity, temperature, or pressure measurement, in positive meters below Mean Sea Level (MSL).  Bins above MSL are represented with negative depths.

- waterdepth: The fluctuating water depth (sea floor to sea surface distance) at the measurement site, in meters, as determined by location of sea surface from ADCP pressure measurements, or if pressure is unavailable, the maximum in ADCP echo intensity.
[9999.0: missing data]

- temp_c: seawater temperature from ADCP sensor

- pressure: pressure measured at the ADCP sensor, measured in decibars with a precision defined as the effective resolution of the instrument

- intensity: ADCP echo intensity (or backscatter), in RDI counts.  This value represents the average of all 4 beams, rounded to the nearest whole number.

- data_quality: A post-processing quantitative data quality indicator.  Specifically, RDI percent-good #4, in earth coordinates (percentage of successful 4-beam transformations), expressed as a percentage, 50, 80, 95, etc.

- eastward: True eastward velocity measurements.  Negative values represent westward velocities. [metersPerSecond]

- northward: True northward velocity measurements.  Negative values represent southward velocities.

- upwards: True upwards velocity measurements.  Negative values represent downward velocities.

- flag: data flag used to qualify data as bad, questionable, etc. [0 - good; 999 - bad]

- errorvelocity: The difference of two vertical velocities, each measured by an opposing pair of ADCP beams [upward and downward]



# Analyzing Attributes
Now let's take a closer look at the attributes to see what happens to them over time.  
```{r}
unique(d203$time)
```
We can tell that the time is incresing by the result above, along with the attribute information of `time` from the metadata file.  
Because the time is always increasing, we can just use the index number, i.e., the first column as the time axis for now.  

> The wording of the metadata is confusing (and wrong) - that two minute sampling interval doesn't mean that the instrument sampled for two minutes, rather, two minutes is the "time per ensemble", which is defined as the "minimum interval between data collection cycles". So all that means is that the ADCP sampled every two minutes, but not that data collection lasted a full two minutes. The metadata further states that the unit sampled 40 pings per ensemble, but if you look at the data you actually have 20 pings per ensemble. What the metadata and CSV don't tell me is the TP - or the time between pings, so there's no real way to tell from this how those 20 pings were spaced out. The unit could have sent out 1 ping per second for 20 seconds, then shut down for 1 minute and 40 seconds, or it could have spread them out further during that two minute ensemble. 
> the ping in row 1 happened before the ping in row 2, and so on

```{r, 6to20, eval=FALSE}
ggplot(data = d203, aes(x=X, y=height)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```
```{r}
ggplot(data = d203, aes(x=X, y=depth)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=waterdepth)) +
  geom_line(color = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=temp_c)) +
  geom_line(color = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=pressure)) +
  geom_point(color = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=data_quality)) +
  geom_point(color = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=eastward)) +
  geom_point(color = "lightblue") +
  theme_minimal()
```

It looks dense so we extract part of it to show the pattern:
```{r}
ggplot(data = head(d203, 500), aes(x=X, y=eastward)) +
  geom_line(color = "lightblue") +
  ggtitle('first 500 eastward') + 
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=northward)) +
  geom_line(color = "lightblue") +
  theme_minimal()
```
Similar as above:
```{r}
ggplot(data = head(d203, 500), aes(x=X, y=northward)) +
  geom_line(color = "lightblue") +
  ggtitle('first 500 northward') + 
  theme_minimal()
```


```{r}
ggplot(data = d203, aes(x=X, y=upwards)) +
  geom_line(color = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = head(d203, 500), aes(x=X, y=upwards)) +
  geom_line(color = "lightblue") +
  ggtitle('first 500 northward') + 
  theme_minimal()
```


# Location  
We want to connect the sea stars data to PISCO data based on their locations.  

## Sea Stars
So first we get the sea stars data inside california 

```{r}
sea_stars <- read.csv("../data/phototranraw_download.csv", stringsAsFactors = FALSE)
sea_stars_ca <- sea_stars[sea_stars$state_province == 'California', ]
# add ids to represent each row
sea_stars_ca <- mutate(sea_stars_ca, id = 1:nrow(sea_stars_ca))
sea_stars_ca
```

```{r}
unique(sea_stars$state_province)
```

## PISCO
Some setup for getting the location data.
```{r}
library(dataone)
library(XML)

id <- dataset_w_pop[1, 3]
cn <- CNode("PROD")

# get the node of this metadata using `dataOne` package
locations <- resolve(cn, id)
mnId <- locations$data[2, "nodeIdentifier"]
mn <- getMNode(cn, mnId)
```

```{r}
getLocation <- function(i) {
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
  
  thisLoc <- data.frame(longitude = c((1/2)*(west + east)), latitude = c((1/2)*(north + south)), ID = c(id))
  
  return(thisLoc)
}
```

```{r}
getLocation(926)
```

> Error in .local(x, ...) : get() error: READ not allowed on doi:10.6085/AA/JALXXX_015ADCP015R00_20030529.50.2

After testing, 926th ID doesn't work

```{r}
allLocation <- data.frame()
for (i in 1:925) {
  allLocation <- rbind(allLocation, getLocation(i))
}

for (i in 927:nrow(dataset_w_pop)) {
  allLocation <- rbind(allLocation, getLocation(i))
}
```

We save this data frame into a `.csv` file.
```{r}
write.csv(allLocation, '../data/pisco-locations.csv')
allLocation
```

## Find the nearest PISCO site for each sea star data entry
```{r}
nearSite <- function(i, sea_star_dt, pisco_dt) {
  cur_sea_star <- sea_star_dt[i, ]
  cur_sea_star_loc <- c(cur_sea_star$longitude, cur_sea_star$latitude)
  
  min_dis <- 100000
  which_pis <- 0
  for (j in 1:nrow(pisco_dt)) {
    cur_pisco <- pisco_dt[j, ]
    cur_pisco_loc <- c(cur_pisco$longitude, cur_pisco$latitude)
    dis <- (cur_sea_star_loc[1] - cur_pisco_loc[1])**2 + (cur_sea_star_loc[2] - cur_pisco_loc[2])**2
    if (dis < min_dis) {
      min_dis <- dis
      which_pis <- j
    }
  }
  return(which_pis)
}
```

```{r}
ca_which_pis <- c()
for (m in 1:nrow(sea_stars_ca)) {
  ca_which_pis <- c(ca_which_pis, nearSite(m, sea_stars_ca, allLocation))
}

```

