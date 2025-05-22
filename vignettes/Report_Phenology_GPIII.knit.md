---
title: "Report_Phenology_GPIII"
author: "Dario Maccioni Vasquez"
date: "2025-05-22"
output: html_document
---

# 1. Search for the right data in MODISTools

Search for all products in MODISTools and get the right one


```r
# load the library
library("MODISTools")

# list all available products
# (only showing first part of the table for brevity)
MODISTools::mt_products() |> 
  head()
```

```
##        product
## 1       Daymet
## 2 ECO4ESIPTJPL
## 3      ECO4WUE
## 4       GEDI03
## 5     GEDI04_B
## 6      MCD12Q1
##                                                                          description
## 1 Daily Surface Weather Data (Daymet) on a 1-km Grid for North America, Version 4 R1
## 2               ECOSTRESS Evaporative Stress Index PT-JPL (ESI) Daily L4 Global 70 m
## 3                          ECOSTRESS Water Use Efficiency (WUE) Daily L4 Global 70 m
## 4                GEDI Gridded Land Surface Metrics (LSM) L3 1km EASE-Grid, Version 2
## 5     GEDI Gridded Aboveground Biomass Density (AGBD) L4B 1km EASE-Grid, Version 2.1
## 6              MODIS/Terra+Aqua Land Cover Type (LC) Yearly L3 Global 500 m SIN Grid
##   frequency resolution_meters
## 1     1 day              1000
## 2    Varies                70
## 3    Varies                70
## 4  One time              1000
## 5  One time              1000
## 6    1 year               500
```

```r
View(mt_products())
```

## 1.1 Search for right Band for Greenup

Select from the chosen products the right bands, which are Greenup.Num_Modes_01 and Maturity.Num_Modes_01.


```r
# list bands for the MOD11A2
# product (a land surface temperature product)
MODISTools::mt_bands("MCD12Q2") |> 
  head()
```

```
##                         band    description               units    valid_range
## 1      Dormancy.Num_Modes_01 Onset_Dormancy days since 1-1-1970 11138 to 32766
## 2      Dormancy.Num_Modes_02 Onset_Dormancy days since 1-1-1970 11138 to 32766
## 3 EVI_Amplitude.Num_Modes_01  EVI_Amplitude           NBAR-EVI2     0 to 10000
## 4 EVI_Amplitude.Num_Modes_02  EVI_Amplitude           NBAR-EVI2     0 to 10000
## 5      EVI_Area.Num_Modes_01       EVI_Area           NBAR-EVI2      0 to 3700
## 6      EVI_Area.Num_Modes_02       EVI_Area           NBAR-EVI2      0 to 3700
##   fill_value scale_factor
## 1      32767          1.0
## 2      32767          1.0
## 3      32767       0.0001
## 4      32767       0.0001
## 5      32767          0.1
## 6      32767          0.1
```

```r
View(mt_bands("MCD12Q2"))
```


## 1.2 Search for right Band for Land Cover (IGBP)

Search for bands in land cover product


```r
# list bands for the MOD11A2
# product (a land surface temperature product)
MODISTools::mt_bands("MCD12Q1") |> 
  head()
```

```
##                  band
## 1            LC_Prop1
## 2 LC_Prop1_Assessment
## 3            LC_Prop2
## 4 LC_Prop2_Assessment
## 5            LC_Prop3
## 6 LC_Prop3_Assessment
##                                                       description   units
## 1 FAO-Land Cover Classification System 1 (LCCS1) land cover layer   class
## 2                               LCCS1 land cover layer confidence percent
## 3                                        FAO-LCCS2 land use layer   class
## 4                                 LCCS2 land use layer confidence percent
## 5                               FAO-LCCS3 surface hydrology layer   class
## 6                        LCCS3 surface hydrology layer confidence percent
##   valid_range fill_value
## 1     1 to 43        255
## 2    0 to 100        255
## 3     1 to 40        255
## 4    0 to 100        255
## 5     1 to 51        255
## 6    0 to 100        255
```

```r
View(mt_bands("MCD12Q1"))
```


# 2. Download of the Data

## 2.1 Greenup Phenology Data

Download Greenup Phenology


```r
# load libraries
library(MODISTools)

# download and save phenology data
Greenup <- MODISTools::mt_subset(
  product = "MCD12Q2",
  lat = 43.5,
  lon = 74.5,
  band = "Greenup.Num_Modes_01",
  start = "2001-01-01",
  end = "2009-12-31",
  km_lr = 100,
  km_ab = 100,
  site_name = "Adirondacks (North-Eastern United States)",
  internal = TRUE,
  progress = FALSE)

# print the dowloaded data
head(subset)
```

```
##                      
## 1 function (x, ...)  
## 2 UseMethod("subset")
```


## 2.2 LC_Type1 (IGBP)

Nur IGBF Klassen herunerladen


```r
# load libraries
library(MODISTools)

# download and save phenology data
IGBP <- MODISTools::mt_subset(
  product = "MCD12Q1",
  lat = 43.5,
  lon = 74.5,
  band = "LC_Type1",
  start = "2010-01-01",
  end = "2010-12-31",
  km_lr = 100,
  km_ab = 100,
  site_name = "Adirondacks (North-Eastern United States)",
  internal = TRUE,
  progress = FALSE)

# print the dowloaded data
head(subset)
```

```
##                      
## 1 function (x, ...)  
## 2 UseMethod("subset")
```



# 3. Filter and Merge Broadleaf and Mixed Forest Data (LC_Type1 Band) with Greenup Data


```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
Landcover_filtered <- IGBP |> 
  filter(value %in% c(4, 5))
```






# 4. Screening of the Greenup Data

Aus Data Frame Daten filtern (NA werte)


```r
library(dplyr)
library(terra)
```

```
## terra 1.7.78
```


```r
library(dplyr)

Greenup <- Greenup |> 
  mutate(
    value = ifelse(value > 32656, NA, value),
    doy = as.numeric(format(as.Date("2010-01-01") + value, "%j")),
    doy = ifelse(doy < 200, doy, NA),
    year = format(as.Date(calendar_date), "%Y")
  )
```


# 5. LTM and SD Calculation and Early/Late Greenup for 2009

## 5.1 LTM and SD


```r
# Greenup: nur 2001–2009
Greenup_ltm <- Greenup |>
  filter(year >= 2001 & year <= 2009) |>
  group_by(latitude, longitude) |>
  summarise(
    ltm = mean(doy, na.rm = TRUE),
    sd = sd(doy, na.rm = TRUE),
    .groups = "drop"
  )
```

## 5.2 Early/Late Greenup


```r
# Greenup 2010
Greenup_2009 <- Greenup |> filter(year == 2009)

Greenup_eval <- Greenup_2009 |>
  left_join(Greenup_ltm, by = c("latitude", "longitude")) |>
  mutate(early_greenup = doy < (ltm - sd))
```

















