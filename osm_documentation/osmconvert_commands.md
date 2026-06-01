# Documentation: Commands to use osmconvert
This document describes the commands that I used in Windows PowerShell to handle osmconvert which "can be used to convert and process OpenStreetMap files." [1](https://github.com/annika-essmann/day-trip-munich/blob/main/osm_documentation/osmconvert_commands.md#sources) It doesn't have a real interface and is thus handled in the command line. 

I utilised osmconvert to merge the following files which contain the original data that I pulled from the OpenStreetMap Overpass API: 
- tile1_rail.osm
- tile2_rail.osm 
- tile3_rail.osm 
- tile4_rail.osm<br>
--> all_rail.osm

- tile1_hotel.osm 
- tile2_hotel.osm 
- tile3_hotel.osm 
- tile4_hotel.osm<br>
--> all_hotel.osm

The result of these merges are stored in the folder [mapdata/original](mapdata/original). 

## Context: Tile numbering
I divided the map - so the surroundings of Munich - into four tiles. The division moves clockwise like this:  
- **tile_1**: north west of Munich `[bbox:48.1333728, 10.5939383, 49.1833728, 11.6439383]`
- **tile_2**: north east of Munich `[bbox:48.1833728, 11.5939383, 49.1833728, 12.5939383]`
- **tile_3**: south east of Munich `[bbox:47.1833728, 11.5439383, 48.2333728, 12.5939383]`
- **tile_4**: south west of Munich `[bbox: 47.1833728, 10.5939383, 48.1833728, 11.5939383]`

See [tile_numbering.pdf](tile_numbering.pdf) for details. 

## Context: Tile content
To make the analysis of the map data easier later on, I pulled data from one tile twice - once for the rail network and once for the hotels. That way I could establish separate layers in the software QGIS.

## Command: Merge four map tiles of a railway network into one file

`D:\MyProject>osmconvert tile1_rail.osm tile2_rail.osm tile3_rail.osm tile4_rail.osm --merge-versions -o=all_rail.osm`

## Command: Merge four map tiles of hotels into one file

`D:\MyProject>osmconvert tile1_hotel.osm tile2_hotel.osm tile3_hotel.osm tile4_hotel.osm --merge-versions -o=all_hotel.osm`

## Sources
[1]
https://wiki.openstreetmap.org/wiki/Osmconvert