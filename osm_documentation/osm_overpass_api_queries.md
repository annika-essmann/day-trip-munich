# Documentation: Queries used in the OSM Overpass API
This document lists the queries which I used in the OpenStreetMap Overpass API to obtain the data listed in the folder mapdata/original. This data served as the basis to make calculations in the programme QGIS, a geographic information system software to analyse geospatial data. The result of these calculations are files in the folder mapdata. 

## Tile numbering
To reduce the volume of the data that I requested from the API, I divided the map - so the surroundings of Munich - into four tiles. The division moves clockwise like this:  
- **tile_1**: north west of Munich: *[bbox:48.1333728, 10.5939383, 49.1833728, 11.6439383]*
- **tile_2**: north east of Munich: *[bbox:48.1833728, 11.5939383, 49.1833728, 12.5939383]*
- **tile_3**: south east of Munich: *[bbox:47.1833728, 11.5439383, 48.2333728, 12.5939383]*
- **tile_4**: south west of Munich: *[bbox: 47.1833728, 10.5939383, 48.1833728, 11.5939383]*

See [tile_numbering.pdf](tile_numbering.pdf) for details.

## Tile content
To make the analysis of the map data easier later on, I pulled data from one tile twice - once for the rail data and once for the hotels. That way I could establish separate layers in the software QGIS.

## QUERY TILE 1 - RAIL
The following code serves as an example for the *rail* data pulled from tiles 2-4. Only the bounding box (bbox) changed. The rest of the code stayed the same. 

`/* Timeout not included, default 180 seconds.*/`

`/*coordinates for tile_1 = Munich north west*/`<br>
`[bbox:48.1333728, 10.5939383, 49.1833728, 11.6439383];`

`way["railway"="rail"];`

`/*added by auto repair*/`<br>
`(._;>;);`<br>
`/*end of auto repair*/`

`/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/`<br>
`out geom;`

## QUERY TILE 1 - HOTEL
The following code serves as an example for the *hotel* data pulled from tiles 2-4. Only the bounding box (bbox) changed. The rest of the code stayed the same. 

`/* Timeout not included, default 180 seconds.*/`

`/*coordinates for tile_1 = Munich north west*/`<br>
`[bbox:48.1333728, 10.5939383, 49.1833728, 11.6439383];`

`node["tourism"="hotel"];`

`/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/`<br>
`out geom;`