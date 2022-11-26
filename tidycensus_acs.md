Survey Data Analysis and Visualization with Tidycensus
================
Geoff
2022-11-19

# Introduction

This script follows code provided in [Analyzing US Census Data: Methods,
Maps, and Models in R](https://walker-data.com/census-r/index.html) to
create a map linked to a chart. The example used is median annual
household income in the Northern Virginia area.

# Code

Load the necessary packages

``` r
library(tidycensus)
library(tidyverse)
library(patchwork)
library(ggiraph)
library(scales)
```

Census API key

View available variables in ACS5 data set for 2019

``` r
# ACS5
#View(load_variables(2019, dataset = "acs5"))
```

Get Median HH Income from ACS5

``` r
nova <- c("Fairfax", "Fairfax City", "Loudoun", "Prince William", "Arlington", "Fauquier", "Culpeper", "Warren", "Clarke", "Rappahannock", "Madison", "Fredericksburg City", "Stafford", "Alexandria City", "Manassas City", "Falls Church City", "Manassas Park City", "Winchester City", "Frederick", "Caroline", "Shenandoah")

va_income <- get_acs(variables = c("B19013_001"), 
               geography = "county", 
               state = "VA", 
               year = 2019, 
               survey = 'acs5',
               geometry = TRUE) %>%
  mutate(NAME = str_remove(NAME, " County, Virginia"),
         NAME = str_replace(NAME, "city, Virginia", "City")) %>%
  filter(NAME %in% nova)
```

Next we can visualize median household income by county in the last 12
months

``` r
va_map <- ggplot(va_income, aes(fill = estimate)) + 
  geom_sf_interactive(aes(data_id = GEOID)) + 
  scale_fill_distiller(palette = "Blues", 
                       direction = 1, 
                       guide = "none") +
  theme_void()

va_plot <- ggplot(va_income, aes(x = estimate, y = reorder(NAME, estimate), fill = estimate)) +
  geom_errorbar(aes(xmin = estimate - moe, xmax = estimate + moe)) + 
  geom_point_interactive(color = "black", size = 4, shape = 21, aes(data_id = GEOID)) +
  scale_fill_distiller(palette = "Blues", direction = 1, labels = label_dollar()) + 
  scale_x_continuous(labels = label_dollar()) + 
  labs(title = "NOVA Median Household Income (county)",
       subtitle = "2015-2019 American Community Survey",
       y = "",
       x = "ACS estimate (bars represent margin of error)",
       fill = "ACS estiamte") + 
  theme_minimal(base_size = 14)

girafe(ggobj = va_map + va_plot) %>%
  girafe_options(opts_hover(css = "fill:cyan;"))
```

![](tidycensus_acs_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->
