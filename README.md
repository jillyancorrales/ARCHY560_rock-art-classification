# README


<link href="Readme_files/libs/htmltools-fill-0.5.9/fill.css" rel="stylesheet" />
<script src="Readme_files/libs/htmlwidgets-1.6.4/htmlwidgets.js"></script>
<script src="Readme_files/libs/jquery-3.6.0/jquery-3.6.0.min.js"></script>
<link href="Readme_files/libs/leaflet-1.3.1/leaflet.css" rel="stylesheet" />
<script src="Readme_files/libs/leaflet-1.3.1/leaflet.js"></script>
<link href="Readme_files/libs/leafletfix-1.0.0/leafletfix.css" rel="stylesheet" />
<script src="Readme_files/libs/proj4-2.6.2/proj4.min.js"></script>
<script src="Readme_files/libs/Proj4Leaflet-1.0.1/proj4leaflet.js"></script>
<link href="Readme_files/libs/rstudio_leaflet-1.3.1/rstudio_leaflet.css" rel="stylesheet" />
<script src="Readme_files/libs/leaflet-binding-2.2.3/leaflet.js"></script>
<link href="Readme_files/libs/leaflet-markercluster-1.0.5/MarkerCluster.css" rel="stylesheet" />
<link href="Readme_files/libs/leaflet-markercluster-1.0.5/MarkerCluster.Default.css" rel="stylesheet" />
<script src="Readme_files/libs/leaflet-markercluster-1.0.5/leaflet.markercluster.js"></script>
<script src="Readme_files/libs/leaflet-markercluster-1.0.5/leaflet.markercluster.freezable.js"></script>
<script src="Readme_files/libs/leaflet-markercluster-1.0.5/leaflet.markercluster.layersupport.js"></script>

## Hypothesis

Hypothesis: There are discernible spatial clusters in the motifs of the
Kokentau ochre rock art paintings.

Null Hypothesis: There is no spatial patterning in the motifs within the
Kokentau ochre rock art paintings.

Alternative Explanation: The data collected within the Kokentau Database
is biases - perhaps some motifs are easier to see or classify than
others, leading to more of these in the data.

## Variables

``` r
# Import data
library(dplyr)
```

    Attaching package: 'dplyr'

    The following objects are masked from 'package:stats':

        filter, lag

    The following objects are masked from 'package:base':

        intersect, setdiff, setequal, union

``` r
attributes <- readxl::read_excel("data/Kokentau_Attributes.xlsx")

# Select columns and show the first five rows
features <- attributes |> select( Number, N, E, `Amount of figures`, `Anthropomorphic stick figures`, `Panel-bodied anthropomorphs`, `Zoomorphs`, `Geometric motifs`, `Undetermined motifs`)

features_sample <- features |> slice_head(n = 5)
```

The main variables of interest here are the spatial information,
recorded in the ‘N’ and ‘E’ (Northing and Easting) columns of the data
set, and the motifs classified in the
columns`Anthropomorphic stick figures`, `Panel-bodied anthropomorphs`,
`Zoomorphs`, `Geometric motifs`, `Undetermined motifs`. These are the
input variables that are needed for the current project. In addition, I
included the total number of figures recorded to compare with the
numbers classified in each column and the `Number` column to act as an
identifier. For this project, the weakest link is the
`Undetermined motifs` column, which the researchers used to record the
motifs that could not be classified. I think that this column is the
weakest link because it lumps together anything that could not be
classified, no matter what they look like. These may end up as outliers
or complicating factors. I don’t think any variables are necessarily
missing.

## A First Look without Machine Learning

Because I am interested in spatial clustering, I am creating a basic map
in order to see if there are any identifiable clusters in the data. To
do so, I will plot each point on the map and color-code them by motif
type. As there are several motifs within one latitude/ longitude, the
map will be interactive using the leaflet package so that users can
click through different motif clusters.

