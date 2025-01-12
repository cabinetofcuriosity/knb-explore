KNB Notebook
================

KNB
===

> The Knowledge Network for Biocomplexity (KNB) is an international repository intended to facilitate ecological and environmental research.

You don't have to become a user to access data in KNB.

Accessing KNB Data
==================

You could either search and download that specific data or use their [API](https://knb.ecoinformatics.org/api).

To use their API, you need to install the [rDataone](https://github.com/DataONEorg/rdataone) package in Java, Python or R. I chose R, using the following:

``` r
install.packages("dataone")
```

However, when installing the package, I came across this error:

> Configuration failed because redland was not found.

Following its error message, I installed "redland" using the command (MacOS):

``` bash
$ brew install redland 
```

And then I'm able to install the dataone package.

To search and get the result, follow the example on the [website](https://github.com/DataONEorg/rdataone) or use `??dataone` to see manual.

``` r
# load the package first
library(dataone)

# search the KNB database with the keyword "climate"
cn <- CNode("PROD")
mn <- getMNode(cn, "urn:node:KNB")
mySearchTerms <- list(q="abstract:climate",
                     rows= "500", fl="id,title,dateUploaded,abstract,size",
                     sort="dateUploaded+desc")

# show the search result as a data frame
result <- query(mn, solrQuery=mySearchTerms, as="data.frame")
head(result, 10)

# get the id of the first result
result[1, c("id", "title")]
id <- result[1,'id']
```

Now get the data from the database using `XML` and `dataone` packages.

``` r
library(XML)
metadata <- rawToChar(getObject(mn, id))
doc = xmlRoot(xmlTreeParse(metadata, asText=TRUE, trim = TRUE, ignoreBlanks = TRUE))
print(doc[1])
tf <- tempfile()
saveXML(doc, file="test.xml")
file.show(tf)
```

Write a data frame to a .csv file.

``` r
dataRaw <- getObject(mn, "knb.92077.1")
dataChar <- rawToChar(dataRaw)
theData <- textConnection(dataChar)
df <- read.csv(theData, stringsAsFactors=FALSE)
write.csv(df, file = "~/Documents/knb-susquehanna-river-flow.csv")
```

Since there are a lot of XML files in the databases, I need to learn about XML, which looks really daunting. [w3schools](https://www.w3schools.com/xml/xml_whatis.asp) is a good website for introduction to XML, yet its method of reading xml files is pretty complicated.

So I googled "import data from XML using R", and found the the Youtube [tutorial](https://www.youtube.com/watch?v=1cM_ZNZ9hhE) of using XML package.

Installing `XML`

``` r
install.packages("XML")
install.packages("methods")
```

``` r
library(XML)
library(methods)
xml_tf <- xmlParse(tf)
df <- xmlToDataFrame("xml_tf")
```

When I try to change the the xml file to a data frame it gives me an error. So I'll keep working on xml files.
