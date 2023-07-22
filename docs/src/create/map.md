---
title: Map Preparation
---

# Maps

Describes how to use the prepared data to create maps. 

![map5topo](../assets/images/map5topo-oosterbeek-2.jpg){ align=left }

## 1. Mapnik Test

Directly render tiles with Mapnik Python script. Output (JPEG) will be
under `tools/mapnik/output/`.


``` {.bash linenums="1"}

cd tools

render multiple tiles at different zooms per category
./mapnik-render-cat.sh  roads | resident | rural

Render any single tile by zoom, x, y
./mapnik-render-tile.sh  z x y

```

## 2. Tile Seeding

Create tile caches (GeoPackages).


``` {.bash linenums="1"}

cd services/mapproxy/seed
# Entire netherlands, NB takes extremely long!
./seed-rd.sh

# Muiden Area, takes about 30mins-1h depending on system
./seed-muiden-rd.sh

```

## 3. Test the Services

Several apps are available.

* Run service stack: `cd services; ./start.sh`
* Local: http://localhost:8000/mp (MapProxy) http://localhost:8000/app (apps) http://localhost:8000/pgadmin (pgadmin)
* Production: https://topo.map5.nl/mp https://topo.map5.nl/app
