# GenAI as a project tutor 
I used GenAI, more specifically, ChatGPT and GreenPT, for the following reasons: 
- as a faster search engine because my questions had multiple variables that would have been difficult to convert for a search engine, even with operators like AND.
- as a tutor - but not as an easy way to progress - that I asked for help when I didn't find an answer myself after 30-60min.  

## Used models
ChatGPT (until 2 Mai 2026): GPT-5.3 and GPT-5.5
https://chatgpt.com/ 

GreenPT (starting at 2 Mai 2026): Main
https://greenpt.com/ 

I switched services due to OpenAI's contract with the US Pentagon and because GreenPT is hosted on servers run by renewable energies. 

## Data protection
Neither the map data nor the raster data nor any other files were uploaded to ChatGPT or GreenPT. 

### Hallucinations (selection)
ChatGPT provided commands to handle osmconvert in Windows Command Prompt. The commands were wrong because they somehow mixed Windows and Linux commands.

GreenPT suggested the algorithm 'Split lines by points', as part of the QGIS Processing Toolbox. This algorithm doesn't exist. 

## ChatGPT initial chat prompt
You are a senior data analyst who has years of experience working with geospatial data. I will use this chat to ask you questions on how to retrieve and analyse geospatial data. I want you to use concise and precise answers with examples. My questions will concern, amongst others, the Overpass query language.

## GreenPT project prompt
Role
You are a senior data analyst with years of experience using the following languages or software: SQL (specifically SQLite and BigQuery), DB Browser, Python, Overpass Query Language, Visual Studio Code, Command Prompt, Windows PowerShell, Microsoft Excel, Power BI. 

Knowledge
You know how to analyse different data types such as geo-spatial data (EPSG-codes, longitude, latitudes, vertices), commerce data (orders, invoices) and marketing data (impressions, page views, open rates). 

Check
If you need more information to answer my tasks or questions, ask me a maximum of 3 questions first. Then I provide you the information. Then you proceed with your answer. 

Output
Always write your answers in a professional tone. Professional means that you give a balanced answer.
Always write your answers in English. 
Always write concise answers. Concise means that you state an information only once.
Always use examples to explain your answers. Only use text in those examples. 
Always provide the sources to your answers as hyperlinks.
Only use hyperlinks that are accessible on the internet.

## Usage examples
Below I only provide a selection of my questions passed to ChatGPT or GreenPT since only those are my own work. 

### Explain concepts
What are nodes, ways and relations? Specify your answer to open street map data. 

What is the EPSG code? Explain it to me concisely with one example. 

### Explain file types
What do files contain, respectively, that have the following formats: GEOjson, GPX, KML, OSM? 

### Understand the Open Street Map Overpass API
If the values of the key place are city or town or village or suburb: Are those key-value-pairs assigned to a node or to a way or to a relationship? 

### Understand the syntax of the Overpass Query Language (OQL)
What's the wildcard character for string values? Example: I want to search for names that include "Hbf". There might be multiple characters before or after. 

Can I change the out statement in such a way that only certain tags are printed? 

### Handle QGIS, a geographic information system software
The GDAL OSM driver leads to a problem: It stores multiple tags that I need later in one column named 'other tags'. This is due to how tags are formatted during the import of an osm file. The driver defaults to HSTORE formatting. How can I assign a column to each tag in my osm file? 

The EPSG code for my QGIS project is 25832, but the EPSG code of my imported osm file is 4326. When I change the EPSG code of my imported osm file, the data disappears from the map view. What happens? 

### Handle osmconvert, a program to format osm files
How are the files in osmconvert combined? Example: when I have two different bounding boxes that are adjacent to each other, are they combined correctly?

### Help when being stuck
Read all the following sentences. Then answer my question. I have a raster layer in QGIS. I have two layers with rail points and rail lines. Rail points can be signaling points or train stations. Now, I want to calculate this: start at point A. Point A is always the same. Run along one line of railways. So, QGIS uses the rail lines, not the shortest distance. Stop at max. 100km and output the name of the nearest railway point within max. 5km radius. Start again until all possibilities are part of the output. Output should be a new layer. How do I do this in QGIS? Explain it to me step by step by using Munich as point A.

### Help with error messages
I'm getting an error message on this statement. What I want to achieve: An OR logic for the last statement: 
So: ["tram"="no"] AND ( ["name"="Hauptbahnhof"] OR ["name"="Bahnhof"] ); # Original statement [timeout:25] [bbox:{{bbox}}]; node["public_transport"="station"] ["railway"="station"] ["train"="yes"] ["subway"="no"] ["tram"="no"] ( ["name"="Hauptbahnhof"]; ["name"="Bahnhof"] ); out geom;

### Help with project scope
How many GB of data can QGIS process at once? In this context: What about data formats like GEOjson, GPX, KML, OSM raw data?