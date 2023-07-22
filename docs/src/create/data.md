---
title: Data Preparation
---

# Data

Describes how to prepare data using the GitHub repo. This is also called "ETL".

## 1. Startup (once)
 
* `git clone https://github.com/map5nl/map5topo.git`
* edit `git/env.sh` for your local hostname such that the main dirs are set by adding your host to `case` switch
* get credentials storage server, have your SSH public key (usually `id_rsa.pub`)  added for rsync without password

Two global env vars are important, but have defaults:

* `DEPLOY_SERVER`: set via hostname, usually `"local"`, only for production server it is `"prod"`. Has to do with SSL and domain settings mostly.
* `MAP_AREA`  defaults: `muiden` on local systems and `nl`  when `DEPLOY_SERVER=prod`.

Possible values for MAP_AREA: `muiden, utrecht, nl`.
To override `MAP_AREA`, e.g. to create full NL on local system 
e.g.: `export MAP_AREA=nl` or `export MAP_AREA=utrecht`
in bash shell before calling any tools.

## 2. Overall

Data steps:

* DEM: pre-processing AHN DTM and hillshading and contour lines
* BGT download/import
* BRT download/import
* BRK (Parcels) download/import
* NWB (Hectoborden and Kilometerpoints) download/import
* OSM Data ETL: download PBF and convert to PostGIS
* DEM: generate contour-lines and put in PostGIS as lines
* tiles: seed
* test: generate Mapnik tiles and web services

All steps from git root dir. Many datasets have already been prepared
in particular DEM (AHN Hillshade and Contours) so these only need to be downloaded.

`map5` is a special dataset: this is a DB schema generated locally from the above dataset. It creates
tables that mix records from loaded datasets. So it always needs to run as last.

## 2. Data ETL
 
These are the minimum steps. Most datasets have smaller areas available: 'muiden' and (province) 'utrecht'

You may want to set MAP_AREA to test
'muiden' first: `export MAP_AREA=muiden` in the same terminal session as running the commands. 

`MAP_AREA` possible values `muiden`, `utrecht`, `nl` (whole country)

The database needs to run, as a minimum do: `cd services/postgis; ./start.sh`. This also starts
pgadmin4, a web-based DB manager. To access you need to start 'Traefik' service as well: `cd services/traefik; ./start.sh`

Or run entire service stack, including MapProxy and apps: `cd services; ./start.sh`

Now download and load all data.

``` {.bash linenums="1"}
cd tools

# all steps in one:  
# downloads all data and runs etl for DB, all for MAP_AREA
./etl-all.sh

```
On 'local' installations, all downloaded data is under `git/data`.
This should be guarded to not push these to GitHub!

You can also run individual steps for download and load:

``` {.bash linenums="1"}

# Download multiple datasets
./download.sh bgt brk brt osm dem nwb

# or single dataset
./download.sh brk

# Load downloaded data into DB (BRT not yet)
./etl-load.sh bgt
./etl-load.sh brk
./etl-load.sh brt
./etl-load.sh nwb
./etl-load.sh osm
./etl-load.sh dem-contours
./etl-load.sh map5 

# or only OSM
./etl-osm-all.sh

```

If all goes well, you can test if you can generate tiles with Mapnik (more below):

`./mapnik-render-cat.sh roads`

To force full NL preparation locally, set `export MAP_AREA="nl"`.

## 3. DEM
 
NB: this step is only required if you ever need to prepare DEM. Normally you download
the already prepared files for Hillshade and Contours via `./download.sh dem` (see above).
                                                          
To prepare entire NL you will need storage for about 600GB. If you only download the already prepared
(dem/output) files, about 60GB is needed.
 
### Download Ready files
Use this to download entire NL (hillshade and contours in 5m and 05m). No local preparation needed:

``` {.bash linenums="1"}
cd tools/dem
./dem-download.sh output
```

### Preparing: Download, shrink and fill holes

Use this to prepare small area mainly. 
Using `tools/dem-prepare.sh <muiden|utrecht|nl> 5m [05m]`. 

Full NL, all resolutions:


``` {.bash linenums="1"}

cd tools
./dem-prepare.sh nl 5m 05m
```

Takes about 40h. About 1370 DTM tiff files per resolution.
But we'll have clean TIFF input files. Only needed once. Hillshade and Contour are separate
steps using the result of this step.

Melding, sommige bestanden. Hopelijk alleen met afronding te maken.

```
Warning 1: The definition of geographic CRS EPSG:4289 got from 
GeoTIFF keys is not the same as the one from the EPSG registry, 
which may cause issues during reprojection operations. Set GTIFF_SRS_SOURCE 
configuration option to EPSG to use official parameters (overriding the ones from GeoTIFF keys), 
or to GEOKEYS to use custom values from GeoTIFF keys and drop the EPSG code.

```

For local, small area (muiden or utrecht)

``` {.bash linenums="1"}

cd tools
./dem-prepare.sh muiden 5m 05m
```

### Generate Hillshade


``` {.bash linenums="1"}

cd tools
./dem-hillshade.sh 5m 05m
```

Generates one GeoTIFF with b&w hillshade per resolution.
Start za 11 juni, 2022, 19:45. Ready  june 12, 04:08. About 8.5 hrs. 

### Generate Contour Lines

``` {.bash linenums="1"}

cd tools
./dem-contours.sh

```

## 4. Sea and Water Shapes (Sea Only)

**(NOT REQUIRED FOR LOCAL SYSTEM since data will be downloaded during OSM ETL)**

Need the generalized water polygons from osmdata.openstreetmap.de as SQL Dump files.
See `tools/etl/osm-prepare-sea.sh`.  Basically this sequence to prepare. Need GDAL `ogr2ogr`.

```
/bin/rm -rf *-water-polygons-split-3857* *.sql.gz > /dev/null 2>&1

export OGR_OPTS="--config PG_USE_COPY YES -lco GEOMETRY_NAME=geom -overwrite -a_srs EPSG:28992 -s_srs EPSG:3857 -t_srs EPSG:28992 -spat 296778 6514442 853487 7143686"

# 1. simplified-water-polygons
wget https://osmdata.openstreetmap.de/download/simplified-water-polygons-split-3857.zip
unzip simplified-water-polygons-split-3857.zip
export DUMP_FILE="sea-polygons-simplified-28992-nl.sql"
ogr2ogr -f PGDump ${DUMP_FILE} simplified-water-polygons-split-3857 ${OGR_OPTS}
gzip ${DUMP_FILE}

# 2. water-polygons
wget https://osmdata.openstreetmap.de/download/water-polygons-split-3857.zip
unzip water-polygons-split-3857.zip
export DUMP_FILE="sea-polygons-28992-nl.sql"
ogr2ogr -f PGDump ${DUMP_FILE} water-polygons-split-3857 ${OGR_OPTS}
gzip ${DUMP_FILE}

/bin/rm -rf *water-polygons-split-3857*


```

Uploaded to: https://data.nlextract.nl/osm/nl/. 
Will be imported into the `map5.water` table during `map5` ETL.

