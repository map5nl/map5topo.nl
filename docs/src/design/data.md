---
title: Data Design
---

# Data Design

Source Datasets and Data model.

## Intro

map5topo is composed of multiple (open) datasets. The aim is to fit
these all into a single model, (feature) classification and DB-schema.

This is an ongoing process. The name of the (PostGIS) schema is `map5`.
Tables in this schema are filled with data from other schemas/tables that
contain the various (vector) sources like OpenStreetMap, BRT, BRK, BGT, BAG etc.
SQL scripts are used for this transformation.

Inspiration:

* [OpenMapTiles Schema OMT](https://openmaptiles.org/schema/)
* [PDOK Vector Tiles BGT BRT](https://github.com/PDOK/vectortiles-bgt-brt/tree/master/sql)

Both approaches use a hierarchical feature classification: OMT uses `class`, `subclass`, others use
"Level-Of-Detail": `lod1`, `lod2` etc. We like the latter conventions as it allows an endless 
hierarchy and is short to type.

## Challenges

Although reclassification has been exercised in many mapping projects like the two mentioned above,
the extra challenge here is that completely disjunct datasets are used: Dutch Key Registries (BAG, BGT, BRT etc) and OpenStreetMap.

An additional challenge is that part of the map include neighbouring countries, for which only OSM
and maybe later local datasets, is/are available. The aim is to work with completely integrated
data tables/layers, not seperate layers/styles for neighbouring countries.

## Table Setup

The table-SQL for the ETL 
can be found here: https://github.com/map5nl/map5topo/tree/main/tools/etl/sql/map5/tables.

Each table in the map5 schema has a similar setup, i.e. columns:

```
CREATE TABLE map5.xyz (
    -- table-specific columns, usually classifications, area, population etc 
    (lod1-lod3)
    ..
    ..
    ..
    
    -- COMMON COLUMNS
    --
    
    -- Relative height
    z_index INTEGER DEFAULT 0,
    
    -- min and maxzoom in Dutch RD
    -- when to show an object on the map
    rdz_min INTEGER, 
    rdz_max INTEGER, 
    
    -- Where the data record originates from
    src_schema TEXT,
    src_table TEXT,
    src_idref TEXT,
    
    -- Is the object in NL or outside (abroad)?
    abroad BOOLEAN DEFAULT FALSE,
    
    -- Geometry in Dutch EPSG
    geom GEOMETRY(POINT|POLYGON|LINESTRING, 28992)
);


```

Many tables will contain a classification, like for Landcover or POIs.
For example:

```
CREATE TABLE map5.landcover (
    lod1 TEXT,
    lod2 TEXT,
    lod3 TEXT,
    area BIGINT DEFAULT 0,
    
    -- Common columns
    z_index INTEGER DEFAULT 0,
    rdz_min INTEGER DEFAULT -1,
    rdz_max INTEGER DEFAULT 13,
    src_schema TEXT,
    src_table TEXT,
    src_idref TEXT,
    abroad BOOLEAN DEFAULT FALSE,
    geom GEOMETRY(POLYGON, 28992)
);

```

Here `lod` through `lod3` provide an *hierarchical* classification of the
feature. This is defined within the project. Each source dataset-specific 
classification is mapped. Usually `lod1` and `lod2` is sufficient. 

For example: `lod1`: `trees`, `lod2: broadleaved|pine|mixed`. `lod3` is usually
the source-specific value like `naaldbos`, mainly for debugging.

## Zoom-specific Selection

Each table may contain multiple geometry generalizations (simplifications) 
for the same object. Per record the zoomlevel range
is specified with `rdz_min`-`rdz_max`. Mapnik always provides a `scaledenominator`
when accessing a `Layer`.  Via the SQL Function `rdz()` this scaledenominator is
converted to an RD Zoomlevel (range 1-13, equal to WebMerc 6-18) that is used in the query on that Layer.
This way only the relevant records for that zoomlevel are selected. 
Many zoom-ranges also have VIEWs, for example for analysis. If needed, for performance,
PostgreSQL *materialized VIEWs* may be applied. But at least
data for a single feature type, usually a layer, is not spread over multiple tables now.

For example a VIEW for low-zoom Terrain:

```
CREATE VIEW map5.terrain_z0_z3 AS SELECT
   lc.*
FROM map5.terrain lc WHERE lc.rdz_min >= 0 AND lc.rdz_max <= 3;

```

This also works for Web Mercator tiles as scale-ranges are shared, see e.g.
```
.

.
<!ENTITY maxscale_zoom15_rd10 "<MaxScaleDenominator>20000</MaxScaleDenominator>">
<!ENTITY minscale_zoom15_rd10 "<MinScaleDenominator>10000</MinScaleDenominator>">
<!ENTITY maxscale_zoom16_rd11 "<MaxScaleDenominator>10000</MaxScaleDenominator>">
<!ENTITY minscale_zoom16_rd11 "<MinScaleDenominator>5000</MinScaleDenominator>">
.
.
```

Using these common ranges allows parallel zoomlevels: for example RD level 11 is 16 in Web Mercator.
RD level 13 is WM level 18, the highest level etc.
Usually zoom WM = Zoom RD +5.

## Abroad

The map area includes data from countries both bordering (Germany, Belgium) and in vicinity 
(Northern France) of The Netherlands. By intersecting a (multi)polygon that contains The Netherlands' borders,
in each OSM-record can be marked as located abroad or not This is realized through a simple boolean column, 
aptly called `abroad`. For data from Dutch Key Registries, the fixed `abroad` value is `False`.

## OSM Source Data

OSM data is downloaded from geofabrik.de, and comprises:

```
  # The Netherlands
  https://download.geofabrik.de/europe/netherlands-latest.osm.pbf
  
  # Abroad
  http://download.geofabrik.de/europe/belgium-latest.osm.pbf 
  http://download.geofabrik.de/europe/germany/niedersachsen-latest.osm.pbf 
  http://download.geofabrik.de/europe/germany/nordrhein-westfalen-latest.osm.pbf 
  http://download.geofabrik.de/europe/germany/rheinland-pfalz-latest.osm.pbf 
  http://download.geofabrik.de/europe/france/nord-pas-de-calais-latest.osm.pbf 
  
  # Sea
  https://osmdata.openstreetmap.de/download/simplified-water-polygons-split-3857.zip
  https://osmdata.openstreetmap.de/download/water-polygons-split-3857.zip
```

"The Netherlands" and "Abroad" and stitched together using Osmosis. "Water" data is basically
sea, with exception of Oosterschelde. These are directly converted, using GDAL `ogr2ogr`, to PostgreSQL `PGDump`
files for quick reuse.

## Data assignment

Per map5 object type (table) and zoomlevel the following source datasets
are used. With some exceptions a single zoom-range contains data
from a single dataset.


|ZRD|ZWM   |area_label|landcover         |
|:---|:--- | :--------| :------------    |
|0   |5    |TOP10NL - OSM|OSM            |
|1   |6    |TOP10NL - OSM|OSM            |
|2   |7    |TOP10NL - OSM|OSM            |
|3   |8    |TOP10NL - OSM|OSM            |
|4   |9    |TOP10NL - OSM|OSM            |
|5   |10   |TOP10NL - OSM|OSM            |
|6   |11   |TOP10NL - OSM|TOP50NL - OSM (a)|
|7   |12   |TOP10NL - OSM|TOP50NL - OSM (a)|
|8   |13   |TOP10NL - OSM|TOP50NL - OSM (a)|
|9   |14   |TOP10NL - OSM|TOP50NL - OSM (a)|
|10  |15   |TOP10NL - OSM|TOP10NL - OSM (a)|
|11  |16   |TOP10NL - OSM|TOP10NL - OSM (a)|
|12  |17   |TOP10NL - OSM|TOP10NL - OSM (a)|
|13  |18   |TOP10NL - OSM|BGT - OSM (a)  |

* `(a)` means abroad-only
* ZRD = Zoom in RD Grid
* ZWM = Zoom in WebMercator

TO BE SUPPLIED FURTHER!!