``` r
# Import packages
library(leaflet)
library(tidyr)

# Rename northing and easting columns
features <-rename(features, lng = E, lat = N)
features$lng <- as.numeric(features$lng)
features$lat <- as.numeric(features$lat)

#Transform to tidy data
features_tidy <-  pivot_longer(features, 
                               cols= `Anthropomorphic stick figures`:`Undetermined motifs`, 
                               names_to = "motifs", 
                               values_to = "frequency") |>
  filter(frequency > 0)

# Create Map
map <- leaflet(features_tidy) %>%
  addTiles() %>%
  setView(lng = 79.6, lat = 49.8, zoom = 10) %>% 
  addCircleMarkers(lng = ~lng, lat = ~lat,
                   radius = 5,
                   fill = TRUE,
                   stroke = FALSE,
                   clusterOptions = markerClusterOptions(),
                   color = ~case_when(motifs == "Anthropomorphic stick figures" ~ "red",
                                      motifs == "Panel-bodied anthropomorphs" ~ "darkgreen",
                                      motifs == "Zoomorphs" ~ "orange",
                                      motifs == "Geometric motifs" ~ "blue",
                                      motifs == "Undetermined motifs" ~ "darkgrey"), 
                   popup = ~paste0("<b>Motif:</b> ", motifs, "<br>","<b>Frequency:</b> ", frequency)) |>
                     addLegend(position = "bottomright", 
                               colors = c("red", "darkgreen", "orange", "blue", "darkgrey"),,
                               labels = c("Anthropomorphic stick figures", "Panel-bodied anthropomorphs",
                                          "Zoomorphs","Geometric motifs", "Undetermined motifs"))


map
```

