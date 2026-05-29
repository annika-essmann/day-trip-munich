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
Inspo
https://www.chronotrains.com

Inspiriert vom Serviceportal der DB: https://www.bahn.de/angebot/urlaub/bahnreisen/summerrail 


-	Max. 8h per Zug ab München (Reisezeit per Zug, nicht Autofahrzeit) 
-	Max. X Umstiege (?)
-	Reiseziel muss Bahnhof haben
-	Reiseziel muss groß genug sein für Unterkünfte z.B. auf Airbnb oder Booking
-	Reiseziel muss groß genug sein für Sehenswürdigkeiten z.B. X Einträge auf TripAdvisor

## Narrowing down and question
From 8h on the train to 1h on the train
why? data volume, data handling, data access of raster data

Zusammengenommen kommen die 4 Kartensegmente auf 47MB. Diese 4 Kartensegmente bilden aber nur ca. 100km in jede Himmelsrichtung ab. Für 800km in jede Himmelsrichtung wäre das Datenvolumen also nochmal deutlich höher. 
- 8 tiles in jede Himmelsrichtung
- also der tile 1 oben links (north east) würde 8x8 = 64 tiles beinhalten
- das mal 4 = 256 tiles
- das mal 10 MB = 2.560 MB (2,56 GB)
- too much for QGIS to handle when in a text-based format
- too many tiles to handle for me

Aber: Die Overpass API bietet nur textbasierte Dateiformate als Export an, z.B. ist das Dateiformat .osm im XML Format. Das ist problematisch, weil Anwendungen wie QGIS mit mehr als 1-2GB im XML Format sehr langsam sind in der Verarbeitung. Daher ist eine Konvertierung in ein binäres Format von Vorteil, z.B. mit osmconvert. 
Im binären Format kann QGIS auch größere Datenmengen verarbeiten. 




## Methods: steps, tools and challenges solved 
e.g. SQL query that does xyz
which algorithm and why

### Segment map
Die Datenmenge, die ich von der API abfrage für den gesamten Kartenbereich, ist enorm. Was mache ich jetzt? 
Die Karte in Segmente unterteilen. Lösung. Ich habe 4 Segmente angelegt mit jeweils 1° Unterschied, das entspricht ganz, ganz grob 100km in jede Himmelsrichtung. 1° lässt sich gut addieren in jede Himmelsrichtung. 
Wichtig: bei den Segmenten musste ich einen Overlap von 0.05° einbauen, damit später bei der Zusammenführung keine nodes oder ways fehlen bzw. broken sind. Duplikate sind hier nicht schlimm. Programme wie osmconvert entfernen die Duplikate, da sie doppelte IDs aussortieren. Jede Node und jeder Way haben eine einzigartige ID.

### Check geometry – Duplicated Geometry 
Das ist wichtig, weil ich einen Overlap für Tile 1 und Tile 3 angelegt habe: 
Tile 1: +0,05° nach Osten und Süden
Tile 3: +0.05° nach Norden und Westen
Dadurch entstand ein Overlap für Tiles 2 und 4. 
Ich habe die vier Tiles dann zusammengeführt mit OsmConvert. Üblicherweise entfernt OsmConvert alle Duplikate. Duplikate sind jene, bei denen der unique identifier hier „id“ bzw. „osm_id“ gleich ist. 
Die merged files werden in QGIS in mehreren Layers importiert. Für jeden einzelnen Layer habe ich den Algorithmus laufen lassen. 
Ergebnis für Rails und Hotel: keine Duplikate, nicht in der Karte sichtbar, double check jeweils in den attribute tables: keine values

### Data cleaning: convert API export to correct EPSG code
Die Exports aus der Overpass API werden in QGIS mit dem EPSG-Code 4326 reingeladen – weil die bounding boxes in lat/lon angegeben waren. Die Rasterkarten sind aber mit einem EPSG-Code 25832 angegeben für den UTM Bereich, in dem sich auch Deutschland befindet. Hier wird in Metern gerechnet. Die Layer musste ich permanent auf EPSG-Code 25832 setzen, damit später bei der Berechnung der Distanzen keine Fehler auftreten. 
-	In order to project the layers, I had to export them as new gpkg files 
- that's how the change happened from the original .osm files to the .gpkg files finally used in the QGIS file result_destinations.qgz

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