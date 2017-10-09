---
layout: splash
title: "scraping historical data "
author: "David M soto (davidmontesoto@gmail.com)"
date: "oct 17, 2016"
description: "scraping funds historical data from financial times"
category: Sample-Posts
tags: [sample post, code, highlighting]
comments: true
share: true
---

timing is the answer
the only way to outsmart the market is with the desing of a system before start investing, if we don't desing anythig emotions, such as fear or greed will be our worst enemy, doesn't matter if your are going to invest  in CDFs, stocks, ETFs or even funds.
I have open an account in fidelity (UK), with an ISA,the first thing that they offer you is a 50 top selected funds. let's see if there is an oportunity in any of them. 

I've donwnloade this this little dataset from the fidelity webpage of the 50 selected funds, you can do it even if you are not a customer. 
fidelity to 50 :  [here](https://www.fidelity.co.uk/investor/funds/find-funds/select-list/default.page)


{% highlight r %}
rm(list = ls())
setwd("~/dataScience/blog/mfunds/")
library(data.table)
selected_fidelity = fread('./input/top50.csv')
str (selected_fidelity) 
{% endhighlight %}



{% highlight text %}
## Classes 'data.table' and 'data.frame':	88 obs. of  15 variables:
##  $ Full fund name                         : chr  "M&G European High Yield Bond I Acc" "Fidelity Special Situations Fund W-Accumulation" "Fidelity Enhanced Income Fund W-MINC-GBP" "Fidelity Global Dividend Fund W-MINC-GBP" ...
##  $ Fund ISIN                              : chr  "GB00B76MD544" "GB00B88V3X40" "GB00BYSYZP12" "GB00BYSYZL73" ...
##  $ Fund code                              : chr  "MGELA" "WSSA" "WEIIM" "WGDIM" ...
##  $ Asset Class                            : chr  "Bonds" "UK" "UK" "Global" ...
##  $ Morningstar rating                     : int  NA NA NA NA 5 5 NA NA NA NA ...
##  $ Morningstar category                   : chr  "Europe High Yield Bond" "UK Flex-Cap Equity" "UK Equity Income" "Global Equity Income" ...
##  $ OCF/TER(%)                             : num  0.92 0.94 0.94 0.95 0.96 0.96 0.95 0.69 0.19 0.19 ...
##  $ Eligible for ISA                       : chr  "Yes" "Yes" "Yes" "Yes" ...
##  $ Eligible for SIPP                      : chr  "Yes" "Yes" "Yes" "Yes" ...
##  $ Eligible for general investment account: chr  "Yes" "Yes" "Yes" "Yes" ...
##  $ Performance in % (Current Yr-4)        : num  20.4 36.3 NA NA 25.4 ...
##  $ Performance in % (Current Yr-3)        : num  7.2 10.94 NA NA 8.43 ...
##  $ Performance in % (Current Yr-2)        : num  -12.38 12.56 NA NA 8.46 ...
##  $ Performance in % (Current Yr-1)        : num  16.93 -4.11 NA NA 23.51 ...
##  $ Performance in % (Current Yr)          : num  16.5 30.6 11.3 15 15 ...
##  - attr(*, ".internal.selfref")=<externalptr>
{% endhighlight %}


## data scraping .

there are some nice packages like quuantmod that have some functions to donwload historiacal data from stocks, 
when it comes to funds, I have not found any better source of historical data like the financial times. 

it the below code we download from the Ft the last 3 years of hitorical data, basically we extract from the JSON that the FT respond our http request the XML

This process will take some minutes,  later on i will post how can we make another process for updating the file so we don't need to download all the historical data from the Financial times every day. 



{% highlight r %}
library(jsonlite) 
library(XML)
library(RCurl)
library(stringr)
library(plyr)

library(lubridate)


end.date <-gsub("-", "/", Sys.Date())
start.date <-gsub("-", "/", Sys.Date()-years(3))



url_ft_base <- paste0("https://markets.ft.com/data/equities/ajax/get-historical-prices?startDate=",
                      start.date, "&endDate=", end.date, "&symbol=" )


df <- data.frame(stringsAsFactors=FALSE) 

start.time <- Sys.time()

for (i in 1:2){
  
  url_ft = paste0(url_ft_base,selected_fidelity[i,2]) # second column is sthe isnn
  ft.xml <- getURL(url_ft) %>% fromJSON() %>%  htmlParse(asText = TRUE)  #extract from the json the xml
  xmltop = xmlRoot(ft.xml) #gives content of root
  ft.df =ldply(xmlToList(xmltop[[1]]), data.frame)   #convert the xml into a dataframe
  
  ft.df <- ft.df [-1,c(5,8,9,10,11)] # select the columns we need 
  
  names(ft.df )<-c("date","Open","High","Low","Close") # rename de columms
  ft.df [ , "fundsname"] <- selected_fidelity[i,1] # add the fundsname column from the fidelity dataset
  ft.df [ , "isin"] <-selected_fidelity[i,2]       #add the ISIN 
  ft.df [ , "AssetClass"] <-selected_fidelity[i,4]    #add the Asset Class 
  ft.df [ , "Mcategory"] <-selected_fidelity[i,6]   #add the Morning start category 
  df <-rbind( df ,ft.df )
}

str (df)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	1518 obs. of  9 variables:
##  $ date      : chr  "Fri, Oct 06, 2017" "Thu, Oct 05, 2017" "Wed, Oct 04, 2017" "Tue, Oct 03, 2017" ...
##  $ Open      : chr  "1,571.47" "1,561.99" "1,554.10" "1,553.24" ...
##  $ High      : chr  "1,571.47" "1,561.99" "1,554.10" "1,553.24" ...
##  $ Low       : chr  "1,571.47" "1,561.99" "1,554.10" "1,553.24" ...
##  $ Close     : chr  "1,571.47" "1,561.99" "1,554.10" "1,553.24" ...
##  $ fundsname : chr  "M&G European High Yield Bond I Acc" "M&G European High Yield Bond I Acc" "M&G European High Yield Bond I Acc" "M&G European High Yield Bond I Acc" ...
##  $ isin      : chr  "GB00B76MD544" "GB00B76MD544" "GB00B76MD544" "GB00B76MD544" ...
##  $ AssetClass: chr  "Bonds" "Bonds" "Bonds" "Bonds" ...
##  $ Mcategory : chr  "Europe High Yield Bond" "Europe High Yield Bond" "Europe High Yield Bond" "Europe High Yield Bond" ...
{% endhighlight %}

#time taken



{% highlight r %}
end.time <- Sys.time()
time.taken <- end.time - start.time
time.taken
{% endhighlight %}



{% highlight text %}
## Time difference of 13.27004 secs
{% endhighlight %}

#data munging
Lets format the historical data from the financial times and store it, we will use this historical data for some analysis and backtesting in other post. 


{% highlight r %}
df[,1] <- as.Date(df[,1], format="%a, %b %d, %Y") # date format
converToNumeric = function(x) { as.numeric(gsub(",", "", x, fixed = TRUE)) }
df[,2:5] = sapply(df[,2:5], converToNumeric) # numeric format
str (df)
{% endhighlight %}



{% highlight text %}
## 'data.frame':	1518 obs. of  9 variables:
##  $ date      : Date, format: "2017-10-06" "2017-10-05" ...
##  $ Open      : num  1571 1562 1554 1553 1545 ...
##  $ High      : num  1571 1562 1554 1553 1545 ...
##  $ Low       : num  1571 1562 1554 1553 1545 ...
##  $ Close     : num  1571 1562 1554 1553 1545 ...
##  $ fundsname : chr  "M&G European High Yield Bond I Acc" "M&G European High Yield Bond I Acc" "M&G European High Yield Bond I Acc" "M&G European High Yield Bond I Acc" ...
##  $ isin      : chr  "GB00B76MD544" "GB00B76MD544" "GB00B76MD544" "GB00B76MD544" ...
##  $ AssetClass: chr  "Bonds" "Bonds" "Bonds" "Bonds" ...
##  $ Mcategory : chr  "Europe High Yield Bond" "Europe High Yield Bond" "Europe High Yield Bond" "Europe High Yield Bond" ...
{% endhighlight %}



{% highlight r %}
saveRDS(df, "~/dataScience/blog/mfunds/input/selected2.rds")
{% endhighlight %}