<div class="leaflet html-widget html-fill-item" id="htmlwidget-c0131c32e1176b792d90" style="width:672px;height:480px;"></div>
<script type="application/json" data-for="htmlwidget-c0131c32e1176b792d90">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"https://openstreetmap.org/copyright/\">OpenStreetMap<\/a>,  <a href=\"https://opendatacommons.org/licenses/odbl/\">ODbL<\/a>"}]},{"method":"addCircleMarkers","args":[[49.84513,49.84513,49.84513,49.854313,49.854313,49.854313,49.854313,49.854313,49.854328,49.854328,49.854328,49.854329,49.854329,49.8543036622116,49.857625,49.857625,49.857625,49.857625,49.857832,49.857965,49.857795,49.857795,49.857795,49.858903,49.858903,49.858903,49.86003,49.86003,49.859797,49.859762,49.859762,49.859762,49.859791,49.859791,49.859835,49.859783,49.859783,49.859822,49.859822,49.859822,49.859742,49.85981,49.85981,49.8597937901674,49.859376,49.859376,49.859376,49.859376,49.859376,49.859446,49.859446,49.859446,49.8594283,49.8594283,49.859415,49.859415,49.859415,49.8593183,49.8593183,49.8593183,49.8593183,49.8592533,49.8592533,49.8592533,49.8593917,49.859318,49.859318,49.859318,49.85924,49.859183,49.859183,49.859183,49.8591699999999,49.862386,49.862386,49.862386,49.864191,49.864191,49.864191,49.864199,49.864824,49.865159,49.865159,49.865867,49.865867,49.865867,49.865868,49.865868,49.865983,49.865983,49.865983,49.865983,49.86651,49.86651,49.86651,49.8665952289568,49.8666193665602,49.866894,49.866894,49.866894,49.866894,49.8668547075608,49.867048,49.867145,49.867145,49.8672499571741,49.8672499571741,49.8672499571741,49.8672499571741,49.8458267,49.857675,49.8576831032935,49.8576831032935,49.8634867,49.8634867,49.865495,49.865495,49.865525619299,49.865525619299,49.865525619299,49.865525619299,49.866065,49.866065,49.8670483,49.8670483,49.8670483,49.8670483,49.8679217,49.8678333,49.86803,49.86812,49.86812,49.8685617,49.866345,49.866345,49.866345,49.86085,49.860725,49.860725,49.860725,49.8592317,49.8593067,49.8593067,49.8592967,49.8592967,49.8592967,49.8605383,49.8631883,49.8631883,49.8631883,49.8643067,49.8643067,49.8643833,49.8441217,49.8441217,49.8441217,49.8441217,49.8441217,49.856926,49.856926],[79.607938,79.607938,79.607938,79.60147499999999,79.60147499999999,79.60147499999999,79.60147499999999,79.60147499999999,79.601355,79.601355,79.601355,79.601371,79.60129000000001,79.60146388789261,79.60060900000001,79.60060900000001,79.60060900000001,79.60060900000001,79.60087,79.60051799999999,79.600472,79.600472,79.600472,79.59959600000001,79.59959600000001,79.59959600000001,79.60185,79.60185,79.601596,79.601851,79.601851,79.601851,79.601821,79.601821,79.601884,79.601871,79.601871,79.601722,79.601722,79.601722,79.601676,79.601878,79.601878,79.60187405663009,79.605751,79.605751,79.605751,79.605751,79.605751,79.605608,79.605608,79.605608,79.60558,79.60558,79.60569169999999,79.60569169999999,79.60569169999999,79.60559000000001,79.60559000000001,79.60559000000001,79.60559000000001,79.6055083,79.6055083,79.6055083,79.60554329999999,79.61448300000001,79.61448300000001,79.61448300000001,79.61475299999999,79.614653,79.614653,79.614653,79.614576,79.593022,79.593022,79.593022,79.60281500000001,79.60281500000001,79.60281500000001,79.60264100000001,79.59815399999999,79.597498,79.597498,79.596538,79.596538,79.596538,79.596537,79.596537,79.601061,79.601061,79.601061,79.601061,79.60230300000001,79.60230300000001,79.60230300000001,79.6025264648222,79.6025451886447,79.604522,79.604522,79.604522,79.604522,79.60474289731511,79.605564,79.605727,79.605727,79.60589675287891,79.60589675287891,79.60589675287891,79.60589675287891,79.596765,79.59775670000001,79.5977495495972,79.5977495495972,79.6043217,79.6043217,79.60893830000001,79.60893830000001,79.6090107586146,79.6090107586146,79.6090107586146,79.6090107586146,79.6085183,79.6085183,79.6081717,79.6081717,79.6081717,79.6081717,79.6069517,79.60675500000001,79.6069883,79.60567500000001,79.60567500000001,79.6055633,79.616085,79.616085,79.616085,79.61602499999999,79.5930817,79.5930817,79.5930817,79.5912233,79.59108670000001,79.5907883,79.59029169999999,79.59029169999999,79.59029169999999,79.5874067,79.58952499999999,79.58952499999999,79.58952499999999,79.5894517,79.5894517,79.58942500000001,79.60097500000001,79.60097500000001,79.60097500000001,79.60097500000001,79.60097500000001,79.612562,79.612562],5,null,null,{"interactive":true,"className":"","stroke":false,"color":["red","darkgreen","blue","red","darkgreen","orange","blue","darkgrey","red","darkgreen","blue","darkgreen","darkgreen","red","red","orange","blue","darkgrey","red","red","red","orange","darkgrey","red","darkgreen","blue","darkgreen","darkgrey","darkgreen","red","darkgreen","orange","red","darkgreen","blue","red","darkgreen","darkgreen","blue","darkgrey","blue","darkgreen","darkgreen","red","red","darkgreen","orange","blue","darkgrey","red","darkgreen","blue","red","darkgreen","red","darkgreen","darkgrey","red","darkgreen","blue","darkgrey","red","darkgreen","darkgrey","blue","red","darkgreen","darkgrey","darkgreen","red","blue","darkgrey","darkgreen","red","darkgreen","darkgrey","red","darkgreen","blue","darkgreen","red","red","darkgrey","red","darkgreen","darkgrey","darkgreen","darkgrey","red","darkgreen","blue","darkgrey","red","darkgreen","darkgrey","darkgrey","red","red","darkgreen","orange","darkgrey","red","red","orange","blue","red","orange","blue","darkgrey","blue","darkgreen","red","darkgrey","red","darkgrey","red","darkgrey","red","darkgreen","blue","darkgrey","red","darkgrey","red","darkgreen","blue","darkgrey","red","darkgreen","darkgrey","blue","darkgrey","red","red","darkgreen","darkgrey","darkgrey","red","darkgreen","blue","blue","darkgrey","darkgreen","red","darkgreen","darkgrey","blue","red","darkgreen","blue","red","darkgreen","darkgrey","red","darkgreen","orange","blue","darkgrey","red","darkgreen"],"weight":5,"opacity":0.5,"fill":true,"fillColor":["red","darkgreen","blue","red","darkgreen","orange","blue","darkgrey","red","darkgreen","blue","darkgreen","darkgreen","red","red","orange","blue","darkgrey","red","red","red","orange","darkgrey","red","darkgreen","blue","darkgreen","darkgrey","darkgreen","red","darkgreen","orange","red","darkgreen","blue","red","darkgreen","darkgreen","blue","darkgrey","blue","darkgreen","darkgreen","red","red","darkgreen","orange","blue","darkgrey","red","darkgreen","blue","red","darkgreen","red","darkgreen","darkgrey","red","darkgreen","blue","darkgrey","red","darkgreen","darkgrey","blue","red","darkgreen","darkgrey","darkgreen","red","blue","darkgrey","darkgreen","red","darkgreen","darkgrey","red","darkgreen","blue","darkgreen","red","red","darkgrey","red","darkgreen","darkgrey","darkgreen","darkgrey","red","darkgreen","blue","darkgrey","red","darkgreen","darkgrey","darkgrey","red","red","darkgreen","orange","darkgrey","red","red","orange","blue","red","orange","blue","darkgrey","blue","darkgreen","red","darkgrey","red","darkgrey","red","darkgrey","red","darkgreen","blue","darkgrey","red","darkgrey","red","darkgreen","blue","darkgrey","red","darkgreen","darkgrey","blue","darkgrey","red","red","darkgreen","darkgrey","darkgrey","red","darkgreen","blue","blue","darkgrey","darkgreen","red","darkgreen","darkgrey","blue","red","darkgreen","blue","red","darkgreen","darkgrey","red","darkgreen","orange","blue","darkgrey","red","darkgreen"],"fillOpacity":0.2},{"showCoverageOnHover":true,"zoomToBoundsOnClick":true,"spiderfyOnMaxZoom":true,"removeOutsideVisibleBounds":true,"spiderLegPolylineOptions":{"weight":1.5,"color":"#222","opacity":0.5},"freezeAtZoom":false},null,["<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 53","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 6","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 5","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 6","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 7","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 31","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 6","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 12","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 5","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 9","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 6","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 9","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 5","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 7","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 5","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 7","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 5","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 16","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 12","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 37","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 36","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 7","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 6","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 12","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 5","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 8","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 7","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 29","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 12","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 9","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 4","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 2","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 3","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 11","<b>Motif:<\/b> Zoomorphs<br><b>Frequency:<\/b> 11","<b>Motif:<\/b> Geometric motifs<br><b>Frequency:<\/b> 40","<b>Motif:<\/b> Undetermined motifs<br><b>Frequency:<\/b> 9","<b>Motif:<\/b> Anthropomorphic stick figures<br><b>Frequency:<\/b> 1","<b>Motif:<\/b> Panel-bodied anthropomorphs<br><b>Frequency:<\/b> 2"],null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]},{"method":"addLegend","args":[{"colors":["red","darkgreen","orange","blue","darkgrey"],"labels":["Anthropomorphic stick figures","Panel-bodied anthropomorphs","Zoomorphs","Geometric motifs","Undetermined motifs"],"na_color":null,"na_label":"NA","opacity":0.5,"position":"bottomright","type":"unknown","title":null,"extra":null,"layerId":null,"className":"info legend","group":null}]}],"setView":[[49.8,79.59999999999999],10,[]],"limits":{"lat":[49.8441217,49.8685617],"lng":[79.5874067,79.616085]}},"evals":[],"jsHooks":[]}</script>

