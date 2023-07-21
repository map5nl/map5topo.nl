# TODO List Map5Topo Dev
 
Two sections: "TODO", open items, when done move to "DONE", done items (duh).
See also the issue list: https://github.com/map5nl/map5topo/issues

## TODO
 
### General


### Data Prepare
 
- optimize DB, indexes etc full NL single tile very slow
- Saddles - OTM tools/update_saddles.sh (needed?) 
- Isolations - OTM tools/update_isolations.sh (needed?)
- viewpointdirection.sql (?)
- color-relief (AHN) needed?
- Top10NL (landuse --> terrein, functioneel gebied?)
- Top1000-50 for lower zooms

### Mapnik

- make table to compare zooms and resolutions for both RD and WebMerc
- seems that Mapbox GL/MapLibre GL is CartoCSS follow-up, and can all generate Mapnik XML style files (?)
- Style: color-relief (basemap-relief.xml) from AHN
- Legenda: mapnik Legendary: https://github.com/gravitystorm/mapnik-legendary ?
- Fonts: Noto Sans, Fira, PT Sans?
- Symbols: create fieldHockey Pitch SVG (missing)
- Style: too busy: cities, towns, villages (population-based)
- Style: create separate shaded relief map style XML
- Style: create 'light'/simple map (ala OpenSimpleTopo) style XML

### Data Serving

- MapProxy: tiling process too slow and errors
- MapProxy: create B&W maps
- MapProxy: create relief and simple layers+seeding
- MapProxy render WebMerc version

### Demo Apps

- sidebyside: investigate if WebMerc layers in particular OTM can be added. If not, allow for WM version.

## DONE

### General
- split over multiple dirs: dataprep, postgis+pgadmin4 and mapnik (later mapporxy)

### Data Prepare

- integrate parts of Belgium, Lux, Germany
- determine NL area extent in 3857
- prepare: water polygons, create NL area extent (Shp?)
- AHN3 or mix ANH3 with AHN4 --> AHN3 for now 5m + 0.5m
- DEM output dir on server under `var/map5/data`
- contour lines (AHN)
- hillshading (AHN)
- Cadastral Parcels from BRK-DKK

### Mapnik

- Style: enable water polygons (style basemap-sea.xml)
- Style: landuse=farmland was missing
- Style: natural=heath was missing
- Style: background color DEDECE such that all is a bit green/brownish where area missing
- missing fonts? install `unifont`  (?) NOT: commented out Unifont Medium to suppress warnings
- Style: hillshading from AHN
- MapProxy integration
- Render sample: render more tiles/zoomlevels
- Style: contour
- Style: Cadastral Parcels RD levels 12+13
 
### Data Serving

- MapProxy: create "Utrecht" as greater area than "Muiden"

### Demo Apps
- sidebyside: allow switching other maps to compare (now only OpenTopo)
