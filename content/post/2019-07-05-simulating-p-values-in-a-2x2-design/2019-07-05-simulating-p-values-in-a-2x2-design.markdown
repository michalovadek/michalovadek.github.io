---
title: Simulating p-values in a 2x2 design
author: 'Michal Ovadek'
date: '2019-07-05'
slug: simulating-p-values-in-a-2x2-design
categories: 
  - Data
tags:
  - data
  - rstats
  - viz
image:
  caption: ''
  focal_point: ''
---

In a recent [paper](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=3352467) in which I analysed experimental data on law students' aversion to politically motivated legal arguments, I realized there was a fairly trivial but systematic relationship between changes in the distribution of outcomes drawn from a binomial distribution in the control group and p-values. For the graphically minded among us (myself) a picture is worth a thousand Greek letters, so below I am including a plot explaining the relationship on simulated data, including the simulation and `ggplot2` code. The data concerns a 2x2 contingency matrix (control, treatment x 2 conditions with varying impact on outcome distribution in the control group) and p-values are obtained using Fisher's exact test for count data (one-sided).


```r
library(tidyverse)
library(Exact)
library(ggrepel)

# simulate

set.seed(1923)

n = as.integer(40)
amb <- seq(0.05,0.50,by=0.05) 
  
d <- dbinom(x = 0:n, size = n, prob = amb[1]) %>% 
  tibble(x = 0:n, n, p = ., amb = amb[1])

for (i in 1:length(amb)){
  
  d <- dbinom(x = 0:n, size = n, prob = amb[i]) %>% 
    tibble(x = 0:n, n, p = ., amb = amb[i]) %>% 
    bind_rows(d,.)
  
}

sampled <- d %>% 
  sample_n(10, replace = TRUE, weight = p) %>% # increase sample to > 1000
  rename(b = x) %>% 
  mutate(a = as.integer(n - b),
         amb_obs = b/n) %>% 
  crossing(c = 0:40) %>% 
  mutate(d = as.integer(n - c)) %>% 
  rowwise() %>% 
  mutate(p.value = fisher.test(tibble(j = c(a,c), k = c(b,d)),alternative = "greater") %>% 
           broom::tidy() %>% 
           select(p.value) %>% 
           deframe())

dat <- sampled %>% 
  group_by(amb,d) %>% 
  summarise(p.value = mean(p.value)) %>% 
  mutate(lbl = ifelse(amb %in% c(0.05,0.50) & p.value < 0.05 & p.value > 0.035,paste("x = ",d,sep = ""),NA))

# plot

dat %>% 
  ggplot(aes(x=d, y = p.value, color = amb)) +
  geom_hline(yintercept = 0.05, color = "grey90", lty = 2, size = 1.2) +
  annotate(geom = "text", x = 35, y = 0.1, label = expression(alpha~" = 0.05"), color = "grey90", size = 5) +
  geom_point(alpha = 0.7) +
  #geom_smooth(aes(y = p.value, color = amb)) +
  geom_label_repel(aes(label = lbl, x = d, y = p.value),
                   fontface = "italic",
                   segment.color = "grey80",
                   segment.alpha = 0.5,
                   segment.size = 0.8,
                   nudge_x = ifelse(deframe(na.omit(dat)[,2])>32,2,-3.7),
                   nudge_y = ifelse(deframe(na.omit(dat)[,2])>32,0.1,-0.03)) +
  scale_color_distiller(palette = "Spectral", direction = -1,
                        limits = c(0.05,0.50),
                        breaks = c(0.05,0.20,0.35,0.50),
                        labels = c(0.05,0.20,0.35,0.50)) +
  theme_minimal() +
  labs(x = "Number of hypothesis-confirming responses in treatment group", y = "p-value", color = "Ambiguity") +
  theme(panel.background = element_rect(fill = "grey20"),
        panel.grid = element_line(color = "grey40"),
        legend.position = c(0.775,0.72),
        legend.background = element_blank(),
        legend.key.size = unit(10,"mm"),
        legend.text = element_text(color = "grey90"),
        legend.title = element_text(color = "grey90", face = "bold"),
        legend.spacing.y = unit(4,"mm"),
        legend.box = "horizontal",
        legend.direction = "horizontal",
        panel.grid.major.y = element_blank(),
        panel.grid.minor.y = element_line(color = "grey30", linetype = 2),
        panel.grid.major.x = element_line(linetype = 3),
        panel.grid.minor.x = element_blank()) +
  guides(color = guide_colourbar(title.position="top", title.hjust = 0.5, reverse = F))
```

<div class="figure">
<img src="/post/2019-07-05-simulating-p-values-in-a-2x2-design/2019-07-05-simulating-p-values-in-a-2x2-design_files/figure-html/unnamed-chunk-1-1.png" alt="Note the low number of samples makes some lines look crooked. In a real application the number of iterations must be at least 1000. I am skimping on samples here to reduce computation." width="672" />
<p class="caption">Figure 1: Note the low number of samples makes some lines look crooked. In a real application the number of iterations must be at least 1000. I am skimping on samples here to reduce computation.</p>
</div>

This little project also made me realize that the spectral color theme looks rather good on a dark background. Although not the most conventional, I will definitely consider darker plot backgrounds in future visualizations.


