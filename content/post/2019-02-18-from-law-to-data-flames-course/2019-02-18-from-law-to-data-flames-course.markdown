---
title: From Law to Data (FLAMES course)
author: Michal Ovadek
date: '2019-02-18'
slug: from-law-to-data-flames-course
categories:
  - Teaching
tags:
  - teaching
  - law
  - data
  - rstats
image:
  caption: ''
  focal_point: ''
publication_type: article
---

In February 2019 I organized and taught a 12-hour course introducing legal researchers to data analysis. This initiative was born out of the recognition that lawyers' data skills lagged behind most other disciplines.

The class was offered through the [FLAMES network](https://www.flames-statistics.com/courses-seminars/from-law-to-data-a-gentle-introduction-to-data-based-analysis-in-law/) which supports methodological training for young researchers at Flemish universities. It proved considerably popular: course registrations reached room capacity (over 50 students) with several more on the waiting list.

The course was a first of a kind in Belgium in that it tailored introduction to data analysis to legal research. We faced a number of obstacles in designing and teaching the course:
 
  - our target audience did not have a basic scientific vocabulary (e.g. variables/observations)
  - no/minimal previous methodological or statistical training
  - little interest in empirical research
  - no experience with programming
  - a dearth of continuous variables
  
The interest in the course demonstrated, however, that a good portion of young legal researchers in Belgium was at least wary of their limited analytical skillset and willing to learn. We were heartened to see the enthusiasm of many students in the class.

The course was split in four 3-hour classes which covered roughly the following topics:
  
  1. Rectangular spreadsheets and manually coding information in Excel
  2. First steps in R and importing Excel spreadsheet into R
  3. Data wrangling and visualization
  4. Reading PDF files, document-term matrices and wordclouds

We tried combining topics of general data-related importance, such as rectangular spreadsheets and summary statistics, with tools more specifically relevant to lawyers, such as processing text-data from PDF files.

In the Excel part we drew heavily on a fantastically useful paper by [Broman and Woo (2018)](https://www.tandfonline.com/doi/abs/10.1080/00031305.2017.1375989) which catalogues best data organization practices for spreadsheets. The content of this paper was perfectly suited to open our gentle introduction to data analysis.

Unlike most R courses we focused in particular on working with binary, categorical and character data, at some cost to continuous and numerical data. This was a conscious choice made with the objective of bringing the course as close as possible to how our target group conducted research.

We made `tidyverse` functions a core part of the course, although we also introduced `base` alternatives. This resulted, among others, in an interesting split between students who quickly took to piping (`%>%`) and others who preferred the standard method. The full list of packages used during the course:


```r
library(tidyverse)
library(readxl)
library(writexl)
library(corrplot)
library(stargazer)
library(quanteda)
```

Plotting focused on trends over time, with data filtered, grouped and summarized using the `dplyr` toolkit.




```r
decisions %>%
  group_by(year) %>%
  mutate(avg_length = mean(length)) %>%
  ggplot(aes(x=year,y=avg_length,color=n_judges)) + geom_line() + theme_classic()
```

<img src="/post/2019-02-18-from-law-to-data-flames-course/2019-02-18-from-law-to-data-flames-course_files/figure-html/unnamed-chunk-3-1.png" width="672" />

The course received overall good feedback from the participants. The average rating of the course was 7.9 out of 10. The written feedback brought up the issue of seeing better what R can do. We found it at times a difficult balancing act to demonstrate the very far-reaching capacities of R and teaching the basics of programming. In the future we will present more of the former to stimulate interest in R, as ultimately learning programming hinges a great deal on individual motivation to develop skills outside classroom.



