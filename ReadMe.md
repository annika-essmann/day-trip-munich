# strong titel here 

## Executive summary
business problem, solution, next steps, impact in numbers

This project demonstrates how...
-	Wie gehe ich mit Daten um, mit denen ich vorher noch nie gearbeitet habe? 
-	Wie gehe ich mit einer Programmiersprache um, die ich nicht kenne? 
-	Wie erschließe ich mir eine neue Software? 
-	Exploratory data analysis 
-	
-	Nächste Schritte: 
Ein Modell für muenchen.de 
Stunden einstellen und Tags nach Aktivität und Tags nach Land

Gained skills in this project: 
- ABC

Hours invested from idea to GitHub initial commit: 
XXX




## Starting point
This project is inspired by the website https://www.chronotrains.com which allows to search for all possible destinations that can be reached within a desired travel time from a chosen origin. However, this website mostly features bigger cities as destinations, and the filter only allows for a travel time between one and eight hours. One example: The website only shows Augsburg as potential destination starting in Munich and travelling for one hour. Though, there are other interesting villages that meet these criteria.

So, I asked myself: If I wanted to do a day or weekend trip for which I wanted to spend less than one hour on the train: Which destinations can I reach? This question was the starting point of my work.

Parameters: 
- *origin*: Munich central station because that's where I live
- *travel* time: max. one hour
- *travel speed*: 80km/h because it is more likely to take a regional train in this scenario which travels on average between 70km/h and 90km/h [1]
- *destinations*: must have a train station and must have at least one hotel in case I want to spend the night

## Methods: Tools
I used the following API, languages and software to complete this project: 
- *OpenStreetMap Overpass API*: to access the geo-spatial data needed
- *Overpass Query Language (OQL)*: to write the query that pulls the data from the API
- *osmconvert, a programme to edit OpenStreetMap data*: to merge the pulled data into one file
- *Windows PowerShell*: to handle osmconvert because it doesn't have a graphical interface
- *QGIS, a a geographic information system software*: to analyse and visualise the data
- *Structured Query Language (SQL)*: to filter the data in QGIS
- *git*: to track changes and upload the project to GitHub
- *Visual Studio Code*: to handle git and edit mark down files

## Methods: Steps taken and challenges solved 
In this section, I describe in detail and chronologically which methods I applied. 

