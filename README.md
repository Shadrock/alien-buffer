# alien-buffer
Mapbox [tutorial for buffering with Turf](https://labs.mapbox.com/education/proximity-analysis/selecting-within-a-distance/).
to complete:
- finish readme (tutorial)
- restructure repo: tutorial points to Data
- Add content from `metadata.md` in camp-water-analysis folder to the end of the "data readme" so that the readme goes from finding/downloading data through to simplifying it.

# SPHERE mapping with OpenStreetMap, Mapbox, & Turf
Water distribution in refugee camps is a critical part of good camp management and is mandated by the [SPHERE standards](https://spherestandards.org/), which sets standards for humanitarian action to promote quality and accountability. The SPHERE handbook sets forth standards such as how much water the resident of a camp should have access to and a maximum distance that one should have to travel to access water: less than 500 meters ([see the handbook chapter on water here](https://handbook.spherestandards.org/en/sphere/#ch006_004)). A practical GIS analysis of water accessibility in a refugee camp, then, is ensuring that all portions of the camp are within 500 meters of a water distribution point of some sort.

In this exercise we're going to build an interactive map that allows the user to determine which points fall within a specified distance or radius by clicking a map. This type of analysis is also known as buffer analysis. The goal is to provide a tool that camp managers could use to identify parts of a camp that don't have adequate access to water. There are multiple ways to do this; we will be using data from OpenStreetMap (or "OSM"), Mapbox, and [turf.js](https://turfjs.org/). Our end map will allow a user to click anywhere on a map and find all the water distribution points within a 500 meter radius of the point clicked. To do this we will:
- Add data on water distribution points to a map.
- Create a search radius (buffer) around any clicked events
- Join water point data to buffered polygon and return ‘confirmed’ water points within the selected search radius.

What you will need to finish this tutorial:
- A code editor,
- an OpenStreetMap account,
- a Github account for hosting website and data, and
- a Mapbox account for styling basemaps.

## Expected Outputs for this week / what to submit:
Once you have completed this tutorial you will have a working web map. However, I would like you to think about ways the map could be changed to improve the goal of helping a user identify parts of a camp that do not have appropriate access to water and then make those changes to your map. You will submit a link to your Github repo and the working website that it hosts.

## Data from OSM
The data for this exercise come from OSM. The `DATAREADME.md` file in the data folder introduces the structure of OSM data, how to search OSM for data, downloading data, and then simplifying it. This is a tutorial in its own right that will give you the basic skills to the vast range of data available in OSM.
***ADD LINK.***

# Getting Started - Initializing our Map
Open a text editor and create a file called index.html. Set up the document by copying and pasting this template code below into your new HTML file.

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="utf-8" />
  <title>Alien Map</title>
  <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
  <meta name="viewport" content="initial-scale=1,maximum-scale=1,user-scalable=no" />
  <script src='https://api.mapbox.com/mapbox-gl-js/v1.11.1/mapbox-gl.js'></script>
  <link href='https://api.mapbox.com/mapbox-gl-js/v1.11.1/mapbox-gl.css' rel='stylesheet'/>
  <script src="https://cdn.jsdelivr.net/npm/@turf/turf@5/turf.min.js"></script>
  <style>
    body {
      margin: 0;
      padding: 0;
    }

    #map {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 100%;
    }
    #instructions {
      position: absolute;
      top: 20px;
      left: 20px;
      background-color: #fff;
      padding: 5px 20px 5px 20px;
      border-radius: 10px;
      font-family: Lato, sans-serif;
    }

  </style>
</head>

<body>
  <div id="map"></div>
  <div id="instructions">
    <h2>🚰 Instructions</h2>
    <p>Click anywhere to see water sites within a 500m radius.</p>
  </div>
  <script>
    // Data showing water points - replace the dummy URL with your own.
    var url = 'https://raw.githubusercontent.com/USERNAME/'


    mapboxgl.accessToken = 'Your Access Token Goes Here!';
    var map = new mapboxgl.Map({
      container: 'map',
      style: 'mapbox://styles/mapbox/satellite-v9', // imagery basemap - you may choose something else
      center: [36.3295, 32.2963], // starting position at Zaatari Camp
      zoom: 14, // starting zoom
    });

    map.on('load', function () {

      // Map layers go here!
    });

    //Click event goes here!

    //makeRadius function goes here!

    //spatialJoin function goes here!

  </script>

</body>

</html>
```
The variable `url` stores a link to the OSM data you created following the instructions in the data folder **(LINK)**. These data should be a feature collection with information about the location of each water point. The data also contains a field `type` which has one of three unique values: `Free_standing`, `Attached_to_WASH_centre`, or `Oxfam_water_storage`. When setting the the variable, be sure to point to the page containing *raw* data and *not* simply to the Github page that displays the data!  

The css `#instructions` defines the properties for the instruction pane on the map: feel free to play with the design.

### Add your Mapbox token:
Without an access token, the rest of the code will not work.‍ Find your access token on your Access tokens page or the main page you sign into your Mapbox account.

Copy and paste your access token into the code that reads:
```html
mapboxgl.accessToken = 'Your Access Token Goes Here!';
```
### Adding Layers to the Map
We need to add three layers to the map - one to display all the water points, one to store water points within the radius, and one for drawing the search radius itself. We will use `map.addLayer()` to add each layer to the map and to define how the data is styled. All the layers will be initialized inside of a load event to make sure that the map has finished loading before the layers are added.

First, add a layer that displays all the water points. Remember this goes inside of the 'load' event, underneath where it's marked `//Map layers go in here`.

```javascript
//Map layers go in here!
map.addLayer({
  id: 'OSM-water',
  source: {
    type: 'geojson',
    data: url
  },
  type: 'circle',
  paint: {
    'circle-color': '#83e8fc',
    'circle-radius': 3,
    'circle-opacity': 1,
    'circle-stroke-width': 1,
    'circle-stroke-color': '#0557fa',
  }
});
```
There are a few things happening here. We've given this layer a unique name or `id`. In this case it's 'OSM-water'. We've set the `type` and `data` to be the URL that points to a geojson dataset. Then, we've symbolized the points. You may want to play with the design choices made. For more documentation about the various properties you can use to symbolize point data, see [the Mapbox documentation here](https://docs.mapbox.com/mapbox-gl-js/style-spec/layers/#circle).  

At this point, your map should look something like this:
![screenshot of initialized map](images/Initalized_map_data.png)

Next, add a layer to store the information on confirmed water points (those within the selected radius). This layer’s source data is set to an empty feature collection (`[]`). When a user clicks on a region, any confirmed water points will be added to this layer. For this layer, set the layer `type` to `symbol` and use the Mapbox Maki `water-15` icon to display any confirmed sightings. Maki icons are a handy design element available in Mapbox: [see them all and the documentation here](https://labs.mapbox.com/maki-icons/). Remember this all goes inside of the 'load' event.

```javascript
// When the map has finished loading, add a new layer that will be empty
// at first, but will eventually house our confirmed water sites.
map.addLayer({
  id: 'selected-water',
  source: {
    type: 'geojson',
    data: { "type": "FeatureCollection", "features": [] }
  },
  type: 'symbol',
  layout: {
     'icon-image': 'water-15',
     'icon-size': 1,
     'icon-allow-overlap': true
     }
});
```
You may want to use different symbols or play with the design. Find out more about [icon (or "symbol") configurations here](https://docs.mapbox.com/mapbox-gl-js/style-spec/layers/#symbol).

Next, add a layer to draw a search radius on the map (still within the load event). This layer’s source data is also set to an empty feature collection but will soon be filled with information about our search radius. Remember this goes inside of the 'load' event.
```javascript
// Draw the alien search radius on the map
map.addLayer({
  id: 'search-radius',
  source: {
    type: 'geojson',
    data: { "type": "FeatureCollection", "features": [] }
  },
  type: 'fill',
  paint: {
    'fill-color': '#fcf683',
    'fill-opacity': 0.5
  }
});

```
Depending on what basemap and data symbology you've chosen, you may want to change the `paint` specifications above for the color, opacity, or other characteristics of the buffer circle. For more information on paint properties you can use see [the Mapbox documentation](https://docs.mapbox.com/help/glossary/layout-paint-property/#paint-properties). 

### Add a "Click Event"
We need to create search radius around any point on the map that the user clicks. To do this, we need the coordinate information for each click event. This will go under the comment code, ```//Click event goes here!``` Create a click event and store the information about the clicked coordinates in a variable called `eventLngLat`. `Console.log(eventLngLat)` to view the clicked coordinates in your browser. Find out more about different click events in the [Mapbox documentation here](https://docs.mapbox.com/mapbox-gl-js/api/events/). This event should go outside of the load event.

```javascript
map.on('click', function(e) {
      var eventLngLat = [e.lngLat.lng, e.lngLat.lat];
      console.log(eventLngLat)
    });
```

After doing this, comment out `console.log(eventLngLat)` when finished.
**??**

### Create a Buffer with Turf
Now we will use the turf.js library to transform our event coordinates into point features and to create a buffer polygon around each point. This will go under the comment code ``` //makeRadius function goes here!```.

Initialize a function called `makeRadius()` and give it two parameters: `lngLatArray` and `radiusInMeters`, like so:

```javascript
function makeRadius(lngLatArray, radiusInMeters){}
```
Before we create a buffer, we need to turn our clicked location coordinates into a point feature. *Inside* of the function add a new variable called `point` and set it to `turf.point(lngLatArray)`, like so:
```javascript
var point = turf.point(lngLatArray)
```
Now that our coordinates are stored as a point feature, we can use turf's buffer function to make a polygon around the clicked point. Add the new buffered variable below the point variable and return it with the following code.
```javascript
var buffered = turf.buffer(point, radiusInMeters, { units: 'meters' });
  return buffered;
```
Once you've done all of the above, the complete code for the `makeRadius()` function should look like:
```javascript
function makeRadius(lngLatArray, radiusInMeters) {
    var point = turf.point(lngLatArray);
    var buffered = turf.buffer(point, radiusInMeters, { units: 'meters' });
    return buffered;
  }
```
### Call the makeRadius() function
Use the `makeRadius()` function to generate the search radius as a GeoJSON polygon by calling it *within your 'click' event*. This means it will go *under* `var eventLngLat = [e.lngLat.lng, e.lngLat.lat];`. This is also where you commented out ```console.log(eventLngLat)```

The function takes two arguments: a longitude and latitude array (`eventLngLat`) and a search radius. We'll use SPHERE's define radius of 500 meters.
```javascript
var searchRadius = makeRadius(eventLngLat, 500);
```
### Set the Search-Radius Layer
Set the search-radius source layer to our newly made radius feature. Below `var = searchRadius` (in your click event), add the following:
```javascript
  map.getSource('search-radius').setData(searchRadius);
```
Save your code and open the HTML document in a browser. When you click on the map, it should look something like this.
![screenshot of map with radius](images/Map-buffer.png)

### Identify Water Points within Buffered Radius
In this step we are going to create a function that loops through all the features in our original GeoJSON data. Next, the function we will use `turf.booleanPointInPolygon` to filter features that are both inside the buffered region and are confirmed landing sites. This code will go under the comment ```//spatialJoin function goes here!```

```javascript
function spatialJoin(sourceGeoJSON, filterFeature) {
    // Loop through all the features in the source geojson and return the ones that
    // are inside the filter feature (buffered radius) and are confirmed landing sites
    var joined = sourceGeoJSON.features.filter(function (feature) {
      return turf.booleanPointInPolygon(feature, filterFeature) && feature.properties.type === 'Free_standing';
    });

    return joined;
  }
```
**`type` is part of key in data. code set for freestanding only**

### Call the spaptialjoin() Function
Within the click function, create a new variable called `featuresInBuffer` and assign it the value of `spatialJoin()`. Remember that the function takes two arguements: a GeoJSON and a filter feature. For this exercise, we are using the `searchRadius` as the filter feature.
```javascript
var featuresInBuffer = spatialJoin(url, searchRadius);
```
**points to `url`, could be OSM-water**

### Set the Identified Water Points Layer
Set the alien-truth source layer to our newly made featuresInBuffer variable. Below `var = featuresInBuffer`, add the following:
```javascript
map.getSource('alien-truth').setData(turf.featureCollection(featuresInBuffer));
```
***set for `selected-water`???***

# Citations & Further Info
This tutorial is based on Mapbox's "[Selecting within a distance (one to many)](https://labs.mapbox.com/education/proximity-analysis/selecting-within-a-distance/)" tutorial. I've changed the use-case from finding alien landing sites to using real-world humanitarian data about water availability in refugee camps. I've also changed the code from using in-line GeoJSON to referencing data via a URL.
