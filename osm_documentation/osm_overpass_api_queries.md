# Documentation: Queries used in the OSM Overpass API
This document lists the queries which I used in the OpenStreetMap Overpass API to obtain the data listed in the folder mapdata/original. This data served as the basis to make calculations in the programme QGIS, a geographic information system software to analyse geospatial data. The result of these calculations are files in the folder mapdata. 

## Tile numbering
To reduce the volume of the data that I requested from the API, I divided the map - so the surroundings of Munich - into four tiles. The division moves clockwise like this:  
- **tile_1** = north west of Munich
- **tile_2** = north east of Munich
- **tile_3** = south east of Munich
- **tile_4** = south west of Munich

See tile_numbering.pdf for details. 

## Tile content
To make the analysis of the map data easier later on, I pulled data from one tile twice - once for the rail network and once for the hotels. That way I could establish separate layers in the software QGIS.

## QUERY TILE 1 - RAIL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_1 = Munich north west*/
[bbox:48.1333728, 10.5939383, 49.1833728, 11.6439383];

way["railway"="rail"];

/*added by auto repair*/
(._;>;);
/*end of auto repair*/

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 1 - HOTEL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_1 = Munich north west*/
[bbox:48.1333728, 10.5939383, 49.1833728, 11.6439383];

node["tourism"="hotel"];

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 2 - RAIL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_2 = Munich north east*/
[bbox:48.1833728, 11.5939383, 49.1833728, 12.5939383];

way["railway"="rail"];

/*added by auto repair*/
(._;>;);
/*end of auto repair*/

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 2 - HOTEL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_2 = Munich north east*/
[bbox:48.1833728, 11.5939383, 49.1833728, 12.5939383];

node["tourism"="hotel"];

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 3 - RAIL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_3 = Munich south east*/
[bbox:47.1833728, 11.5439383, 48.2333728, 12.5939383];

way["railway"="rail"];

/*added by auto repair*/
(._;>;);
/*end of auto repair*/

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 3 - HOTEL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_3 = Munich south east*/
[bbox:47.1833728, 11.5439383, 48.2333728, 12.5939383];

node["tourism"="hotel"];

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 4 - RAIL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_4 = Munich south west*/
[bbox:47.1833728, 10.5939383, 48.1833728, 11.5939383];

way["railway"="rail"];

/*added by auto repair*/
(._;>;);
/*end of auto repair*/

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;

## QUERY TILE 4 - HOTEL

/* Timeout not included, default 180 seconds.*/

/*coordinates for tile_4 = Munich south west*/
[bbox:47.1833728, 10.5939383, 48.1833728, 11.5939383];

node["tourism"="hotel"];

/* I didn't use the output statement 'out skel' although this would have resulted in a lighter file because this wouldn't have included data lables which I needed later for my analysis.*/
out geom;