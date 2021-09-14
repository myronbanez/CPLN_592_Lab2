# Lab 2
author: "Myron Bañez"
date: "9/13/2021"
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

```{r setup_package, warning = FALSE, message = FALSE}
library(tidyverse)
library(tidycensus)
library(sf)
library(tmap) # mapping, install if you don't have it
set.seed(717)
```

## Load data from {tidycensus}

```{r acs_vars, cache = TRUE, message = FALSE, warning = FALSE, results=FALSE}
Lab_acs_vars <- c("B08111_001E", # Total Means of Transportation
                  "B08111_005E", # Means of Transportation for Non U.S. Citizens
                  "B05001_001E", # Total Citizens
                  "B05001_006E") # Total Non U.S. Citizens 
Lab_myTracts <- c("42101000200", "42101000500", "42101000901", "42101000902", "42101001101","42101001102", "42101001500", "42101001800","42101002400")
Lab_acsTractsPHL.2016.sf <- get_acs(geography = "tract",year = 2016, variables = Lab_acs_vars, geometry = TRUE, state = "PA",county = "Philadelphia", output = "wide") %>% 
  rename (total_transit.2016 = B08111_001E,
          total_noncitizenTransit.2016 = B08111_005E,
          total_citizen.2016 = B05001_001E,
          total_noncitizen.2016 = B05001_006E) %>%
  mutate(citizenPct.2016 = total_noncitizen.2016/total_citizen.2016,
         pctnoncitizenTransit.2016 = total_noncitizenTransit.2016/total_transit.2016) %>%
mutate(AsianNeighborhood = ifelse(GEOID %in% Lab_myTracts, "SOUTH PHILLY", "REST OF PHILADELPHIA"))
```

## Transform to WGS84 with {sf}

```{r}
Lab_acsTractsPHL.2016.sf_WGS84 <- Lab_acsTractsPHL.2016.sf %>% 
  st_transform(crs = "EPSG:4326")
st_crs(Lab_acsTractsPHL.2016.sf_WGS84)
```

## Plot with {ggplot2}

```{r ggplot_geom_sf, warning = FALSE, echo = FALSE}
ggplot()+
  geom_sf(data = Lab_acsTractsPHL.2016.sf, aes(fill = pctnoncitizenTransit.2016),
          color = "transparent")+
  geom_sf(data = Lab_acsTractsPHL.2016.sf %>%
            filter(AsianNeighborhood == "SOUTH PHILLY") %>%
            st_union(),
          color = "white",
          fill = "transparent")+
  labs(
    title = "Percentage of Non U.S. Citizens with Means of Transportation",
    subtitle = "",
    caption = "Data: US Census Bureau, ACS 5-year estimates")
