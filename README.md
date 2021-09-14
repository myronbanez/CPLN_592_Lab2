# Lab2
---
title: "Lab2_assignment"
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

This assignment if for you to complete a short version of the lab notes, but you have to complete a number of the steps yourself.
You will then knit this to a markdown (not an HTML) and push it to your GitHub repo.
Unlike HTML, the RMarkdown knit to `github_document` can be viewed directly on GitHub.
You will them email your lab instructor with a link to your repo.

Steps in this assignment:

1.  Make sure you have successfully read, run, and learned from the `MUSA_508_Lab2_sf.Rmd` Rmarkdown

2.  Find two new variables from the 2019 ACS data to load.
    Use `vars <- load_variables(2019, "acs5")` and `View(vars)` to see all of the variable from that ACS.
    Note that you should not pick something really obscure like count_38yo_cabinetmakers because you will get lots of NAs.

3.  Pick a neighborhood of the City to map.
    You will need to do some googling to figure this out.
    Use the [PHL Track Explorer](https://data-phl.opendata.arcgis.com/datasets/census-tracts-2010/explore?location=40.002759%2C-75.119097%2C11.91) to get the `GEOID10` number from each parcel and add them to the `myTracts` object below.
    This is just like what was done in the exercise, but with a different neighborhood of your choice.
    Remember that all GEOIDs need to be 10-characters long.

4.  In the first code chunk you will do that above and then edit the call-outs in the dplyr pipe sequence to `rename` and `mutate` your data.

5.  You will transform the data to `WGS84` by adding the correct EPSG code.
    This is discussed heavily in the exercise.

6.  You will produce a map of one of the variables you picked and highlight the neighborhood you picked.
    There are call-out within the `ggplot` code for you to edit.

7.  You can run the code chunks and lines of code as you edit to make sure everything works.

8.  Once you are done, hit the `knit` button at the top of the script window (little blue knitting ball) and you will see the output.
    Once it is what you want...

9.  Use the `Git` tab on the bottom left of right (depending on hour your Rstudio is laid out) and click the check box to `stage` all of your changes, write a commit note, hit the `commit` button, and then the `Push` button to push it to Github.

10. Check your Github repo to see you work in the cloud.

11. Email your lab instructor with a link!

12. Congrats!
    You made a map in code!

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
```
© 2021 GitHub, Inc.
Terms
Privacy
Security
Status
Docs
Contact GitHub
Pricing
API
Training
Blog
About
