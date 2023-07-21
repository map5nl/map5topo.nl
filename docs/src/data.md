# Data

Source Datasets and Data model.

## Intro

map5topo is composed of multiple (open) datasets. The aim is to fit
these all into a single model, classification and DB-schema.

This is an ongoing process. The name of the (PostGIS) schema is `map5`.
Tables in this schema are filled with data from other schemas/tables that
contain the various (vector) sources like OpenStreetMap, BRT, BRK, BGT, BAG etc.
SQL scripts are used for this transformation.

Inspiration:
* https://openmaptiles.org/schema/
* https://github.com/PDOK/vectortiles-bgt-brt/tree/master/sql


## Table Setup

The table-SQL can be found here: https://github.com/map5nl/map5topo/tree/main/tools/etl/sql/map5/tables.

Each table in the map5 schema has a simlar setup, i.e. columns:

```
CREATE TABLE map5.xyz (
    -- table-specific columns, usually classifications 
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
classification is mapped.

For example: `lod1`: `trees`, `lod2: broadleaved|pine|mixed`. `lod3` is usually
the source-specific value like `naaldbos`.

## Zoom-specific Selection

Each table contains multiple generalizations. Per record the zoomlevel range
is specified with `rdz_min`-`rdz_max`. Mapnik always provides a scaledenominator
when accessing a `Layer`.  Via the SQL Function `rdz()` this scaledenominator is
converted to an RD Zoomlevel (1-13) that is used in the query on that Layer.
This way only the relevant records are selected. Many zoom-ranges also have VIEWs.
For example for low-zoom Terrain:

```
CREATE VIEW map5.terrain_z0_z3 AS SELECT
   lc.*
FROM map5.terrain lc WHERE lc.rdz_min >= 0 AND lc.rdz_max <= 3;

```

This also works for Web Mercator tiles are scale ranges are shared, see e.g.
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

## Data assignment

Per map5 object type (table) and zoomlevel the following source datasets
are used. 


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

## Refs

* https://openmaptiles.org/schema/
* https://github.com/PDOK/vectortiles-bgt-brt
