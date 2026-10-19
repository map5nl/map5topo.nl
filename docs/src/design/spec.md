# Specification

Goal: develop a Dutch national classic topographic map (or maps). 
Below overall specifications and constraints.

## Criteria

* at first raster (tiles)-only
* Vector (Tiles) later
* both RD and Web Mercator projections and tiling schemes (EPSG:28992 and EPSG:3857)
* use Dutch Open Data like Key Registries ("Basisregistraties") as much as possible  (see datasets below)
* may use OSM or other datasets where they provide more detail
* try **not** to use Corine and Natural Earth
* Hillshading from Dutch DEM Open Data: "AHN" in 5m and 50cm resolutions
* attempt classical design, but keep fresh
* services: tiling and WMS (may use MapProxy)
* High-quality *Retina Tiles* (512x512 pixels) 
* license and openness: start proprietary, then make open source when all in place: clean code, documentation.
* NL neighbouring countries/vicinity: Germany/Belgium/Northern France
* guard copyrights of existing maps!

## Datasets

This is a snapshot in time. Anything abroad: all from OpenStreetMap.

## Now In use (Dutch Datasets/layers)

* BRT TOP10NL:  terrein, water, hoogtepunten, dijkpalen, strandpalen
* BRT TOP50NL:  terrein, water
* BRT TOP100NL:  terrein, water
* BAG: panden
* OpenStreetMap" niet-BAG panden, huisnummers, verharde infra, onverharde wegen, terreinen (lowzoom), zee, water (lowzoom), bushaltes, boeien, administratieve grenzen (gem/prov)
* BRK-DKK: open digitale kadastrale kaart met perceelgrenzen, van het Kadaster.
* BGT grootschalige topografie: alleen zoom 13, wegdeel-vlak, alle terrein-gerelateerde vlakken, water 
* AHN5+AHN6 5m en 50cm: reliëfschaduw en contourlijnen
* RWS Nationaal Wegenbestand: hectometer en kilometer-bordjes
* Gridlines: calculated according to Kadaster conventions

## Now In use (abroad)

Anything abroad near borders (Germany, Belgium, bit France): 

* All vector data from OpenStreetMap.

For relief/hillshade and extracting contourlines the following DEM datasets are used:

### Belgium

* Flanders: [Digitaal Hoogtemodel Vlaanderen II, DTM, raster, 1m](https://www.vlaanderen.be/datavindplaats/catalogus/digitaal-hoogtemodel-vlaanderen-ii-dtm-raster-1-m): DTM, version 2014.01 - EPSG:31370 Lambert 1972 projection - [License](http://inspire.ec.europa.eu/metadata-codelist/LimitationsOnPublicAccess/noLimitations)
* Wallonie: [Relief de la Wallonie - Modèle Numérique de Terrain 2021-2022 0.5m](https://geoportail.wallonie.be/catalogue/a004e570-99d6-4fe5-b83d-49b774409278.html): DTM version 2021-2022-  EPSG:3812 Lambert Belge 2008 [License](https://creativecommons.org/licenses/by/4.0/)

### Germany

* Nordrhein-Westfalen: [Digitales Geländemodell - Rasterweite 1 m](https://www.opengeodata.nrw.de/produkte/geobasis/hm/dgm1_tiff/dgm1_tiff/), DTM version 2022-2025 - EPSG:25832 - [License](https://creativecommons.org/licenses/by/4.0/)
* Niedersachsen: [Digitales Geländemodell 1m - DGM1](https://www.lgln.niedersachsen.de/startseite/geodaten_karten/3d_geobasisdaten/dgm/digitale-gelandemodelle-dgm-143150.html) - EPSG:25832 (ETRS89/UTM 32N) - [License](https://www.govdata.de/dl-de/zero-2-0)

### France
TODO

## To consider for later

* TOPNamen: naamlabels voor woonplaatsen, wateren, natuur, dijken, diversen
* TOP10NL  spoorwegen, inrichtings- elementen, niet-BAG panden, hoogbouw, hoogtelijnen, dieptelijnen, grenzen (ook bebouwde kommen).
* BAG  woonplaatsgrenzen, postcode centroid labels.
* OpenStreetMap  stoplichten, max snelheden op wegen, toeristische punten.
* BGT grootschalige topografie: alleen zoom 13, overige objecten, bomen(?) 
* Provinciale Risicokaart: risicogevende objecten ; buisleidingen ; dijkringen.
* RWS Nationaal Wegenbestand: vaarwegen; 
* RWS Bewegwijzeringsdienst: paddenstoelen
* Hydrografische Dienst Defensie: dieptelijnen en kleurrasters zoetwater; (toestemming nodig!)
* EU-DEM: France DEM
* NS/Prorail - stationsnamen

## Extent

So the map does not stop at NL country borders!

- EPSG:28992 LL: -20000 275000  UR: 300000 650000  
- EPSG:4326  LL: 2.923846278 50.438927366  UR: 7.588656577	53.815565633    
- EPSG:3857 LL: 325481.052318 6522640.17149 UR: 844765.411364 7135303.75439
- GeoFabriek OSM Buitenland (Osmosis) `--bounding-box left=2.666 bottom=50.392  right=7.667 top=53.860`  iets ruimer
- GeoFabriek OSM Buitenland/Water Polygons EPSG:3857 LL: 296778 6514442  UR: 853487 7143686

- NL Tiling Extent is: '-285401.920, 22598.080, 595401.920, 903401.920' wel heel ruim.
 
This is the extent to be provided in EPSG:28992: LL: -20000 275000  UR: 300000 650000.

## References

* https://www.freemap.sk/?map=15/48.552055/19.321260&layers=X  https://github.com/FreemapSlovakia/freemap-mapnik/ nice hillshading!
* https://github.com/Amsterdam/tiles/ nice stack: DB T-rex, Tileserver-gl, mapproxy
* https://github.com/der-stefan/OpenTopoMap