### Step: Define map area
My starting point is Munich, and I want to potentially travel into each direction (north, east, south, west) by 80km. To make the map area large enough, I rounded the distance up to 100km which is approx. equivalent to 1° latitude and 1° longitude (disregarding the Earth's curvature in this case). So, if I take Munich as the focal point of the map, then I have the following coordinates in latitude and longitude: 
- *Munich*: 48.1833728, 11.5939383
- *Coordinates for the map*: 47.1833728, 10.5939383, 49.1833728, 12.5939383 (south, west, north, east)

I used latitude and longitude because they were an effective means to define the bounding box that the OpenStreetMap Overpass API uses to fetch the data that lies within the borders of this box.

### Challenge: Segment map
*Problem*: The data that I wanted to pull from the API, using the coordinates above as the bounding box, is quite large: 20MB. 
*Solution*: I divided the map into the following four segments: 
- tile_1 = north west of Munich
- tile_2 = north east of Munich
- tile_3 = south east of Munich
- tile_4 = south west of Munich
I also added an overlap of +/- 0.05° for tiles 1 and 3, so that there is a small amount of redundant data. This makes the merge of the files easier later on. 
See a visualisation in tile_numbering.pdf

### Step: Pull map data
To make the analysis of the map data easier later on, I pulled data from one tile twice - once for the rail network and once for the hotels. The documentation of the OQL queries can be found in osm_overpass_api_queries.md 

### Step: Combine map tiles
By using osmconvert, I merged the data of the four tiles together. The result are two files, the file all_rail.osm contains the rail network; the file all_hotel.osm contains the hotels. In the following, I refer to the data of these files as map data. The documentation of the applied commands can be found in osmconvert_commands.md

### Step: Check Geometry – Duplicated Geometry 
I imported the map data into QGIS. First, I ran the algorithm Check Geometry - Duplicated Geometry from the QGIS Processing Toolbox. It reports any duplicated geometries in a vector layer as errors. 
*Why?* I used an overlap for tile 1 and 3 in the original map data, and osmconvert normally eliminates all duplicates when merging files. With this algorithm I verified that. 

### Step: Reproject map data to correct EPSG code
QGIS imported the map data with the EPSG code 4326 which measures distances in latitude and longitude. However, the raster data I used has the EPSG code 25832 which measures distances in meters. To align the map data, I reprojected it to the EPSG code 4326 in QGIS which means I exported it as a new file with the correct code.

### Data cleaning: show all necessary tags
Einige Tags werden beim Import der osm Datei in QGIS nicht richtig gelesen. Mehrere Tags werden in eine Spalte ‚other_tags‘ zusammengelegt. Das liegt daran, wie der OSM Driver in QGIS arbeitet. Er benutzt HSTORE formatting, um Tags zu verarbeiten. Das ist problematisch, weil in der ‚other_tags‘ Spalte Tags stehen, die ich später brauche – vor allem die Name der Bahnhöfe, die nicht in diesem Tag stehen „name“=“xxx“, wie bei den Hotels, sondern manchmal unter „ref_name“ oder „official_name“. Dafür muss die osmconf.ini im C:\ Laufwerk geändert werden – also die Konfiguration, wie eine osm Datei importiert wird. 
-	C:\Program Files\QGIS 3.44.9\apps\gdal\share\gdal

### Filter
Ich habe zu viele rail points, die ganzen Signale sind mit dabei. Also habe ich in QGIS die point layer gefiltert mit “railway” IN(‘stop’) AND “train“ IN(‘yes’). Dadurch werden nur Bahnhöfe angezeigt, auch S-Bahnhöfe, aber das ist nicht so schlimm, weil die Distanz bei max. 100 km liegt – das ist außerhalb des S-Bahn-Einzugsgebiets.

### Snap geometries to layer
-	Algorithm Vector geometry - Snap geometries to layer
	Snap rail – points to rail – lines 
	Prefer aligning nodes, insert extra vertices where required
	Important: because I had a filter on the layer rail – points, the snapping was only done for the filtered points
	Output was a new layer with the pre-filtered snapped points
- Result: V2.4_rail_points_after_snap

### Creating the file rail_start

#### New layer with Munich as starting point
-	Copy V2.4_rail_points_after_snap
-	to define a new point layer only with München Hauptbahnhof
-	Filter for "name" LIKE 'München Hauptbahnhof%' OR "name" LIKE 'München Hbf%'
- result: V2.4_rail_points_after_snap_point_A_munich

### Creating the file rail_network

#### Service area
-	algorithm Network Analysis – Service Area (from layer) from QGIS processing toolbox
-	Vector layer representing network: rail – lines
-	Path type to calculate: fastest
-	Vector layer with start points: rail – points – after snap – point A Munich
-	Travel cost (time in hours): 1
-	Default direction: both directions
-	Default speed (km/h): 80 (because regional trains travel between 70-90km/h
-	Output in temporary layer: 
-	service area lines: path from point A Munich to point B 
-	service area (boundary nodes): every possible point B
- Result: V2.4_80_1_service_area_boundary_nodes / lines

#### Join layer by nearest
Why? The reachable network just stopped after 80km, not necessarily at a train station
to fix that: used this algo
downside: a lot of duplicates that I had to clean up later

-	Algorithm Vector General – Join Attributes by Nearest
-	Input layer 1: rail – points – after snap to nodes (because this contains the train stations)
-	Input layer 2: 80_1_Service area (lines) (because this is the reachable network by 80km/h for max. 1h, starting at München Hauptbahnhof
-	Layer 2 fields to copy: default – all 
-	Discard records (from input layer 1) which could not be joined (because I don’t want the train stations in my data set that can’t be reached) 
-	Joined field prefix: j_ (that means all column headers from input layer 2 have this prefix now)
-	Maximum nearest neighbors: default – 35
-	The algorithm didn’t work with the default 1 (probably because I’m looking for multiple nearest neighbors on each route)
-	Error message said: found 33 neighbors instead of 1
-	So I set the maximum to 35
-	Maximum distance: 5 km

### Creating the file rail_end

#### Extract within distance
Algorithm: Vector Selection – Extract within distance
-	Extract features from: V2.4_joined_layer… (because I want to create a new layer with these rail points under a certain condition)
-	By comparing to: V2.3_all_hotel_points (select by comparing to the hotel points)
-	Where the features are within: 5km (only select rail points from the joined_layer if there are hotel points within 5km range)
-	Resulting file, saved in V2\calculations
-	V2.5_joined_layer_hotels_5km

#### Only one starting point München tief
Copy of layer V2.4_rail_points_after_snap_point_A_munich
-	Filter only for one point München Hauptbahnhof (tief) 
-	"osm_id"=237680533
-	Why? So that I only have one starting point for the algorithm Vector analysis – Distance to nearest hub (point) 
-	Resulting layer: 
V2.4_rail_points_after_snap_point_A_munich_only (tief) 


#### Distance to nearest hub
Algorithm Vector analysis – Distance to nearest hub (point) 
-	Why? To get a column with distance in km from München Hauptbahnhof (tief) to all rail points in the layer V2.5_joined_layer_hotels_5km 
- Why? because I want to filter for this distance later, so that I only have the rail points at the end of the network left 

-	all railway stations with close hotels
-	Source points layer: V2.5_joined_layer_hotels_5km 
-	because the algorithm only searches for the nearest points when starting at the source points layer – but I’m looking at the furthest points from Munich
-	Destination hubs layer: V2.4_rail_points_after_snap_point_A_munich_only (tief)
-	Hub layer name attribute: osm_id
-	Measurement: km

#### Layer only with the rail points at the end
Copy and filter: V2.5_joined_layer_hotels_5km_distance 
-	First for HubDistance > 40 km
-	So that the view is easier to work with
-	Insert labels so that I know the names of the railway stations
-	Manually write down each station at the end of each railway line
-	Change filter to 
"name" IN ('Baar-Ebenhausen', 'Landshut (Bay) Hbf', 'Mühldorf (Oberbay)', 
'Jettenbach', 'Bad Endorf (Oberbay)', 'Flintsbach', 'Bayrischzell', 
'Tegernsee', 'Gaißach', 'Wolfratshausen', 'Kochel', 'Murnau', 
'Herrsching', 'Schongau', 'Landsberg (Lech)', 
'Türkheim (Bayern)', 'Bobingen', 'Merching', 
'Kutzenhausen', 'Langweid (Lech)', 'Augsburg Hauptbahnhof')

Algorithm Fix Geometry – Delete duplicate geometries
-	Input: V2.5_joined_layer_hotels_5km_distance_only_end
-	Resulting layer: 
V2.5_joined_layer_hotels_5km_distance_only_end_CLEAN

Edit V2.5_joined_layer_hotels_5km_distance_only_end_CLEAN
-	Go to the attribute table
-	Delete manually all duplicate names 
-	Result: only 21 railway stations


### Creating the file hotel_at_end

#### New hotel layer under a certain condition
Algorithm: Vector Selection – Extract within distance
-	Extract features from: V2.3_all_hotel_points (because I want to create a new layer, only with hotels under a certain condition)
-	By comparing to: V2.5_joined_layer_hotels_5km_distance_only_end_clean (because I only want hotels that are within range of these railway stations)
-	Where the features are within: 5km 

### GenAI usage
In this project, I used ChatGPT (GPT-5.3 and GPT-5.5) and GreenPT (Main) as a project tutor. The tool helped me to understand: 
- the syntax of the Overpass Query Language,
- new concepts like EPSG codes, 
- the software osmconvert and QGIS. 
As a rule of thumb, when I received an error message in a program or contemplated the next step in my analysis, I thought about it and researched myself for 30-60min before passing a question to the ChatGPT or GreenPT.

See details in the file genai_usage.md

## Results 

## Next steps 
address limitations, measure XYZ for more insights 

Business case: muenchen.de gibt auch Tipps für Ausflüge im Münchner Umland. Wie viele Destinationen gibt es? Wie lange dauert die Zugfahrt? Service: Liste der Städtenamen und mögliche Hotels. Basis für einen Service-Post.

Add a regional section here
https://www.chronotrains.com
https://www.bahn.de/angebot/urlaub/bahnreisen/summerrail 


-	Include other accomodation types, merge into one layer together with hotels
-	“tourism”=”alpine_hut”
-	“tourism”=”camp_site”
-	“tourism”=”caravan_site”
-	“tourism”=”chalet”
-	“tourism”=”guest_house”
-	“tourism”=”hostel”
-	“tourism”=”motel”
-	Include attractions to visit, as a new layer
-	“tourism”=gallery”
-	“tourism”=”museum”
-	“tourism”=”theme_park”
-	“tourism”=”viewpoint”
-	“tourism”=”zoo”

mark Streamlit as potential next steps, why not done: this is an exploration > list is sufficient for the start, conserving hours

## Credits 
Copy the current status of the credits.md file here

## Sources
[1] https://en.wikipedia.org/wiki/Regional-Express 