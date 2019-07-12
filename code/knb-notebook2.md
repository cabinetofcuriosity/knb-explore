KNB Notebook2
================

Scope of the database
=====================

What are the measurements? Are there variables that are required for each data point? Try to explain in a programmatic way.

Since KNB has multiple datasets, it has info of all types of animals, including birds, fish, mammals, etc. Depending on the dataset, it has different types of variables and measurements.

Take one dataset for example, which is the one I downloaded in the first notebook,

``` r
river <- read.csv("~/Documents/nahis/knb-susquehanna-river-flow.csv")
str(river)
```

    ## 'data.frame':    227035 obs. of  10 variables:
    ##  $ X     : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ Agency: Factor w/ 1 level "USGS": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ a     : int  1570500 1570500 1570500 1570500 1570500 1570500 1570500 1570500 1570500 1570500 ...
    ##  $ date  : Factor w/ 4674 levels "1/1/2000","1/1/2003",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ time  : Factor w/ 441 levels "0:00","0:02",..: 1 3 9 16 24 231 239 316 324 331 ...
    ##  $ zone  : Factor w/ 2 levels "EDT","EST": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ flow  : int  18500 18500 18500 18500 18500 18500 18400 18400 18400 18200 ...
    ##  $ b     : Factor w/ 2 levels "A","A:[91]": 2 2 2 2 2 2 2 2 2 2 ...
    ##  $ month : int  1 1 1 1 1 1 1 1 1 1 ...
    ##  $ year  : int  2000 2000 2000 2000 2000 2000 2000 2000 2000 2000 ...

