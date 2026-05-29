# Documentation: Commands used in osmconvert
This document describes the commands that I used in Windows PowerShell to use osmconvert which "can be used to convert and process OpenStreetMap files." [1] It doesn't have a real interface and is thus handled in the command line. The result of these two merges are stored in the folder mapdata/original.

## Tile numbering
I divided the map - so the surroundings of Munich - into four tiles. The division moves clockwise like this:  
- tile_1 = north west of Munich
- tile_2 = north east of Munich
- tile_3 = south east of Munich
- tile_4 = south west of Munich
See tile_numbering.pdf for details. 

## Tile content
To make the analysis of the map data easier later on, I pulled data from one tile twice - once for the rail network and once for the hotels. That way I could establish separate layers in the software QGIS.

## Merge four map tiles of a railway network into one file

D:\MyProject>osmconvert V2_tile1_rail.osm V2_tile2_rail.osm V2_tile3_rail.osm V2_tile4_rail.osm --merge-versions -o=all_rail.osm

## Merge four map tiles of hotels into one file

D:\MyProject>osmconvert V2_tile1_hotel.osm V2_tile2_hotel.osm V2_tile3_hotel.osm V2_tile4_hotel.osm --merge-versions -o=all_hotel.osm

## Sources
[1]
https://wiki.openstreetmap.org/wiki/Osmconvert
