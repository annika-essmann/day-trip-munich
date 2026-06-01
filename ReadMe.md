# Data exploration of map data to find potential travel destinations from Munich

## Executive summary
In my project I explored which cities can be reached from Munich within one hour and which have at least one hotel. 

To answer this question, I first pulled data from the OpenStreetMap Overpass API. Specifically, I requested information on the rail network, train stations and hotels around Munich. 

Then, I analysed the data in the geographic information system software QGIS by employing its algorithms to narrow down the rail network and train stations. I also used SQL WHERE statements to filter my data.

My exploration resulted in 21 destinations that I could travel to, for example Baar-Ebenhausen, Landshut (Bay) and Mühldorf (Oberbay). Since I limited my analysis by only looking at hotels, and thus disregarding other accomodation types, the amount of cities could be enlarged by adding further types such as guest houses or hostels.

![A screenshot of a map displaying a rail network around Munich and train stations at the end of each rail line](https://github.com/annika-essmann/day-trip-munich/blob/76c854d3bd8ac14b2a21243ff0587e2042114f26/result_destinations.png)
**Map data**: [OpenStreetMap](https://www.openstreetmap.org/copyright)<br> 
**Raster**: © GeoBasis-DE / [BKG](https://www.bkg.bund.de/) (2026) [dl-de/by-2-0](https://www.govdata.de/dl-de/by-2-0) (Daten verändert)<br> 
**Details**: [credits.md](credits.md)

## Context and gained skills
I started this project without having ever worked with geospatial data, the Overpass Query Language, or the software QGIS before. Thus, this demonstrates how I handle a task that I don't fully understand and that doesn't come with an explanation. In other words, I'm a strong autodidact who completed this project in roughly 60 hours. 

## Starting point
This project is inspired by the website https://www.chronotrains.com which allows to search for all possible destinations that can be reached within a desired travel time from a chosen origin.

However, this website mostly features bigger cities as destinations, for example it only suggests Augsburg as potential destination starting in Munich and travelling for one hour. Though, there are other interesting villages that meet these criteria.

So, I asked myself: If I wanted to spend one hour on the train: Which destinations can I reach from Munich? This question was the starting point of my work.

**Parameters**:<br> 
- **Origin**: Munich central station because that's where I live
- **Travel time**: max. one hour
- **Travel speed**: 80km/h; because it is more likely to take a regional train in this scenario which travels on average between 70-90km/h [[1]](https://github.com/annika-essmann/day-trip-munich/blob/main/ReadMe.md#sources)
- **Destinations**: must have at least one hotel; because it's not only an accomodation option, but also a proxy for tourist attractions

## Applied Tools
I used the following API, languages and software to complete this project: 
- **OpenStreetMap Overpass API**: to access the geo-spatial data needed
- **Overpass Query Language (OQL)**: to write the query that pulls the data from the API
- **osmconvert, a programme to edit OpenStreetMap data**: to merge the pulled data into one file
- **Windows PowerShell**: to handle osmconvert because it doesn't have a graphical interface
- **QGIS, a geographic information system software**: to analyse and visualise the data
- **Structured Query Language (SQL)**: to filter the data in QGIS
- **git**: to track changes and upload the project to GitHub
- **Visual Studio Code**: to handle git and edit mark down files

## Steps taken and challenges solved 

### Define map area
My starting point is Munich, and I want to potentially travel into each direction (north, east, south, west) by 80km. To make the map area large enough, I rounded the distance up to 100km which is approx. equivalent to 1° latitude and 1° longitude (disregarding the Earth's curvature in this case). 

So, if I take Munich as the focal point of the map, then I have the following coordinates in latitude and longitude:
- **Munich**: 48.1833728, 11.5939383
- **Coordinates for the entire map**: 47.1833728, 10.5939383, 49.1833728, 12.5939383 (south, west, north, east)

I used latitude and longitude because the OpenStreetMap Overpass API uses a bounding box to fetch data that lies within this box.

### Challenge: Segment map
**Problem**: The data that I wanted to pull from the API, using the coordinates above as the bounding box, is quite large: 20MB.

**Solution**: I divided the map into the following four segments: 
- **tile_1**: north west of Munich
- **tile_2**: north east of Munich
- **tile_3**: south east of Munich
- **tile_4**: south west of Munich

I also added an overlap of +/- 0.05° for tiles 1 and 3, so that there is a small amount of redundant data. This makes the merge of the files easier later on. 

See a visualisation of the segmentation in [tile_numbering.pdf](tile_numbering.pdf)

### Pull map data
To make the analysis of the map data easier later on, I pulled data from one tile twice - once for the rail data and once for the hotels. 

The documentation of the OQL queries can be found in [osm_overpass_api_queries.md](osm_documentation/osm_overpass_api_queries.md)

The data that I pulled for each tile can be found in [mapdata/original](mapdata/original)

### Combine map tiles
By using osmconvert, I merged the data of the four tiles together. The result are two files, the file [all_rail.osm](mapdata/original/all_rail.osm) contains the rail network and the train stations. The file [all_hotel.osm](mapdata/original/all_hotel.osm) contains the hotels. 

In the following, I refer to the data of these files as map data. The documentation of the applied commands can be found in [osmconvert_commands.md](osm_documentation/osmconvert_commands.md)

### Challenge: Show all necessary tags
**Problem**: I imported the map data into QGIS, but the data isn't read correctly. A lot of tags that I need are listed together in the column other_tags. Though, I need some of the tags in this column for filtering the map data.

**Reason**: The OSM driver in QGIS uses a certain way to format the tags.

**Solution**: I changed the settings how an *osm* file is imported directly in my local osmconf.ini file. For example, I added the tags "railway" (values: stop or signal) and "train" (values: yes or no) as standard columns. The explanation in the section [Filter only for train stations](https://github.com/annika-essmann/day-trip-munich/blob/main/ReadMe.md#filter-only-for-train-stations) shows why this is relevant.

### Check for duplicates 
I ran the algorithm 'Check Geometry - Duplicated Geometry' from the QGIS Processing Toolbox. It reports any duplicated geometries in a vector layer as errors.

**Why?** I used an overlap for tile 1 and 3 in the original map data, and the software osmconvert normally eliminates all duplicates when merging files. With this algorithm I verified that. 

### Reproject map data to correct EPSG code
QGIS imported the map data with the EPSG code 4326 which measures distances in latitude and longitude. However, the raster data I used has the EPSG code 25832 which measures distances in meters. To align the map data, I reprojected it to the EPSG code 4326 in QGIS which means I exported it as a new file with the correct code.

### Filter only for train stations
QGIS displays the rail data as a line layer (the network) and a point layer (the stations). However, the point layer also includes all signaling posts. To remove this noise, I filtered the layer with the SQL WHERE statement:<br>
`“railway” IN(‘stop’) AND “train“ IN(‘yes’)` 

### Snap geometries to layer
I used the algorithm 'Vector Geometry - Snap Geometries to Layer' from the QGIS Processing Toolbox. This snaps the rail points to the rail lines.

**Why?** Later, I want to use the rail data to make multiple calculations, e.g. run along the network to a certain station. This step prevents any errors, e.g. a station is not connected to the network.

### Create the file rail_start.gpkg
The following paragraphs describe the steps I undertook to create the file [rail_start.gpkg](mapdata/rail_start.gpkg) which is saved in the folder [mapdata](mapdata) and used as a layer in the QGIS project [result_destinations.qgz](result_destinations.qgz).

I copied the layer containing the train stations and filtered it for Munich central station by using the following SQL WHERE statement:<br>
`"name" LIKE 'München Hauptbahnhof%' OR "name" LIKE 'München Hbf%'`

**Why?** To run the algorithm 'Network Analysis - Service Area (from Layer)', I need to define a starting point. This new layer serves as this starting point. It contains 33 stops in total, one for every platform, because the rail network is quite detailed and runs in several directions.

### Create the file rail_network.parquet
The following sub-sections describe the steps to create the file [rail_network.parquet](mapdata/rail_network.parquet) which is saved in the folder [mapdata](mapdata) and used as a layer in the QGIS project [result_destinations.qgz](result_destinations.qgz).

#### Map the reachable rail network
I used the algorithm 'Network Analysis - Service Area (from Layer)' which, metaphorically speaking, runs along a network and stops after a given time. 

**Parameters**<br>
- **Starting point**: rail_start.gpkg
- **Travel time**: max. one hour
- **Travel speed**: 80 km/h; regional trains travel between 70-90km/h [[1]](https://github.com/annika-essmann/day-trip-munich/blob/main/ReadMe.md#sources); for good measure I chose the middle

#### Clean up the file
The file was 159 MB large. To reduce the size, I deleted all unused columns and saved the file in the binary format parquet. 

### Create the file rail_end.gpkg
The following sub-sections describe the steps to create the file [rail_end.gpkg](mapdata/rail_end.gpkg) which is saved in the folder [mapdata](mapdata) and used as a layer in the QGIS project [result_destinations.qgz](result_destinations.qgz).

#### Add the closest train station
I applied the algorithm 'Vector General - Join Attributes by Nearest' to the layer containing all rail stations. The algorithm joins two layers together by finding the closest feature.

**Why?** The file rail_network.parquet only contains a network, not the train stations. Metaphorically speaking, when a train runs along this network it just stops after one hour; but there isn't necessarily a train station to get off. The algorithm 'Vector General - Join Attributes by Nearest' finds the next train station. 

**Parameters**<br>
- **Input 1**: layer containing all train stations
- **Input 2**: rail_network.parquet
- **Maximum distance**: 5km; to find the next train station within a 5 km radius of the rail network

#### Find rail stations with at least one hotel
I ran the algorithm 'Vector Selection - Extract within distance'. It creates a new layer that only contains features from the input layer that meet a certain condition.

**Why?** I want to find the rail stations where there is at least one hotel within a 5km radius. 

**Parameters**<br> 
- **Extract features from**: rail_network.parquet
- **Comparison with**: all_hotel.osm; the original data pulled from the API
- **Where the features are within**: 5km

#### Create layer with only one train station in Munich
I copied the file rail_start.gpkg - which contains 33 stations - and filtered for only one station by applying the following SQL WHERE statement:<br>
`"osm_id"=237680533`

**Why?** For the next algorithm, I need to have only one starting point - otherwise the algorithm doesn't work. 

#### Calculate the distance between rail end points and Munich
I used the algorithm 'Vector Analysis - Distance to Nearest Hub (Point)' which creates a new layer that contains, amongst others, a new column with the distance between two points.

**Why?** Later, I want to display the labels of the train stations at the end, but the current file contains all train stations. So, I want to filter for a certain distance to select the right train stations, and the algorithm supplies me with the information to do that.

**Parameters**<br> 
- **Source**: the layer that contains the train stations with at least one hotel within a 5km radius
- **Hub**: the layer with only one train station in Munich

**Info**: The column with the distance was added to the layer with the train stations. 

#### Insert labels for the rail end points
I filtered the layer with the train stations to display all stations that are more than 40km away from Munich by applying the following SQL WHERE statement:<br>
`"HubDist">40`

Then, I inserted the labels for the train stations, and manually wrote down each station at the end of each rail line.

**Why?** I didn't find a means to extract the rail end points because their distance from Munich varies quite a bit. So, I couldn't apply an algorithm like 'Vector Selection - Extract within Distance'. Filtering and manually selecting the train stations was more efficient given the small data set.

As a last step, I changed the filter to the following SQL WHERE statement:<br>
`"name" IN ('Baar-Ebenhausen', 'Landshut (Bay) Hbf', 'Mühldorf (Oberbay)',`
`'Jettenbach', 'Bad Endorf (Oberbay)', 'Flintsbach', 'Bayrischzell',` 
`'Tegernsee', 'Gaißach', 'Wolfratshausen', 'Kochel', 'Murnau',` 
`'Herrsching', 'Schongau', 'Landsberg (Lech)',`
`'Türkheim (Bayern)', 'Bobingen', 'Merching',`
`'Kutzenhausen', 'Langweid (Lech)', 'Augsburg Hauptbahnhof')`

#### Clean the file
The algorithm 'Vector General - Join Attributes by Nearest' created a lot of duplicates which I cleaned up by employing the algorithm 'Fix Geometry - Delete Duplicate Geometries'. I also manually deleted all rows with a duplicated osm_id which resulted in 21 rows - exactly the train stations I filtered for in the previous step.

### Create the file hotel_at_end.gpkg
The following paragraphs describe the steps to create the file [hotel_at_end.gpkg](mapdata/hotels_at_end.gpkg) which is saved in the folder [mapdata](mapdata) and used as a layer in the QGIS project [result_destinations.qgz](result_destinations.qgz).

I ran the algorithm 'Vector Selection - Extract within Distance' which creates a new layer that only contains features from the input layer that meet a certain condition.

**Why?** I want to obtain the hotels that are within a 5km radius of the rail end points. 

**Parameters**<br>
- **Extract features from**: all_hotels.osm; the original data from the API
- **By comparing to**: rail_end.gpkg
- **Where the features are within**: 5km

## GenAI usage
In this project, I used ChatGPT (GPT-5.3 and GPT-5.5) and GreenPT (Main) as a project tutor. Amongst others, the tool helped me to understand: 
- the syntax of the Overpass Query Language,
- new concepts like EPSG codes, 
- the software osmconvert and QGIS.

As a rule of thumb, when I received an error message or contemplated the next step, I researched myself for 30-60min before passing a question to ChatGPT or GreenPT.

See details in the file [genai_usage.md](genai_usage.md)

## Results 
I started my project with the question: Which (small) cities can I reach within one hour from Munich? The answer: I could travel to 21 different (small) cities.

![A screenshot of a map displaying a rail network around Munich and train stations at the end of each rail line](https://github.com/annika-essmann/day-trip-munich/blob/76c854d3bd8ac14b2a21243ff0587e2042114f26/result_destinations.png)
**Map data**: [OpenStreetMap](https://www.openstreetmap.org/copyright)<br> 
**Raster**: © GeoBasis-DE / [BKG](https://www.bkg.bund.de/) (2026) [dl-de/by-2-0](https://www.govdata.de/dl-de/by-2-0) (Daten verändert)<br> 
**Details**: [credits.md](credits.md)

## Limitations and further usage
To simplify my analysis, I only included hotels and disregarded other accomodation types like camping grounds, guest houses or hostels. Adding these could enlarge the amount of possible destinations.

Also, the analysis could be further divided into destinations for a day trip and a weekend trip. For the former, I don't need a hotel, but potential tourist attractions such as museums or hiking trails. This would certainly result in different destinations.

Since my project focused on data exploration, I didn't do a full deployment of my results, e.g. in an app like Streamlit. I regarded a list and an image as sufficient because results like these are usually used as an input for a deeper analysis that then merits a full deployment. 

Yet, these results can also stand on their own and could serve as an addition to the list displayed on https://www.chronotrains.com. Also, the list might be used for a service post on the website https://www.muenchen.de/en/home which also recommends travel destinations in Munich's vicinity. 

## Credits 
**Map data**<br> 
[OpenStreetMap](https://www.openstreetmap.org/copyright)

**Raster**<br>
© GeoBasis-DE / [BKG](https://www.bkg.bund.de/) (2026) [dl-de/by-2-0](https://www.govdata.de/dl-de/by-2-0) (Daten verändert)

See details in [credits.md](credits.md)

## Sources
[1] https://en.wikipedia.org/wiki/Regional-Express 