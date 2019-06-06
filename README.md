# Basic Info
__Author__: [Yuqing Lu](https://github.com/lynluyq)  

__Database__: [KNB](https://knb.ecoinformatics.org/)  

__Contents__:  

notebook:
- [knb-notebook1.md](https://github.com/cabinetofcuriosity/knb_explore/blob/master/code/knb-notebook1.md): downloading the API and understanding its usage
- knb-notebook2.Rmd: exploring the database using their API
- knb-notebook3.Rmd: find the most popular headers in the KNB database and download the datasets that have these headers
- knb-process.md: understanding PISCO datasets and analyzing their attributes
- knb-sskat.md: species count and size vs PISCO

data: 
- [knb-attrs.csv](https://github.com/cabinetofcuriosity/knb_explore/blob/master/data/knb-attrs.csv):  
Extracted from the KNB website, around 700,000 rows. Each row contains information of an individual dataset that has header information in its metadata file.  
- [knb-pop-attrs.csv](https://github.com/cabinetofcuriosity/knb_explore/blob/master/data/knb-pop-attrs.csv):  
Common headers of the datasets, ordered by their frequencies.
- [seastarkat_size_count_totals_download.csv](https://github.com/cabinetofcuriosity/knb_explore/blob/master/data/seastarkat_size_count_totals_download.csv):
Species count and size data (sea stars and katharina only) requested from [MARINe](https://marine.ucsc.edu/explore-the-data/contact/index.html).
- [sswd_sea_star_observations_2019_0411.csv](https://github.com/cabinetofcuriosity/knb_explore/blob/master/data/sswd_sea_star_observations_2019_0411.csv):
Sea star wasting syndrome data requested from [MARINe](https://marine.ucsc.edu/explore-the-data/contact/index.html).
- downloaded csv files: two large to show online

# Exploring KNB
This repo has the data, code and reports for my exploratary analysis on KNB, which is a website that aggregates ecology related datasets. 

## Accessing the data in KNB
In order to access the data in KNB programmatically, I downloaded their API. Then following Ciera's suggestion, I was able to find the most popular headers in their database, under the help of the KNB staff. Playing with the headers, I decided to work on the datasets from [PISCO](http://www.piscoweb.org) and I need combine all the datasets first, which are around 200GB in total. 
