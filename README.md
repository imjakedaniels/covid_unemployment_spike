# COVID-19 Unemployment Spike

"Siri, show me what an outlier looks like." <br>
_"Sure, here is this week's chart of the US Unemployment Insurance Weekly Claims Report"_


<p align="center">
  <img src="https://github.com/imjakedaniels/covid_unemployment_spike/blob/master/visuals/ICSA_pandemic_spike.gif">
</p>

```r
library(quantmod) # ICSA data
library(tidyverse) # general manipulation
library(lubridate) # date manipulation
library(gganimate) # animated charts
library(extrafont) # custom fonts
```

# Get Data

```r
getSymbols("ICSA", src="FRED") # get ICSA data from FRED
```

# Convert from XTS to data frame

```r
ICSA_df <- ICSA %>%
  as.data.frame() %>% # turn xts into data frame
  mutate(date = ymd(row.names(.))) %>% # create date column using the dates in row names
  filter(date <= "2020-03-21") # 
```

# Split data

```r
# static data range
historical_ICSA <- ICSA_df %>%
  filter(date <= "2020-03-14") %>%
  rename(date_2 = date) # give different name than the animated variable

# animated data range
pandemic_spike <- ICSA_df %>%
  filter(date >= "2020-03-14")
```

# Base plot

```r
p <- pandemic_spike %>%
  ggplot(aes(x = date, y = ICSA)) +
  geom_line(size = 0.5, colour = "orange") +
  geom_line(data = historical_ICSA, aes(x = date_2, y = ICSA, group = 1),  colour = "orange") + # definte extra data set to make it static
  scale_y_continuous("Weekly Claims", 
                     labels = scales::comma_format(), 
                     breaks = c(250000, 500000, 750000, 1000000, 2000000, 3000000)) +
  expand_limits(y = 0:750000) +
  
  labs(title = "Weekly Unemployment Claims Surge to Record High in USA",
       subtitle = "~3.28 million claims were submitted in the week ending March 21 2020",
       x = "",
       caption = "Viz by @datajake — SOURCE: U.S. Employment and Training Administration, Initial Claims [ICSA]; FRED") +
  
  theme_minimal() +
  theme(text = element_text(family = "Merriweather"),
        panel.grid = element_blank(),
        panel.grid.major.y = element_line(colour = "grey95"),
        axis.title.y = element_text(size = 8, vjust = 3),
        axis.text.y = element_text(size = 6),
        axis.line.x = element_line(),
        plot.title = element_text(hjust = 0.5, face = "bold"),
        plot.subtitle = element_text(colour = "grey30", hjust = 0.5, family = "Roboto Condensed"),
        plot.caption = element_text(colour = "grey50"))
```

# Animated plot

```r
anim_plot <- p +
  transition_reveal(date) + 
  view_follow(fixed_y = FALSE, fixed_x = TRUE) # show all the x-axis and let the y grow

options(gganimate.dev_args = list(height = 4, width = 4*1.777778, units = 'in', type = "cairo", res = 144))
  
animate(plot = anim_plot, 
        renderer = gifski_renderer("visuals/ICSA_pandemic_spike.gif"),
        fps = 20, duration = 12, end_pause = 80)
```
