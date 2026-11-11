# Interactive Leaflet Map in a GitHub README


## How GitHub Renders Interactive Maps in Markdown

GitHub strips `<script>` and `<iframe>` tags from `README.md` files
during HTML sanitization, which prevents R `htmlwidgets` objects from
executing JavaScript on `github.com`. [GitHub
Docs](https://docs.github.com/en/get-started/writing-on-github/working-with-advanced-formatting/creating-diagrams#creating-geojson-and-topojson-maps)
documents native rendering for `geojson` and `topojson` fenced code
blocks in Markdown. When `github.com` displays a `geojson` code block in
`README.md`, its map viewer creates an interactive Leaflet.js map with
OpenStreetMap tiles, zoom controls, and clickable property popups.

## Generating the Interactive Map from R

The code below builds a spatial table of three University of Washington
campus landmarks using `tidyverse` and `sf`. Next, it maps a
color-blind-friendly `viridis` palette to the [Mapbox
simplestyle](https://github.com/mapbox/simplestyle-spec) `marker-color`
property and writes a `geojson` fenced code block into `README.md`.

``` r
library(tidyverse)
library(sf)
library(viridisLite)

geojson_path <- tempfile(fileext = ".geojson")

landmarks <- tibble(
  name = c("Burke Museum", "Denny Hall", "Suzzallo Library"),
  year_opened = c(2019L, 1895L, 1926L),
  lon = c(-122.3106, -122.3087, -122.3080),
  lat = c(47.6603, 47.6584, 47.6557)
) |>
  mutate(`marker-color` = str_sub(viridis(n(), end = 0.85), 1, 7)) |>
  mutate(`marker-size` = "medium") |>
  st_as_sf(coords = c("lon", "lat"), crs = 4326)

st_write(
  obj = landmarks,
  dsn = geojson_path,
  driver = "GeoJSON",
  layer_options = "WRITE_NAME=NO",
  quiet = TRUE
)

geojson_text <- read_file(geojson_path)

str_c("````{=markdown}\n```geojson\n", geojson_text, "```\n````\n") |>
  cat()
```

```geojson
{
"type": "FeatureCollection",
"crs": { "type": "name", "properties": { "name": "urn:ogc:def:crs:OGC:1.3:CRS84" } },
"features": [
{ "type": "Feature", "properties": { "name": "Burke Museum", "year_opened": 2019, "marker-color": "#440154", "marker-size": "medium" }, "geometry": { "type": "Point", "coordinates": [ -122.3106, 47.6603 ] } },
{ "type": "Feature", "properties": { "name": "Denny Hall", "year_opened": 1895, "marker-color": "#277E8E", "marker-size": "medium" }, "geometry": { "type": "Point", "coordinates": [ -122.3087, 47.6584 ] } },
{ "type": "Feature", "properties": { "name": "Suzzallo Library", "year_opened": 1926, "marker-color": "#9AD93C", "marker-size": "medium" }, "geometry": { "type": "Point", "coordinates": [ -122.308, 47.6557 ] } }
]
}
```