flow: cubic feet per second
a: USGS gage number [google "1570500 usgs"](https://pubs.usgs.gov/of/2016/1038/ofr20161038.pdf)

Another dataset

``` r
mya <- read.csv("~/Documents/nahis/knb.92072.1.csv")
str(mya)
```

    ## 'data.frame':    565 obs. of  36 variables:
    ##  $ Line         : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ Region       : Factor w/ 2 levels "LB","UB": 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ River        : Factor w/ 6 levels "Chester River",..: 6 6 6 6 6 6 6 6 6 6 ...
    ##  $ Sitename     : Factor w/ 133 levels " Spaniard Pt 1",..: 89 90 91 92 93 94 95 96 97 42 ...
    ##  $ Site         : Factor w/ 31 levels "Broad Bay","Buoy Rock",..: 20 20 20 21 21 21 22 22 22 9 ...
    ##  $ Number       : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ Season       : Factor w/ 3 levels "Fall","Spring",..: 1 1 1 1 1 1 1 1 1 1 ...
    ##  $ Year         : int  2011 2011 2011 2011 2011 2011 2011 2011 2011 2011 ...
    ##  $ Date         : int  40849 40849 40849 40849 40849 40849 40849 40849 40849 40849 ...
    ##  $ MMDDYY       : Factor w/ 46 levels "10/17/2012","10/18/2011",..: 10 10 10 10 10 10 10 10 10 10 ...
    ##  $ Lat          : num  37.4 37.4 37.4 37.4 37.4 ...
    ##  $ Long         : num  76.7 76.7 76.7 76.7 76.7 ...
    ##  $ Temp         : num  19.7 19.7 19.7 19.3 19.3 19.3 17.4 17.4 17.4 17.4 ...
    ##  $ Sal          : num  5.3 5.3 5.3 5.4 5.4 5.4 13 13 13 13 ...
    ##  $ DO           : num  10.2 10.2 10.2 16.4 16.4 16.4 6.9 6.9 6.9 6.9 ...
    ##  $ Rays         : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ Crabs        : num  60 60 60 35 35 35 40 40 40 40 ...
    ##  $ Crab.length  : num  27.2 27.2 27.2 13.4 13.4 13.4 29 29 29 29 ...
    ##  $ Fish         : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ Fish.Length  : int  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ Volume       : int  100 240 310 150 75 60 850 2790 3380 315 ...
    ##  $ T.AFDW       : num  0 0.113 0.168 0.018 0.006 0 0 0.268 0.112 0 ...
    ##  $ M.AFDW       : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ T.No         : int  0 4 6 1 1 0 0 3 1 0 ...
    ##  $ T.Dens       : num  0 36.36 54.55 9.09 9.09 ...
    ##  $ T.Mean.length: num  NA 19.2 19.4 16.4 12.7 NA NA 25.3 31.2 NA ...
    ##  $ M.No         : int  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ M.Dens       : num  0 0 0 0 0 0 0 0 0 0 ...
    ##  $ M.Mean.length: num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ M.Mean.Dis   : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ T.Mean.Dis   : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ M.Max.Dis    : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ T.Max.Dis    : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ M.Prop.Dis   : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ T.Prop.Dis   : num  NA NA NA NA NA NA NA NA NA NA ...
    ##  $ Sb           : Factor w/ 5 levels "GR","MU","SA",..: 5 5 5 5 5 5 5 5 5 5 ...

``` r
# columns that have no NA's
table(factor(which(!is.na(mya), arr.ind=TRUE)[, 2]))
```

    ## 
    ##   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17  18 
    ## 565 565 565 565 565 565 565 565 565 565 565 565 414 414 414 360 348 336 
    ##  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36 
    ## 324 324 216 216 216 565 565 365 565 565 210 188 246 188 246 216 283 522

Data Quality
============

I don't see any documentation on the standards. Units and abbreviations are quite vague. I have to go to the original site to refer to other similar datasets. Or google some unique data entries, such as IDs.

But if it's about river, we can check the climate there and get a sense of rainfall and water stream, etc. Also for animal sizes, we can compare with their avarage sizes to detect possible outliers.

Completeness
------------

> requires that a particular column, element or class of data is populated and does not feature null values or values in place of nulls (e.g. N/As).

``` r
na1 <- which(is.na(river), arr.ind=TRUE)
na2 <- which(is.na(mya), arr.ind=TRUE)
na1
```

    ##      row col

``` r
head(na2, 50)
```

    ##       row col
    ##  [1,]  25  13
    ##  [2,]  26  13
    ##  [3,]  27  13
    ##  [4,]  28  13
    ##  [5,]  29  13
    ##  [6,]  30  13
    ##  [7,]  31  13
    ##  [8,]  32  13
    ##  [9,]  33  13
    ## [10,]  34  13
    ## [11,]  35  13
    ## [12,]  36  13
    ## [13,] 217  13
    ## [14,] 218  13
    ## [15,] 219  13
    ## [16,] 220  13
    ## [17,] 221  13
    ## [18,] 222  13
    ## [19,] 223  13
    ## [20,] 224  13
    ## [21,] 225  13
    ## [22,] 226  13
    ## [23,] 227  13
    ## [24,] 228  13
    ## [25,] 229  13
    ## [26,] 230  13
    ## [27,] 231  13
    ## [28,] 232  13
    ## [29,] 233  13
    ## [30,] 234  13
    ## [31,] 235  13
    ## [32,] 236  13
    ## [33,] 237  13
    ## [34,] 238  13
    ## [35,] 239  13
    ## [36,] 240  13
    ## [37,] 241  13
    ## [38,] 242  13
    ## [39,] 243  13
    ## [40,] 244  13
    ## [41,] 245  13
    ## [42,] 246  13
    ## [43,] 247  13
    ## [44,] 248  13
    ## [45,] 249  13
    ## [46,] 250  13
    ## [47,] 251  13
    ## [48,] 252  13
    ## [49,] 253  13
    ## [50,] 254  13

Consistency
-----------

> Something that tests whether one fact is consistent with another e.g. gender and title in a CRM database.

We can check seasons and dates/datas and years.

Uniqueness
----------

> Are all the entities or attributes within a dataset unique?

By now I haven't seen any duplicate attributes. Also, since I have data frames, I think all the attributes should be unique and otherwise R will change them automatically.

Integrity
---------

> Are all the relationships populated for a particular entity – for example its parent or child entities?

No. For crabs and fish, we only have their lengths.

Conformity
----------

> Does the data conform to the right conventions and standards. For example a value may be correct but follow the wrong format or recognised standard.

The units are generally unclear.

Accuracy
--------

> the hardest dimension to test for as this often requires some kind of manual checking by a Subject Matter Expert (SME).

I think data coming from a governmental institute or a university is pretty accurate.

What are the variables that are most interesting to you?
========================================================

> At some point you will need to refine the scope of your project. You likely cannot explore ALL the data. Are their questions about the that are particularly interesting to you? Questions can either be about the quality of the data or of biological significance.

Right now I'm thinking about how climate changes affect one species' living conditions. Or maybe just river flows? I think biological significance sounds pretty interesting but I'm not sure if there are a lot of things to be done. Quality of data is also good to study as well...

Skills
======

> Reiterate what skills you particullarly interested in learning. Do you see a clear path from this database to level up on those skills?

I'm most interesting in machine learning.
If I do the quality of data, I think I can do something similar to what I did in a Data8 project, which is making a test set and compare the data with the set to determine the outliers. And I can explore more on that, such as advanced testing methods. On the other hand, biological significance will lead me to a data visualization path, I think. I will compare different pairs of data attributes and find their relationships?

Handle the data
===============

> Is there something that could be done to the data on the database side that would make your life easier when using this data? Do you wish it was in json over XML? Do you wish that there was a tool in Python that would connect to the database? Did you find the documentation incredibly hard to follow? What are some things you googled that helped you? What are the things you googled that had no answer but wish there was?

I'm actually able to read XML now using Atom because it colors different nodes. I also changed some settings to "soft wrap at preffered line length" and it makes xml way easier to read! I'm almost in love with it!
Useful guide to limit line lengths in atom [here](https://stackoverflow.com/questions/49616864/limiting-line-length-in-atom)

As for some data that is stored in the xml file, I can extract that part and use the `xmlToDataFrame` method in `XML` package. [tutorial mentioned last time](https://www.youtube.com/watch?v=1cM_ZNZ9hhE).