Map showing the location and type of motifs in the dataset

**?@fig-features-visualization** shows each point in the Kokentau
database plotted in its spatial location and color coded by motif type.
On first glance, there does not appear to be any discernible pattern in
the motifs. One thing that I did notice was that the southern cliff
faces appear to mostly have geometric and undetermined motifs while the
more northern cliff faces included a wider array. While this figure
alone does not appear to show strong evidence of spatial clustering, a
machine learning method could quantitatively examine whether any
clustering exists that may not be evident in a map alone.

## Predictions

As I will be using primarily unsupervised learning methods, such as
spatially constrained clustering and random forest with
cross-validation, I expect that my output will be maps and confusion
matrices. Since the project is focusing on a spatial clustering
question, my benchmark will be a random model - the clustering method
and random forest should have stronger results than a random model.

If my hypothesis is right, the output will include a map with distinct
spatial clusters and the benchmark will show a higher accuracy (lower
error rates) than the null model in the random forest and k-fold
cross-validation.

If my null hypothesis is right, the output will be a map without
distinct spatial clusters and the benchmark will show an equivalent or
lower accuracy than the null model.

If my alternative explanation is right, the output may be similar to
that for a correct hypothesis, with distinct spatial clusters. The
benchmark will show stronger accuracy than a random model, but perhaps
it will not be as strong as if the hypothesis is correct. If clusters
exist because of collection biases, then they might not appear as
strongly as intentional clustering.
