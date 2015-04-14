---
layout: post
title: "Dublin (California) 2014 Weather Using R"
author: "RP"
date: "2015/03/12"
output:
  html_document:
    keep_md: true
comments: True
---

In the data visualization community Edward Tufte's chart of [New York City 2003 weather](http://www.edwardtufte.com/bboard/q-and-a-fetch-msg?msg_id=00014g) is well-known. A few weeks ago Brad Boehmke published a blogpost with a similar chart for his city, Dayton, titled [Dayton's weather in 2014](http://rpubs.com/bradleyboehmke/weather_graphic) which inspired me to do a similar visualization for my city, Dublin, California. The result is the chart below. The R code to build the chart is located [here](https://github.com/rpappu0206/tufte/tree/master/src) and it draws from Boehmke's post but also includes original code. Further below I describe the steps used to arrive at the chart.
![Dublin 2014 Weather testing]({{ site.url }}assets/images/dublintemps/dublin2014temps.svg)

I started by searching the [NCDC](http://ncdc.noaa.gov/) (National Climatic Data Center) website for Dublin weather data but when the results came up empty I widened the search to neighboring towns. Luckily, data was available for Livermore Municipal Airport, located less than 10 miles from Dublin, which, for all practical purposes, could serve as a substitute for Dublin weather data. I obtained that [data](https://github.com/rpappu0206/tufte/tree/master/data) from NCDC for the most recent 15-year period (January 01, 2000 through December 31, 2014) and used it as the raw data for this project. This data is hereafter referred to as Dublin weather data.

Unlike Tufte's original chart (shown below) which included graphics for Temperature and Precipitation, I decided to include only Temperature in my chart. 

![Edward Tufte's New York City Weather 2003 Chart]({{ site.url }}assets/images/dublintemps/New-York-City-Weather-Chart-2003_3D887A31.jpg)

The raw data contains daily high and low temperatures but also several other 'main' variables such as snowfall, snow depth, precipitation, type of weather, etc. Associated with each main variable are four variables - Measurement Flag, Quality Flag, Source Flag, and Time of Measurement. The [codebook](https://github.com/rpappu0206/tufte/tree/master/doc) for the raw data, GHCND_documentation.pdf, explains these variables in detail. An excerpt of select columns from the raw data CSV file is shown below. The temperature data are in tenths of degrees Celsius.




{% highlight text %}
            station     date tmax quality.flag.4 source.flag.4
1 GHCND:USW00023285 20000101  117             NA             W
2 GHCND:USW00023285 20000102  133             NA             W
3 GHCND:USW00023285 20000103  133             NA             W
4 GHCND:USW00023285 20000104  144             NA             W
5 GHCND:USW00023285 20000105  161             NA             W
6 GHCND:USW00023285 20000106  144             NA             W
{% endhighlight %}

I processed the data in two stages: Stage1 - load and extract relevant data, and Stage2 - clean the data and chart it. Accordingly, the two source files are named load.R and viz_temps.R.

In Stage1 I processed the raw data as follows: extract relevant columns of data into a dataframe, split Dates into Year, Month and Day data, and finally, save the dataframe to file 'livermore\_15yr\_temps.csv' for use as input to Stage2. The relevant columns of data are Dates, and temperature columns Tmax and Tmin and 'qualifier' data columns associated with each, titled Measurement Flag, Quality Flag and Source Flag.


{% highlight r %}
# Preprocessing and summarizing data
library(dplyr)

# Load the NCDC 15-year (2000 Jan - 2014 Dec) Livermore (California) Airport daily weather
weather_15yr_data <- read.csv("ncdc_livermore_15yr_weather.csv", stringsAsFactors=FALSE, sep=",")

# convert all column names into lowercase (for convenience)
colnames(weather_15yr_data) <- tolower(names(weather_15yr_data))

# Create a dataframe containing data related only to temperature measurements; ignore other data
all_temps_15yrs <- weather_15yr_data %>%
        select(date,
               tmax, measurement.flag.4, quality.flag.4, source.flag.4,
               tmin, measurement.flag.5, quality.flag.5, source.flag.5)

# Convert date into three columns - year, month, day
# The date in the raw data file is a string in the format YYYYMMDD

# First convert String object to class "Date"
all_dates <- as.Date(as.character(all_temps_15yrs$date), format="%Y%m%d", origin="1970-01-01")

# Extract parts of the date and make three columns
all_dates <- as.POSIXlt(all_dates)      # POSIXlt object is a list of date parts
year <- all_dates$year + 1900           # years is num of years from 1900, therefore adding 1900
month <- all_dates$mon
day <- all_dates$mday

# Replace the date column with 'split' date part columns (i.e., year, month, day columns)
all_temps_15yrs <- subset(all_temps_15yrs, select=-date)    # drop date column
all_temps_15yrs <- cbind(year, month, day, all_temps_15yrs) # Add back the 'split' date column

# Save the data into a file. Data in this file will be used for visualization
write.csv(all_temps_15yrs, file="livermore_15yr_temps.csv", row.names=FALSE)
{% endhighlight %}

In Stage2 I clean the data first and then process it. I start by including these packages:


{% highlight r %}
# Preprocessing and summarizing data
library(dplyr)

# Visualization development
library(ggplot2)

# For text graphical object (to add text annotation)
library(grid)
{% endhighlight %}

After reading livermore\_15yr\_temps.csv (the file created in Stage1) into a dataframe, as a first step I prepared the data for analysis by cleaning up the datais to clean the data, that is, drop invalid records: those with Tmax and Tmin values of -9999, and those with invalid entries in any of the three flag columns - Measurement, Quality and Source. Next I converted the temperature data from Celcius to Fahrenheit.


{% highlight r %}
# Load 15 years temperature data
all_temps_15yrs <- read.csv("livermore_15yr_temps.csv", stringsAsFactors=FALSE, sep=",")

# Clean the data: drop records with invalid temp values, and missing or invalid
#  measurement, quality or source flags
all_temps_15yrs <- all_temps_15yrs %>%
        filter(tmax != -9999                &   # drop missing max temp identified by -9999
               tmin != -9999                &   # drop missing min temp identified by -9999
               is.na(measurement.flag.4)    &   # keep tmax data with no special measurement info
               is.na(measurement.flag.5)    &   # keep tmin data with no special measurement info
               is.na(quality.flag.4)        &   # keep tmax data that did not fail quality check
               quality.flag.5 == " "        &   # keep tmin data that did not fail quality check
               source.flag.4 != " "         &   # drop tmax data with no source (blank)
               !is.na(source.flag.4)        &   #  or NA                              
               source.flag.5 != " "         &   # drop tmin data with no source (blank)
               !is.na(source.flag.5))           #  or NA

# Raw data temps are in Celsius degrees to tenths. Convert to Fahrenheit and scale for tenths
all_temps_15yrs <- all_temps_15yrs %>%
        mutate(tmaxF = tmax*0.18 + 32,          # convert to Fahrenheit (F = C*9/5 + 32)
               tminF = tmin*0.18 + 32)          # note: temps are also being converted from tenths
                                                #       to real (normal) values
{% endhighlight %}

There were only 2 invalid records in the 15 year dataset, or 0.036503% of the observations, which were dropped. Next I computed the average max and average min temperature for each year to see how Dublin fared in 2014 relative to the previous 14 years. 


{% highlight r %}
# Compute the average per year min and max temps
avg_temps_each_year <- all_temps_15yrs %>%
        group_by(year) %>%
        summarise(avg_min = mean(tminF),
                  avg_max = mean(tmaxF)) %>%
        ungroup()
{% endhighlight %}

Sorting the average yearly temperatures gives an interesting result: 2014 was the warmest of the past 15 years! In a [report](http://www.nasa.gov/press/2015/january/nasa-determines-2014-warmest-year-in-modern-record/) in January NASA and NOAA found that 2014 was the warmest year in modern record, and the above results for Dublin temperatures align with that finding.


{% highlight text %}
## Source: local data frame [15 x 2]
## 
##    year  avg_max
## 1  2014 76.96252
## 2  2013 74.76997
## 3  2012 73.74869
## 4  2008 73.25246
## 5  2001 73.15885
## 6  2007 72.73918
## 7  2009 72.67901
## 8  2002 72.47288
## 9  2006 72.40456
## 10 2003 72.40334
## 11 2005 72.10449
## 12 2004 72.04016
## 13 2000 71.72970
## 14 2011 71.44416
## 15 2010 70.91353
{% endhighlight %}



{% highlight text %}
## Source: local data frame [15 x 2]
## 
##    year  avg_min
## 1  2014 50.83096
## 2  2012 47.54246
## 3  2005 47.49578
## 4  2013 47.40110
## 5  2006 47.32077
## 6  2009 47.14416
## 7  2001 47.13036
## 8  2000 47.09041
## 9  2010 47.08499
## 10 2004 46.94934
## 11 2008 46.90803
## 12 2003 46.87786
## 13 2007 46.48926
## 14 2011 46.24860
## 15 2002 45.86690
{% endhighlight %}

With the preliminaries completed, I created two key dataframes, a `Past` dataframe containing 14-year data from 2000 to 2013, and a `Present` dataframe containing 2014 data. Using the `Past` dataframe I computed two additional metrics: (i) highest and lowest daily temperature over the past 14 years, and (ii) average daily maximum and average daily minimum temperatures. The former will form the wheatish-colored broad background temperature swath, as in Tufte's image, that will show the record daily temperatures over a 14 year period. The latter will form the light-brown swath that represents the _normal_ daily temperature range over the same 14-year period. I created a `newDay` variable for use as the x-axis variable for days 1 through 365 of the year.


{% highlight r %}
# create a dataframe that represents 14 years of historical temp data from 2000-2013
Past <- all_temps_15yrs %>%
        group_by(year, month) %>%
        arrange(day) %>%
        ungroup() %>%
        group_by(year) %>%
        mutate(newDay = seq(1, length(day))) %>%    # label days as 1:365 (will represent x-axis)
        ungroup() %>%
        filter(year != 2014) %>%                # filter out 2014 data
        group_by(newDay) %>%
        mutate(upper = max(tmaxF),              # identify same day highest max temp from all years
               lower = min(tminF),              # identify same day lowest min temp from all years
               avg_upper = mean(tmaxF),         # compute same day average max temp from all years
               avg_lower = mean(tminF)) %>%     # compute same day average min temp from all years
        ungroup()

# create a dataframe that represents 2014 temperature data
Present <- all_temps_15yrs %>%
        group_by(year, month) %>%
        arrange(day) %>%
        ungroup() %>%
        group_by(year) %>%
        mutate(newDay = seq(1, length(day))) %>%    # label days as 1:365 (will represent x-axis)
        ungroup() %>%
        filter(year == 2014)                    # filter out all years except 2014 data
{% endhighlight %}

As we saw earlier, 2014 turns out to be the warmest year, and to further explore if any records were set in the year, I created four additional dataframes called _PastLows_, _PastHighs_, _PresentLows_, and _PresentHighs_. The first two contain the record lows and highs of the past 4 years, and the second two contain the record highs and lows in 2014 relative to the full 15-year period.


{% highlight r %}
# create dataframe that represents the lowest same-day temperature from years 2000-2013
PastLows <- Past %>%
        group_by(newDay) %>%
        summarise(Pastlow = min(tminF)) # identify lowest same-day temp between 2000 and 2013

# create dataframe that represents the highesit same-day temperature from years 2000-2013
PastHighs <- Past %>%
        group_by(newDay) %>%
        summarise(Pasthigh = max(tmaxF)) # identify highest same-day temps between 2000 and 2013

# create dataframe that identifies days in 2014 when temps were lower than in all previous 14 years
PresentLows <- Present %>%
        left_join(PastLows) %>%         # merge historical lows to 2014 low temp data
        mutate(record = ifelse(tminF<Pastlow, "Y", "N")) %>% # current year was a record low?
        filter(record == "Y")           # filter for 2014 record low days

# create dataframe that identifies days in 2014 when temps were higher than in all previous 14 years
PresentHighs <- Present %>%
        left_join(PastHighs) %>%        # merge historical lows to 2014 low temp data
        mutate(record = ifelse(tmaxF>Pasthigh, "Y", "N")) %>% # current year was a record high?
        filter(record == "Y")           # filter for 2014 record high days
{% endhighlight %}

The data necessary to plot the chart is available at this point. We already have the x-axis variable but the y-axis variable needs to be created. Also, the y-axis values must show the degree symbol.


{% highlight r %}
# function: Turn y-axis labels into values with a degree superscript
degree_format <- function(x, ...) {
  parse(text = paste(x, "*degree", sep=""))
}

# create y-axis variable
yaxis_temps <- degree_format(seq(0, 120, by=10))
{% endhighlight %}

The stage is now set to actually create the chart using ggplot2. The chart is created in a series of steps adding layers at each step. Since I followed the steps in Boehmke's post I will not go into details except for short descriptions.

__Step 1__: Create the canvas for the plot and show the 14-year record highs and lows as the broad background


{% highlight r %}
# Step 1: create the canvas for the plot. Also plot the background lowest, highest 2000-2013 temps
p <- ggplot(Past, aes(newDay, tmaxF)) +
        theme(plot.background = element_blank(),
              panel.grid.minor = element_blank(),
              panel.grid.major = element_blank(),
              panel.border = element_blank(),
              panel.background = element_rect(fill = "seashell2"),
              axis.ticks = element_blank(),
              #axis.text = element_blank(),  
              axis.title = element_blank()) +
        geom_linerange(Past,
                       mapping=aes(x=newDay, ymin=lower, ymax=upper),
                       size=0.8, colour = "#CAA586", alpha=.6)
print(p)
{% endhighlight %}

![center](/../figs/step1-1.png) 

__Step 2__: Add the average daily temperatures over the 14-year period, 2000-2013.


{% highlight r %}
# Step 2: Plot average low and high temps from 2000-2013
p <- p +
        geom_linerange(Past,
                       mapping=aes(x=newDay, ymin=avg_lower, ymax=avg_upper),
                       size=0.8,
                       colour = "#A57E69")
print(p)
{% endhighlight %}

![center](/../figs/step2-1.png) 

__Step 3__: Finally, add the 2014 temperatures.


{% highlight r %}
# Step 3: Plot 2014 high and low temps
p <- p +
        geom_linerange(Present, mapping=aes(x=newDay, ymin=tminF, ymax=tmaxF), size=0.8, colour = "#4A2123")
print(p)
{% endhighlight %}

![center](/../figs/step3-1.png) 

__Step 4__: Add the y-axis border and the x-axis gridlines


{% highlight r %}
# Step 4: Add the y-axis border and the x-axis gridlines
p <- p +
        geom_vline(xintercept = 0, colour = "wheat4", linetype=1, size=1) +
        geom_hline(yintercept = 0, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 10, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 20, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 30, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 40, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 50, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 60, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 70, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 80, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 90, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 100, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 110, colour = "ivory2", linetype=1, size=.1) +
        geom_hline(yintercept = 120, colour = "ivory2", linetype=1, size=.1)
print(p)
{% endhighlight %}

![center](/../figs/step4-1.png) 

__Steps 5 and 6__: Add vertical gridlines to mark each month and add labels to the x and y axes.


{% highlight r %}
# Step 5: Add vertical gridlines to mark end of each month
p <- p +
        geom_vline(xintercept = 31, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 59, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 90, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 120, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 151, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 181, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 212, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 243, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 273, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 304, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 334, colour = "wheat4", linetype=3, size=.4) +
        geom_vline(xintercept = 365, colour = "wheat4", linetype=3, size=.4)

# Step 6: Add labels to the x and y axes
p <- p +
        coord_cartesian(ylim = c(0,120)) +
        scale_y_continuous(breaks = seq(0,120, by=10), labels = yaxis_temps) +
        scale_x_continuous(expand = c(0, 0),
                           breaks = c(15,45,75,105,135,165,195,228,258,288,320,350),
                           labels = c("January", "February", "March", "April",
                                      "May", "June", "July", "August", "September",
                                      "October", "November", "December"))
print(p)
{% endhighlight %}

![center](/../figs/steps5n6-1.png) 

__Step 7__: Mark days with record temperatures. Note that there are no record low temperature points in the chart below. That's because there really were no record lows in 2014 because it was the warmest year.


{% highlight r %}
# Step 7: Add points to mark the 2014 record high and low temps
p <- p +
        geom_point(data=PresentLows, aes(x=newDay, y=tminF), colour="blue3") +
        geom_point(data=PresentHighs, aes(x=newDay, y=tmaxF), colour="firebrick3")
print(p)
{% endhighlight %}

![center](/../figs/step7-1.png) 

__Steps 8 and 9__: Add a title to the chart and provide explanation about the data in the top left


{% highlight r %}
# Step 8: Add title to plot
p <- p +
        ggtitle("Dublin (California) Weather in 2014") +
        theme(plot.title=element_text(face="bold",hjust=.012,vjust=.8,colour="gray30",size=18))

# Step 9: Add explanation text under the plot title

grob1 = grobTree(textGrob("Temperature\n",
                          x=0.02, y=0.92, hjust=0,
                          gp=gpar(col="gray30", fontsize=10, fontface="bold")))
p <- p + annotation_custom(grob1)

grob2 = grobTree(textGrob(paste("Bars represent range between daily high and low temperatures.\n",
                                "Data set includes data from Jan 1, 2000 to December 31, 2014.\n",
                                "Average high temperature for 2014 was 76.9F making it\n",
                                "the warmest in 15 years since 2000.", sep=""),
                          x=0.02, y=0.83, hjust=0,
                          gp=gpar(col="gray30", fontsize=8.5)))

p <- p + annotation_custom(grob2)
print(p)
{% endhighlight %}

![center](/../figs/steps8n9-1.png) 

__Step 10__: Add annotation for the record high 2014 temperature points.


{% highlight r %}
# Step 10: Add annotation for points representing the record high 2014 temperatures
#          Note: There were no record lows in 2014, it was the warmest year of the 15-yr period!

grob3 = grobTree(textGrob(paste("In 2014 there were 60 days that were\n",
                                 "hottest since 2000\n",sep=""),
                          x=0.72, y=0.9, hjust=0,
                          gp=gpar(col="firebrick3", fontsize=7)))

p <- p + annotation_custom(grob3)

p <- p +
        annotate("segment", x = 257, xend = 263, y = 99, yend = 108, colour = "firebrick3")
print(p)
{% endhighlight %}

![center](/../figs/step10-1.png) 

__Step 11__: Finally, add legend to explain the three "layers" of data: the background record highs and lows over a 14-year period, the average highs and lows over the same period, and the 2014 daily highs and lows.


{% highlight r %}
# Step 11: Add legend to explain difference between the different data point layers

p <- p +
        annotate("segment", x = 181, xend = 181, y = 5, yend = 25, colour = "#CAA586", size=3) +
        annotate("segment", x = 181, xend = 181, y = 11, yend = 19, colour = "#A57E69", size=3) +
        annotate("segment", x = 181, xend = 181, y = 13, yend = 22, colour = "#4A2123", size=2) +
        annotate("segment", x = 177, xend = 179, y = 18.7, yend = 18.7, colour = "#A57E69", size=.5) +
        annotate("segment", x = 177, xend = 179, y = 11.2, yend = 11.2, colour = "#A57E69", size=.5) +
        annotate("segment", x = 177, xend = 177, y = 11.2, yend = 18.7, colour = "#A57E69", size=.5) +
        annotate("segment", x = 183, xend = 185, y = 13.25, yend = 13.25, colour = "#4A2123", size=.3) +
        annotate("segment", x = 183, xend = 185, y = 21.75, yend = 21.75, colour = "#4A2123", size=.3) +
        annotate("text", x = 165, y = 14.75, label = "NORMAL RANGE", size=2.1, colour="gray30") +
        annotate("text", x = 170, y = 25, label = "RECORD HIGH", size=2.1, colour="gray30") +
        annotate("text", x = 170, y = 5, label = "RECORD LOW", size=2.1, colour="gray30") +
        annotate("text", x = 195, y = 21.75, label = "ACTUAL HIGH", size=2.1, colour="gray30") +
        annotate("text", x = 195, y = 13.25, label = "ACTUAL LOW", size=2.1, colour="gray30")

print(p)
{% endhighlight %}

![center](/../figs/step11-1.png) 

Thanks to Boehmke's detailed post, I was able to create a chart of Dublin 2014 weather very much like Edward Tufte's New York 2003 weather chart.








