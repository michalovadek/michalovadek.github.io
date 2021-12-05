---
title: Kernel density estimation for time series plots
author: Michal Ovadek
date: '2019-12-07'
slug: time-series-data-and-kernel-density
categories:
  - Data
tags:
  - time series
  - rstats
  - kernel density
  - viz
image:
  caption: ''
  focal_point: ''
publication_type: article
---

In my research I frequently encounter count data that has a temporal dimension. The standard way of visualizing such data is to take the counts grouped at some temporal level (month, year, etc). The clustering essentially does away with the problem of the y-axis, namely in that it creates variation at evenly spaced out x-axis intervals. Let's generate randomly a bunch of dates and show an archetypal bar plot at the year level of, say, judgments.


```r
library(tidyverse)
library(lubridate)

set.seed(92139)

dates <- as_date(as.Date("2000-01-01"):as.Date("2020-01-01"))

dat <- sample(dates, 300, replace = F,
              prob = abs(rnorm(length(dates),100, 200))
              )

dat %>% 
  enframe() %>% 
  mutate(year = as.integer(str_sub(value,1,4))) %>% 
  group_by(year) %>% 
  count() %>% 
  ggplot(aes(x = year, y = n)) +
  geom_col()
```

<div class="figure">
<img src="/post/2019-12-07-time-series-data-and-kernel-density/2019-12-07-time-series-data-and-kernel-density_files/figure-html/barplot-1.png" alt="Number of fictitious judgments between 2000-2020" width="672" />
<p class="caption">Figure 1: Number of fictitious judgments between 2000-2020</p>
</div>

Nothing very special here. However, note that instead of using the actual _dates_ of the judgments, we sacrifice precision to obtain variation on the y-axis. Often the grouping (temporal) level -- year in our case -- does not carry any particular meaning beyond the fact that years are readily comprehensible to everyone. Can we do better?

Let's go back to the same data and visualize the date of each judgment. Without any additional variable that could supply variation along a second dimension, I simply fix _y = 1_ to denote that each judgment is counted, well, once.


```r
dat %>% 
  enframe() %>% 
  ggplot(aes(x = value, y = 1)) +
  geom_point(alpha = 0.1) +
  theme_bw()
```

<div class="figure">
<img src="/post/2019-12-07-time-series-data-and-kernel-density/2019-12-07-time-series-data-and-kernel-density_files/figure-html/dots-1.png" alt="Dates of fictitious judgments between 2000-2020" width="672" />
<p class="caption">Figure 2: Dates of fictitious judgments between 2000-2020</p>
</div>

The visualization is not very good. Although the x-axis is now a proper continuous timeseries measured in days, there are too many judgments and days in the 20-year period to see clearly any patterns in the data. Increasing the transparency of the data points is a good first step, as darker areas indicate greater *density* of judgments around that time.

We can exploit the full potential of the dates using the statistical concept of _kernel density_. For ease of interpretation, we can think of this as a smoothed histogram. It is very straightforward to implement in ``ggplot``.


```r
dat %>% 
  enframe() %>% 
  ggplot(aes(x = value)) +
  geom_vline(xintercept = as.Date("2013-01-01"), lty = 2) +
  annotate("text", x = as.Date("2014-12-31"), y = 0.99, label = "Some event", fontface = "bold") +
  geom_histogram(alpha = 0.1, aes(y = ..ncount..), bins = 40) +
  stat_density(geom = "line", adjust = 1/10, kernel = "gaussian", aes(y = ..scaled..)) +
  geom_point(alpha = 0.1, y = 0.1) +
  theme_bw() +
  theme(panel.grid = element_blank()) +
  labs(x = NULL, y = "Kernel density estimates")
```

<div class="figure">
<img src="/post/2019-12-07-time-series-data-and-kernel-density/2019-12-07-time-series-data-and-kernel-density_files/figure-html/kde-1.png" alt="Dates of fictitious judgments between 2000-2020 with density estimated" width="672" />
<p class="caption">Figure 3: Dates of fictitious judgments between 2000-2020 with density estimated</p>
</div>

Using ``stat_density`` we get a nice continuous (estimated) timeseries of our data. The idea is to basically treat the time dimension as just a series of numbers over which a continuous distribution of our data can be drawn.

Setting ``y = ..scaled..`` scales density to the 0-1 range, which is useful if we want to, for example, compare the kernel density estimates with a histogram that we similarly scale with ``y = ..ncount..``. Changing ``adjust`` is the simplest way of controlling the smoothing parameter: the lower the value, the less smooth the curve (picks up on more variation). We can similarly adjust the histogram by increasing (decreasing) the number of ``bins``. There is a variety of methods to calculate kernel density and this is controlled through the ``kernel`` parameter ("gaussian" by default).

Although caution is required -- we are after all estimating a smoothed curve from discrete data points -- I have found kernel density estimation to be a valuable tool for exploring temporal patterns. In most cases it is decisively superior to a bar chart with arbitrarily grouped counts on the y-axis. The below is an example from my PhD thesis where I compare a theoretical model of time variation with a kernel density estimate drawn from submission dates of court cases.

![ ](temporal.png)


