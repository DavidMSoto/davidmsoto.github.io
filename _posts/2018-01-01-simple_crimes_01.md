---
layout: splash
title: "maps of sin "
author: "David M soto (davidmontesoto@gmail.com)"
date: "oct 17, 2016"
description: "london experiment."
category: Sample-Posts
tags: [sample post, code, highlighting]
comments: true
share: true

header:
  image: /assets/images/truedetective.jpg
---


## crime figures revealed (metropolitan police)

provided by:  [here](https://data.police.uk/data/)



{% highlight r %}
rm(list = ls())

library(data.table)
library(ggplot2) # graph
library(plotly)  # grapgh
library(tidyr)   #separate
library(dplyr)#select



readData <- function(path.name, file.name,  missing.types, types) {
  fread(  paste(path.name, file.name, sep="") , 
            colClasses=types,
            na.strings=missing.types, 
          verbose = FALSE, showProgress = FALSE )
}

types <- c(
'Crime ID'             ='character',
'Month'                ='character', 
'Reported by'          ='NULL',
'Falls within'         ='NULL',
'Longitude'            ='integer',   
'Latitude'             ='integer',    
'Location'             ='character',   
'LSOA code'            ='character',
'LSOA name'            ='character',
'Crime type'           ='character',
'Last outcome category'='character',
'Context'              ='NULL'

)


missing.types <- c("NA", "")

crime.raw <- readData('~/dataScience/blog/crimes/input/', "metropolitan_2016.csv",   missing.types,types  )

names(crime.raw) <- c( "CrimeID",  "Month",  "Longitude",  "Latitude", "Location",    "LSOAcode",   "LSOAname",  "Crimetype", "Lastoutcomecategory")
{% endhighlight %}
    





## google maps 2 


{% highlight r %}
library(htmlwidgets)
library(leaflet)

sDrugs <- crime.raw.clean  %>% filter( Crimetype == 'Burglary')
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'crime.raw.clean' not found
{% endhighlight %}



{% highlight r %}
d = sDrugs %>%     mutate(lon = Longitude, lat = Latitude)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'sDrugs' not found
{% endhighlight %}



{% highlight r %}
d$lon <- as.numeric (d$lon)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd' not found
{% endhighlight %}



{% highlight r %}
d$lat <- as.numeric (d$lat)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd' not found
{% endhighlight %}



{% highlight r %}
d$Longitude <-  as.numeric (d$Longitude)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd' not found
{% endhighlight %}



{% highlight r %}
d$Latitude <- as.numeric (d$Latitude)
{% endhighlight %}



{% highlight text %}
## Error in eval(expr, envir, enclos): object 'd' not found
{% endhighlight %}



{% highlight r %}
showmap = function(dataset) {
    library(leaflet)
    
    m <- leaflet() %>%
      addTiles() %>%  # Add default OpenStreetMap map tiles
      addMarkers(lng=as.integer(dataset$Longitude), lat=as.integer(dataset$Latitude),
                 popup=dataset$Crimetype,
                 clusterOptions = markerClusterOptions())
    m  # Print the map
  }

  showmap(d)
{% endhighlight %}



{% highlight text %}
## Error in inherits(f, "formula"): object 'd' not found
{% endhighlight %}
