---
title: "Data Visualization"
date: 2019-03-29
tags: [data visualization, data science, R]
header:
  image: "/images/perceptron/percept.jpg"
excerpt: "Data Visualization, Data Sciene, R"
mathjax: "true"
---

# NBA Shot Chart

I have used the data from NBA players to create a shot chart graph. I have utilized R for this project to produce the images.

An example of the shot chart I created is this: <img src="{{ site.url }}{{ site.baseurl }}/images/klay-thompson-shot-chart.pdf" alt="klay data">

For more images you can also click [here](https://github.com/beverlylt/workout01/tree/master/images).


This is an excerpt of the code I have used specifically for Klay Thompson's shot chart.
```r
library(ggplot2)

data <- read.csv("../data/shots-data.csv", stringsAsFactors = FALSE)
klay <- data[data[,'name'] == 'Klay Thompson', ]

klay_scatterplot <- ggplot(data = klay) + geom_point(aes(x = x, y = y, color = shot_made_flag))

court_file <- "../images/nba-court.jpg"

court_image <- rasterGrob( readJPEG(court_file), width = unit(1, "npc"), height = unit(1, "npc"))

klay_shot_chart <- ggplot(data = klay) + annotation_custom(court_image, -250, 250, -50, 420) + geom_point(aes(x = x, y = y, color = shot_made_flag)) + ylim(-50, 420) +
  ggtitle('Shot Chart: Klay Thompson (2016 season)') + theme_minimal()

pdf(file = '../images/klay-thompson-shot-chart.pdf', width = 6.5, height = 5)
klay_shot_chart
dev.off()
```

In short, I have used the coordinates given by the data to create a dot plot, and laid it over an image of the NBA court. `klay_shot_chart` then created the graph, and I exported it as an image pdf.
