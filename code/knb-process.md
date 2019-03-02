---
title: "Understanding and Processing KNB PISCO data"
output: html_notebook
---

```{r}
library(ggplot2)
library(dplyr)
d203 <- read.csv("../data/d203.csv", stringsAsFactors = FALSE)
```

```{r}
d203

unique(d203$time)
```

```{r}
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
print(doc[1])
tf <- tempfile()
saveXML(doc, file="d203.xml")
```

Because the time is always increasing, we can just use the index number, i.e., the first column as the time axis for now.

```{r, 6to20}
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
  geom_bar(stat="identity", fill = "lightgreen") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=temp_c)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=pressure)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=data_quality)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```

```{r}
ggplot(data = d203, aes(x=X, y=eastward)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```
```{r}
ggplot(data = d203, aes(x=X, y=northward)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```
```{r}
ggplot(data = d203, aes(x=X, y=upwards)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal()
```
These data were collected by PISCO to understand the physical processes of the inner continental shelf and their potential effects on marine ecology.

PISCO is a large-scale marine research program that focuses on understanding the near-shore ecosystems of the U.S. West Coast. An interdisciplinary collaboration of scientists from four universities, PISCO integrates long-term monitoring of ecological and oceanographic processes at dozens of coastal sites with experimental work in the lab and field. We explore how individual organisms, populations, and ecological communities vary over space and time.  Findings are applied to issues of ocean conservation and management, and are shared through our public outreach and student training programs.


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


# Columns
date: Greenwich Mean Time calendar date of each ADCP measurement record, using ISO 8601 standard format YYYY-MM-DD

time: Greenwich Mean Time of each ADCP measurement record, to the 1/100th of a second, using ISO 8601 standard format hh:mm:ss.ssZ.  The &apos;Z&apos; indicates &apos;Zero Meridian&apos; (or Greenwich Mean Time) and must be included. [00z: https://www.weather.gov/jetstream/time]

yearday: Greenwich Mean Time of each ADCP measurement record, expressed as fractional decimal days since Jan. 1 of the year measurement was made.  For example, 12 noon GMT on Jan. 2 is represented by yearday 1.5, NOT yearday 2.5.

height: The height of the velocity, temperature, or pressure measurement, in meters above the sea bottom.

depth: The depth of the velocity, temperature, or pressure measurement, in positive meters below Mean Sea Level (MSL).  Bins above MSL are represented with negative depths.

waterdepth: The fluctuating water depth (sea floor to sea surface distance) at the measurement site, in meters, as determined by location of sea surface from ADCP pressure measurements, or if pressure is unavailable, the maximum in ADCP echo intensity.
[9999.0: missing data]

temp_c: seawater temperature from ADCP sensor

pressure: pressure measured at the ADCP sensor, measured in decibars with a precision defined as the effective resolution of the instrument

intensity: ADCP echo intensity (or backscatter), in RDI counts.  This value represents the average of all 4 beams, rounded to the nearest whole number.

data_quality: A post-processing quantitative data quality indicator.  Specifically, RDI percent-good #4, in earth coordinates (percentage of successful 4-beam transformations), expressed as a percentage, 50, 80, 95, etc.

eastward: True eastward velocity measurements.  Negative values represent westward velocities. [metersPerSecond]

northward: True northward velocity measurements.  Negative values represent southward velocities.

upwards: True upwards velocity measurements.  Negative values represent downward velocities.

flag: data flag used to qualify data as bad, questionable, etc. [0 - good; 999 - bad]

errorvelocity: The difference of two vertical velocities, each measured by an opposing pair of ADCP beams [upward and downward]
