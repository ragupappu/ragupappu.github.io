---
layout: post
title: "Interactive Dublin (California) Weather Visualization Using Shiny"
author: "RP"
date: "07/05/2015"
output:
  html_document:
    keep_md: true
comments: True
tags: [shiny, r, data visualization]
---

A few weeks back I wrote a [post](/2015/03/12/dublintemp/) in which I visualized Dublin (CA) 2014 weather in Edward Tufte's style of New York 2003 weather visualization. That post used a dataset containing 15 years of daily weather data from 2000 to 2014 and showed daily temperature highs and lows against a background of daily _record_ highs and lows and _average_ highs and lows of years before 2014. I wanted to write an interactive application using [Shiny](http://shiny.rstudio.com/), RStudio's web application framework for R, and the Dublin weather visualization seemed like a great candidate. I wrote the Shiny app and chart you see below is an interactive version of that visualization. The user can select the range of years of temperature data to form the background, anywhere between years 2000 and 2014 inclusive.
<iframe src="https://ragupappu.shinyapps.io/shiny-dublin-temps/" style="border: none; width: 100%; height: 700px"></iframe>

The above app is hosted on [my account](http://ragupappu.shinyapps.io/shiny-dublin-temps) at [shinyapps.io](https://www.shinyapps.io/). The code for it is located [here](https://github.com/ragupappu/shiny-dublin-temps).

##Code Changes
A Shiny app requires a minimum of two code modules named ui.R and server.R. I got started writing the app by copying those two modules from lesson 2 of the RStudio [Shiny tutorial](http://shiny.rstudio.com/tutorial/) to my local folder and then making the necessary changes.

I added a `sliderInput` user-interface to allow the user to select the start\_year to bracket data upto and including 2014. By default the weather visualization displays the daily _record_ highs and lows and also the daily _average_ highs and lows in the background. I added two `checkboxInput`s to allow the user to hide those background displays. The code for the original visualization post used a file, viz\_temps.R, which I re-used for this post but with several changes to allow it to work within the Shiny framework. I converted the R script in that file into a function called viztemps() which is invoked from server.R.

The code changes in ui.R and server.R were simple and straightforward as you can see below. Other code modules are not shown (they are located [here](https://github.com/ragupappu/shiny-dublin-temps)).

###ui.R
```{r shiny_ui, echo=TRUE}
library(shiny)

# Define UI for application
shinyUI(fluidPage(

  # Application title
  titlePanel("Dublin (California) 2014 Temperatures"),

  # Show a plot of the generated distribution
  plotOutput("DublinTemps"),

  hr(),

  # Sidebar with a slider input for the number of bins
  fluidRow(
    column(5,
      h4("Data Range (Start Year to 2014)"),
      sliderInput('start_year',
                  'Select StartYear',
                  min = 2000,
                  max = 2014,
                  value = 2004),
      offset=1
    ),
    column(5,
      checkboxInput('hide_hilows', 'Hide background daily record highs and lows bars'),
      checkboxInput('hide_avgs', 'Hide background average daily highs and lows bars'),
      offset=1
    )
  )
))
```

###server.R
```{r shiny_server, echo=TRUE}
library(shiny)

# Preprocessing and summarizing data
library(dplyr)

# Visualization development
library(ggplot2)

# For text graphical objects (to add text annotation)
library(grid)

source("viz_temps.R")

# Define server logic required to draw a histogram
shinyServer(function(input, output) {
  
  output$DublinTemps <- renderPlot({
    viztemps(input$start_year,
             input$hide_hilows,
             input$hide_avgs
    )
  })
})
```

In order to get it to run on this webpage I simply embedded within this post's markdown file an HTML iframe tag using the code below

```
<iframe src="https://ragupappu.shinyapps.io/shiny-dublin-temps/" style="border: none; width: 100%; height: 700px"></iframe>
```

_***NOTE***: Sometimes it can take up to a minute to draw the chart when the page first loads. But it can also take that much time to re-draw when the user changes slider settings or selects/unselects checkboxes. At the time of posting this article I know of no way to speed up the drawing. If you know how to do it, please post in the comments section._
