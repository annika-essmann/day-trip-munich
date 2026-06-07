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

## Context
I started this project without having ever worked with geospatial data before. Neither did I know *any* of the tools that I used (see section [Applied tools](https://github.com/annika-essmann/day-trip-munich/blob/main/ReadMe.md#applied-tools)).

Thus, this demonstrates how I handle a task that I don't fully understand and that doesn't come with an explanation. In other words, I'm a strong autodidact who completed this project in roughly 60 hours. 

## Starting point
This project is inspired by the website https://www.chronotrains.com which allows to search for all possible destinations that can be reached within a desired travel time from a chosen origin.

However, this website mostly features bigger cities as destinations, for example it only suggests Augsburg as potential destination starting in Munich and travelling for one hour. Though, there are other interesting villages that meet these criteria.

So, I asked myself: If I wanted to spend one hour on the train: Which destinations can I reach from Munich? This question was the starting point of my work.

**Parameters**:<br> 
- **Origin**: Munich central station because that's where I live
- **Travel time**: max. one hour
- **Travel speed**: 80km/h; because it is more likely to take a regional train in this scenario which travels on average between 70-90km/h [[1]](https://github.com/annika-essmann/day-trip-munich/blob/main/ReadMe.md#sources)
- **Destinations**: must have at least one hotel; because it's not only an accomodation option, but also a proxy for tourist attractions

## Applied tools
I used the following API, languages and software to complete this project: 
- **OpenStreetMap Overpass API**: to access the geo-spatial data needed
- **Overpass Query Language (OQL)**: to write the query that pulls the data from the API
- **osmconvert, a programme to edit OpenStreetMap data**: to merge the pulled data into one file
- **Windows PowerShell**: to handle osmconvert because it doesn't have a graphical interface
- **QGIS, a geographic information system software**: to analyse and visualise the data
- **Structured Query Language (SQL)**: to filter the data in QGIS
- **git**: to track changes and upload the project to GitHub
- **mark down**: to write a well-layouted ReadMe file
- **Visual Studio Code**: to handle git and edit mark down files

## Steps taken and challenges solved 
Summary here.

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