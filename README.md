Exploring GeoJSON File and Interactive Mapping
================

### Introduction

geoJSON is a file format that d3 can use for generating maps. Tutorials and information about how the current version of d3 operates with geoJSON files are scattered. This guide is intended to piece together useful information I found from my initial attempt at interactive mapping.

### geoJSON

geoJSON is a file format that includes latitude/longitude coordinates for mapping purposes. My focus was on mapping areas in NYC. The NYC government has a useful open data website (linked below) that provides data on the different NYC partitions (e.g. election districts, fire divisions, etc.)

http://www1.nyc.gov/site/planning/data-maps/open-data/districts-download-metadata.page

This guide uses the subdivisions of police precincts in NYC. The link to the geoJSON file can be found below.

http://services5.arcgis.com/GfwWNkhOj9bNBqoJ/arcgis/rest/services/nypp/FeatureServer/0/query?where=1=1&outFields=*&outSR=4326&f=geojson

### Coding in d3

**Part A: Importing and Viewing JSON code in the console**

In d3, write import the data within a script.

``` js
var query = "http://services5.arcgis.com/GfwWNkhOj9bNBqoJ/arcgis/rest/services/nypp/FeatureServer/0/query?where=1=1&outFields=*&outSR=4326&f=geojson"

d3.queue()
  .defer(d3.json, query)
 .await(ready)
```
The ready function is called, which tells d3 what to do with the query. First, store the query data into the console./

``` js
function ready (error, data) {
  console.log(data)
}
```

**Part B: Look into the console data to determine the object names to be called**

After looking into the console, you must find the variable name of the data array that contains the geography data. In this case, the object name is "features". The code below stores the objects into a variable and saves it onto the log.

``` js
function ready (error, data) {
  console.log(data)
  
  var features = data.features;
  console.log(features)
}
```

**Part C: Create a style guide for the different geographic objects that you will create**

In this visualization, I am creating a black area object for each precinct. Since this is the default, there is no need to specify fill. However, on the hover mouse event, I want to make the area red. This can be done in the style section with the following technique:

``` js
<style>
  .areas: :hover {
    fill: red;
  }
</style>
```
Next, I am creating a white border object within the same style tags.

``` js
.borders {
  fill: none;
  stroke: #fff;
  stroke-width: 0.5px;
  stroke-linejoin: round;
  stroke-linecap: round;
  pointer-events: none;
}
```
Finally, I am creating a tooltip that will create a bubble displaying the specific area data from the geoJSON file.

``` js
div.tooltip {
    position: absolute;     
    text-align: center;     
    width: 60px;          
    height: 28px;         
    padding: 2px;       
    font: 12px sans-serif;    
    background: lightsteelblue; 
    border: 0px;    
    border-radius: 8px;     
    pointer-events: none;     
}
```

**Part D: Save useful variables to be used on the data**

In the script, but outside the ready function, initialize variables that will be used for the visualization.

``` js
var svg = d3.select("svg");
```

Create a tooltip called "div" with initial opacity of 0.

``` js
var div = d3.select("body")
  .append("div")
    .attr("class", "tooltip")       
    .style("opacity", 0);
```

Create a projection that maps latitude/longitude coordinates to the world. Note, there are many projections that can be used. The projection chosen is a good choice for maps more granular than full United States. Further reading can be found here:
https://github.com/d3/d3-3.x-api-reference/blob/master/Geo-Projections.md


``` js
var projection = d3.geoTransverseMercator()
    .scale(60000)
    .rotate([73.8, -40.7])
    .translate([500, 250]);
    
var path = d3.geoPath()
    .projection(projection);
```

**Part E: Within the ready function, append the location data onto the map and apply mouseover functions**

Note: Comments for each command can be found in the "Mapping NYC Precincts.html" file in the data repository.

Draw the precincts and add mouse over events. Add this to the ready function.

``` js
svg.append("g")
      .attr("class", "areas")
    .selectAll("path")
    .data(features)
    .enter().append("path")
      .attr("d",path)
    .on("mouseover", function(d) {
      div.style("opacity", .9);
      div.text(d.properties.Precinct)
         .style("left", (d3.event.pageX) + "px")
         .style("top", (d3.event.pageY - 28) + "px");
    })
    .on("mouseout", function(d) {div.style("opacity", 0);});
```

Draw the precinct borders. Add this to the ready function.

``` js
svg.append("path")
  .attr("d",path(data, features, function(a, b) { return a !== b; }))
  .attr("class", "borders"); 
```

### Summary

Mapping with geoJSON is simple in concept, but difficult because information is not centralized. This guide demonstrates concepts I gathered that allowed me to successfully code an interactive map of NYC. The resources I used can be found below. However, if you have any questions about my findings, I am happy to help.

### Resources
https://bl.ocks.org/mbostock/4090848
https://github.com/d3/d3-3.x-api-reference/blob/master/Geo-Projections.md
http://bl.ocks.org/d3noob/a22c42db65eb00d4e369
