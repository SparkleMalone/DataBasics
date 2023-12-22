# Data Basics (EcoClim)

### Learn the basics of manipulating data in R

In this workshop you will:

1. Download climate data with daymetr 
2. Manipulate data with tidyverse: 
    + Format date elements
    + Use select() to choose variables from a data frame.
    + Use filter() to choose data based on values.
    + Use mutate() to create new variables.
    + Use group_by() and summarize() to work with subsets of data.
    + Use full_join() to merge data sets
3. Visualize data with ggplot2
4. Write basic functions

# Data manipulation with tidyverse

### Load the required libraries
```{r, include=T}
library(daymetr)
library(tidyverse)
```
## What is the climate of Yale-Myers Forest?

Yale-Myers Forest is a 7,840 acre research forest Connecticut. It is a hub for education, research, and harvesting operations within the Yale School Forest System. The forest primarily consists of mixed hardwoods on glacial till soils, featuring a significant presence of hemlock, scattered white pine stands from old field origins, and red pine plantations established in the 1940s. 

To explore the climate of the area, download daily daymet data for (41.952909, -72.123859). Review the arguments for the download_daymet() function.
```{r, include=T}
?download_daymet()
```
To use the download_daymet() function you need the latitude, longitude, start (1990), and end (2018). 

```{r, include=T}
Yale.Myers.daymet <- download_daymet( lat =  41.952909,
                                      lon = -72.123859,
                                      start = 1990,
                                      end = 2018)
```

You now have a nested list that includes information on the site location etc. The true climate data is stored in the "data" part of the nested list. Pull the data out of the list and create a dataframe.

```{r, include=T}
Yale.Myers <- as.data.frame(Yale.Myers.daymet$data)
```
Look at the dataframe:
```{r, include=T}
head(Yale.Myers )
```
Evaluate the data types:
```{r, include=T}
summary( Yale.Myers)
```
Format the time variables and create a date in the format YYYY-mm-dd :

Step 1. Use the year and day of the year to create a year.day variable.
```{r, include=T}
Yale.Myers$year.day <- paste( Yale.Myers$year, Yale.Myers$yday, sep="-") %>% as.Date(format="%Y-%j")
```
Look at your work:
```{r, include=T}
Yale.Myers$year.day
```
Step 2. Format the year.day as YYYY-mm-dd and convert to a date.
```{r, include=T}
Yale.Myers$date <- format( Yale.Myers$year.day, "%Y-%m-%d") %>% as.Date()
```
Look at your work:
```{r, include=T}
Yale.Myers$date
```
Check the class:
```{r, include=T}
class(Yale.Myers$date)
```
The function mutate() creates new columns that are functions of existing variables. Use mutate() to create a month and Julian day (JulianD) from the formated date.

```{r, include=T}
Yale.Myers <- Yale.Myers %>% mutate( month = format( Yale.Myers$date, format="%m" ),
                     julianD = format( Yale.Myers$date, format="%j") )
```
With Daymet data you have to calculate the mean temperature using tmax and tmin:

```{r, include=T}
Yale.Myers$tmean = (0.606 *as.numeric(Yale.Myers$tmax..deg.c.)) + 0.394 * as.numeric(Yale.Myers$tmin..deg.c.)
```
Look at your work:
```{r, include=T}
summary(Yale.Myers$tmean)
```

Use the function rename() to rename tmax..deg.c to tmax, tmin..deg.c to tmin, and prcp..mm.day. to prcp:
```{r, include=T}
Yale.Myers.rn <-Yale.Myers %>% rename(tmax=tmax..deg.c., tmin=tmin..deg.c., prcp = prcp..mm.day.)
```
Look at your work:
```{r, include=T}
head(Yale.Myers.rn )
```
Subset the dataset to include only [date, month, year, julianD, tmax, tmin, tmean, and prcp] :
```{r, include=T}
Yale.Myers.sub <-Yale.Myers.rn %>% select(date, month, year, julianD, tmax, tmin, tmean, prcp)
```
Look at your work:
```{r, include=T}
head(Yale.Myers.sub )
```
Create an annual summary of conditions using goup_by() and summarise() :
```{r, include=T}
Yale.Myers.annual <-Yale.Myers.sub %>% group_by(year) %>% summarise( tmax = max(tmax), tmin = min(tmin), tmean = mean(tmean), prcp = sum(prcp))
```
Look at your work:
```{r, include=T}
head(Yale.Myers.annual)
```

Determine the average monthly conditions using goup_by() and summarise() :
```{r, include=T}
Yale.Myers.monthly <-Yale.Myers.sub %>% group_by(month) %>% summarise( tmax = max(tmax), tmin = min(tmin), tmean = mean(tmean), prcp = sum(prcp)/29)
```
Look at your work:
```{r, include=T}
head(Yale.Myers.monthly)
```
Determine the average conditions using goup_by() and summarise() by both year and month:
```{r, include=T}
Yale.Myers.yearmon <-Yale.Myers.sub %>% group_by(month, year) %>% summarise( tmax = max(tmax), tmin = min(tmin), tmean = mean(tmean), prcp = sum(prcp))
```
Look at your work:
```{r, include=T}
head(Yale.Myers.yearmon )
```

Subset the mean annual temperate and year from the annual summary. Then rename tmean with "Yale-Myers" :
```{r, include=T}
Yale.Myers.annual.tmean <- Yale.Myers.annual %>% mutate(Yale.Myers = tmean) %>% select(year, Yale.Myers)
```
Look at your work:
```{r, include=T}
head(Yale.Myers.annual.tmean)
```
Save the two files, Yale.Myers and Yale.Myers.sub, in a .RDTAT object. You will need this for the post workshop assessment:
```{r, include=T}
save(Yale.Myers, Yale.Myers.sub, file="DataBasics.RDATA" )
```

# Introduction to ggplot2


## Post-Workshop Assessment:
Find a location of interest within the USA in google earth (https://earth.google.com/) and record the latitude and longitude in decimal degrees. Next, download Daymet data for that location. Join your location information with Yale.Myers.annual.tmean in a dataframe called climateCompare:
