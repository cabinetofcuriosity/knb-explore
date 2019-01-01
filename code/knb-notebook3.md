KNB 3
================
Yuqing Lu
11/7/2018

# Get the most popular headers in KNB 
Ciera suggested that looking for the most popular headers of all the datasets should be fun. I thought that would be interesting but had no idea where to start. So I went to the KNB website and joined their Slack channel. One of the staff, Matt Jones, helped get all the attributes of 69957 datasets, extracted from their metadata files using their API.

``` r
# load the package
library(dataone)

# set to the KNB database
cn <- CNode("PROD")
mn <- getMNode(cn, "urn:node:KNB")

# this gives a data frame of the first 1000 metadata files 
# with only their ID and attribute information
all <-
  list(
    q = "formatType:METADATA+AND+attributeName:*",
    fl = "identifier,formatId,attributeName,attributeDescription",
    wt = "xml",
    rows = "1000",
    start = "0"
  )
allre <- query(mn, solrQuery = all, as = "data.frame")
```

Then I used a for loop to merged the 70 data frames and the resulting data frame is `allre`, saved as a csv file. This takes really long.

``` r
# load the package
library(dataone)

# set to the KNB database
cn <- CNode("PROD")
mn <- getMNode(cn, "urn:node:KNB")

allre <- data.frame()
for (i in seq(0, 69000, 1000)) {
  all <-
  list(
    q = "formatType:METADATA+AND+attributeName:*",
    fl = "identifier,formatId,attributeName,attributeDescription",
    wt = "xml",
    rows = "1000",
    start = paste0(i)
  )
  allre <- rbind(allre, query(mn, solrQuery = all, as = "data.frame"))
}
write.csv(allre, file = "~/Documents/nahis/data/knb-attrs.csv")
```

Next I extract all the attribute names, in lower case, in the data frame and put them into one vector, `attrs`.

Of course, some attributes may have different names even though they mean the same thing as other attributes. For example, some datasets may have an attribute names "len", which is the same as "length"; or some may have abbreviations. And this may be harmful to this exploration. So we may want to change the similar attribute

``` r
# this function gives all the attribute names for one metafile, ie, one row of allre
get_attr <- function(x) {
  tolower(unlist(strsplit(allre$attributeName[x], "\\s+")))
}
attrs <- sapply(1:69957, get_attr, simplify=TRUE)
```

Now I make a frequency table of all the attribute names and make a barchart of them in descending order of frequency. Thus we can see which attributes are the most popular.

``` r
attr_tab <- table(factor(unlist(attrs)))
most_pop <- sort(attr_tab, decreasing=TRUE)
write.csv(most_pop, file = "~/Documents/nahis/data/knb-pop-attrs.csv")
head(most_pop, 100)
```

``` r
most_pop <- read.csv("../data/knb-pop-attrs.csv", stringsAsFactors = FALSE)
pop_20 <- head(most_pop, 20)
library(ggplot2)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
ggplot(data = pop_20, aes(x=reorder(Var1, Freq), y=Freq)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal() +
  coord_flip()
```

![](../image/6to20-1.png)

So we see that the first five attributes don't have any special meanings. They just represend time/date. So let's get rid of all the "boring" attributes.

``` r
library(stringr)
attr_not_boring <- 
  filter(most_pop, !str_detect(
                    Var1, "(year|date|time|day|site|month|location|station)"))
```

``` r
pop6to20 <- slice(attr_not_boring, 1:20)
ggplot(data = pop6to20, aes(x=reorder(Var1, Freq), y=Freq)) +
  geom_bar(stat="identity", fill = "lightblue") +
  theme_minimal() +
  coord_flip()
```

![](../image/unnamed-chunk-7-1.png)

Let's extract the metafiles with common popular attributes. Now it seems that the datasets that have the first 12 "not boring popular" attributes in common come from the same cluster of datasets, since they have exactly the same attributes and attribute descriptions.

``` r
all_files <- read.csv("../data/knb-attrs.csv", stringsAsFactors = FALSE)
pop_vars <- slice(attr_not_boring, 1:12)$Var1
dataset_w_pop <- filter(all_files, grepl(pop_vars[1], all_files$attributeName))
for (i in 2:length(pop_vars)) {
    dataset_w_pop <- filter(dataset_w_pop, grepl(pop_vars[i], dataset_w_pop$attributeName))
}
id <- dataset_w_pop[1, 2]
```
# Download all the selected datasets
I want to do some analysis on these datasets.  
So I download all the data using their [API](https://github.com/DataONEorg/rdataone) with their metadata files.  
One metadata file shows:
> This metadata record describes bottom-mounted ADCP data collected at Sand Hill Bluff, California, USA, by PISCO. Measurements were collected using an RDI 600 kHz Workhorse Sentinel ADCP beginning 2001-03-12.  The instrument depth was 020 meters, in an overall water depth of 021 meters (both relative to Mean Sea Level, MSL).  The instrument was programmed with a sampling interval of 2.0 minutes and a vertical resolution of 1 meter."

Right now, according to the KNB staff, there is something wrong with the website so we can't directly get the data from KNB. We need to do some trick. So going to DataOne website, we can see [this](https://search.dataone.org/view/doi:10.6085/AA/SHB001_021ADCP020R00_20010312.50.1) is exactly the same dataset.  
By inspecting the HTML code of the website, we found the ID for the data table is "doi:10.6085/AA/SHB001_021ADCP020R00_20010312.40.1". We see the IDs have the same pattern: "doi:10.6085/AA/" + unique ID(extracted from the metadata file with the help of an [XML tutorial](https://www.stat.berkeley.edu/~statcur/Workshop2/Presentations/XML.pdf)).  
We can check later by comparing the total row numbers. 

```{r, eval = FALSE}
# load the package
library(XML)
library(dataone)

# get the node of this metadata using `dataOne` package
locations <- resolve(cn, id)
mnId <- locations$data[2, "nodeIdentifier"]
mn <- getMNode(cn, mnId)

# download the metadata file to find the data table
metadata <- rawToChar(getObject(mn, id))
doc = xmlRoot(xmlTreeParse(metadata, asText=TRUE, trim = TRUE, ignoreBlanks = TRUE))
print(doc[1])
tf <- tempfile()
saveXML(doc, file="d1.xml")

# now extract the node that has the data table's information
node <- getNodeSet(doc, "//objectName")
table_id <- xmlValue(node[[1]])
```

We put the data frame into a .csv file:  
```{r}
# we can see that the ids have the pattern
dataRaw <- getObject(mn, paste0("doi:10.6085/AA/", table_id))
dataChar <- rawToChar(dataRaw)
theData <- textConnection(dataChar)
df <- read.csv(theData, stringsAsFactors=FALSE, header = TRUE, sep = " ")
write.csv(df, file = "../data/d11.csv")
```


```{r, eval = FALSE}
id <- dataset_w_pop[1, 3]
  
# get the node of this metadata using `dataOne` package
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

sapply(39:nrow(dataset_w_pop), downl)

```


