# An Application to Discover and Map Heat Islands
###### Andrew Renninger

# CONTENTS
- [INSPIRATION](#INSPIRATION)
- [EXPLORATION](#EXPLORATION)
- [INVESTIGATION](#INVESTIGATION)
- [EXPECTATION](#EXPECTATION)
- [EXPLANATION](#EXPLANATION)

# INSPIRATION
The goal of this project is to create a process by which we can understand urban heat islands—the phenomenon wherein built and natural conditions conspire to raise temperature in urban areas during periods of hot weather—with readily accessibly data from satellite images. During a spell of hot weather in late July 2019, mercury levels in Philadelphia remained above 90 degrees Fahrenheit during the day and even held above 80 over night. Yet this swelter was not felt equally across the city: the hottest and coolest—comparably—portions of the city differed by as much as 20 degrees. A park could be bearable; a parking lot could be insufferable. Each neighborhood sees unique effects from sun and heat and each season is different. Because of this, the following project is an application that can be used by all. While this application represents a high cost to build, it yields a low cost for experimentation. The subsequent exploration is modest, but folding new cities and layering new methods only becomes easier with such an interface. Additionally, because it lacks advanced statistical analysis, forgoing the statistical for the graphical, it serves as a means to travel the globe by satellite—more qualitatively than quantitatively and with breadth rather than depth, although by allowing for such travel is by no means trivial. To explore the geography of heat with this tool, use [this link](https://asrenninger.users.earthengine.app/view/heatwave).          

###### Temperature in the United States
![](https://github.com/asrenninger/heat/raw/master/temp.gif?raw=true)

*Mapping surface temperature throughout the year in the united states reveals that at times urban areas—like the Acela Corridor—are warmer that regions and comparable latitudes (see: code below to reproduce).*

Remote sensing technology provides robust spatio-temporal data ([USGS's Landsat 8](https://www.usgs.gov/land-resources/nli/landsat/landsat-8?qt-science_support_page_related_con=0#qt-science_support_page_related_con) satellite orbits the earth every 100 minutes and returns to any given location roughly once every two weeks), which, clouds permitting, can be converted into surface temperature. Yet urban heat islands exist at the intersection of global forces and local conditions—where the weather meets the road, so to speak. In addition to investigating the nature of heat islands, this project will also create a workflow and tools for exploring heat islands in counties across the United States. This application will allow any user to zoom in on an area, see hot spots within it, and generate facts and figures that shed additional light on the phenomenon and its possible causes locally.  

Toward this end, the following uses [Google Earth Engine](https://earthengine.google.com/), a platform to geospatial analysis using JavaScript. Because this is an exercise as well as application, in the interest of practice, it involves manifold sources and types of data. The first half will leverage Landsat data from the United States, the second Sentinel data from the European Union. While former has a spatial resolution of 30 meters squared, the latter is sharper at 15. The map above uses [NASA's Moderate Resolution Imaging Spectroradiometer](https://terra.nasa.gov/about/terra-instruments/modis) (MODIS), which is valuable for its temporal if not its spatial resolution, imaging every point on earth every couple of days rather than weeks.  

## A note on compromises
Landsat and Sentinel offer distinct benefits. The former includes a thermal band, making it easy to calculate surface temperature; the latter does not. Yet Sentinel-2 provides a sharper resolution than Landsat 8, at 10 meters for most bands compared to 30. Further complicating the matter is the fact that Sentinel has not be live as long, making it difficult to measure change.  Because this study is concerned with temperature, the ability to accurately derive it is important—to say the least. To be a valuable planning tool, we need a resolution that allows us to target aspects of the built environment driving heat islands.

To create a layer of satellite imagery to ground our analysis of heat islands, we compensate for this using a technique called pan-sharpening, which combines a high resolution panchromatic band with the low resolution multispectral ones. This requires the Landsat 8 top-of-atmosphere, as opposed to surface, image collection; surface Landsat products do not have the panchromatic band (B8) necessary to polish the image.  

```
var cloudy = ee.ImageCollection("LANDSAT/LC08/C01/T1_TOA")
                  .filterDate('2010-01-01', '2018-12-31')
```
We then filter out the cloudy images and run through a script from the guide, converting the collection into an image with the `mosaic` command so that it the function works as written. Note also that we end by embedding visualization parameters into the image with the `visualize` function, which is important to the functioning of the application.  

```                  
var blurry   = cloudy.filter(ee.Filter.lt('CLOUD_COVER', 1));
var cleanest = blurry.mosaic()

var hsv = cleanest.select(['B4', 'B3', 'B2']).rgbToHsv();

var sharpest = ee.Image.cat([
  hsv.select('hue'), hsv.select('saturation'), cleanest.select('B8')
  ])
  .hsvToRgb()
  .visualize({min: 0, max: 0.25, gamma: [1.3, 1.3, 1.3]});

```   
The result is a picture that is not perfect but markedly sharper than the raw image. This image will become a comparison when we layer temperature data over it, allowing us to see the urban forms that dispose an area to relative heat.

###### Sharpening
![](https://github.com/asrenninger/heat/blob/master/sharpening.gif?raw=true)

*The value of sharpening satellite imagery, as above, comes from this improved clarity, as many blurred masses become defined buildings and the relationship between features of built environment become clear (see: code below to reproduce).*

## Aesthetic considerations
As open source software often does, Earth Engine has a strong community behind it. This project uses one particular contribution—a collection of palettes—from Gennadii Donchyts, Fedor Baart & Justin Braaten to improve its look. For more information and to see what other palettes are on offer, see [this repository](https://github.com/gee-community/ee-palettes).    

To pull in a palette without needing to code each hexadecimal string manually, load the package with the `require` function, then choose a palette. The example below shows the scheme used to produce the above animation. Although it saves time in scripting different palettes on the front end, calling the package does slow down processing on the back end.  

```
var palettes = require('users/gena/packages:palettes');
var palette  = palettes.misc.kovesi.rainbow_bgyrm_35_85_c71[7];

```

## Build up from there
With this map as a base, we can then layer on surface temperature along with a battery of explanatory variables—various indices to describe the landscape.

###### Toggling
![](https://github.com/asrenninger/heat/raw/master/use.gif?raw=true)
*The goal of this application is to allow users to toggle between an island of land surface temperature and any covariates of interest; this particularly interface facilitates quick transitions between layers to help the process*  

```
var dataset = ee.ImageCollection('LANDSAT/LC08/C01/T1_SR')
                  .filterDate('2018-01-01', '2018-12-31')
                  .map(maskL8sr)

var palettes = require('users/gena/packages:palettes');

var built = palettes.kovesi.diverging_linear_bjy_30_90_c45[7];
var green = palettes.kovesi.linear_green_5_95_c69[7];
var water = palettes.kovesi.linear_blue_5_95_c73[7];                  
```

```
var addNDBI = function(image) {
  var ndbi = image.normalizedDifference(['B6', 'B5']).rename('NDBI');
  return image.addBands(ndbi);
};

var addNDVI = function(image) {
  var ndvi = image.normalizedDifference(['B5', 'B4']).rename('NDVI');
  return image.addBands(ndvi);
};

var addNDWI = function(image) {
  var ndwi = image.normalizedDifference(['B3', 'B5']).rename('NDWI');
  return image.addBands(ndwi);
};
```

```
var ndbi    = dataset.map(addNDBI);
var ndbiest = ndbi.qualityMosaic('NDBI').select('NDBI').visualize({min: 0, max: 1, palette: built});

var ndvi    = dataset.map(addNDVI);
var ndviest = ndvi.qualityMosaic('NDVI').select('NDVI').visualize({min: -1, max: 1, palette: green});

var ndwi    = dataset.map(addNDWI);
var ndwiest = ndwi.qualityMosaic('NDWI').select('NDWI').visualize({min: -1, max: 1, palette: water});
```


```
var addHeat = function(image){
  var heat = image.select(['B10']).multiply(0.1).subtract(273.5).rename('HEAT');
  return image.addBands(heat);

}

var temp    = dataset.map(addHeat)

var heat = palettes.kovesi.diverging_rainbow_bgymr_45_85_c67[7]
var hottest = temp.qualityMosaic('HEAT').select('HEAT').visualize({min: 18, max: 45, palette: heat});
```



```
var images = {
  'temperature': hottest,
  'reality': sharpest,
  'natural': ndviest,
  'built': ndbiest,
  'water': ndwiest,
};
```
```
var leftMap = ui.Map();
leftMap.setControlVisibility(false);
var leftSelector = addLayerSelector(leftMap, 0, 'top-left');

var rightMap = ui.Map();
rightMap.setControlVisibility(false);
var rightSelector = addLayerSelector(rightMap, 1, 'top-right');
```

```
function addLayerSelector(mapToChange, defaultValue, position) {
  var label = ui.Label('Choose an image to visualize');

  function updateMap(selection) {
    mapToChange.layers().set(0, ui.Map.Layer(images[selection]));
  }

  var select = ui.Select({items: Object.keys(images), onChange: updateMap});
  select.setValue(Object.keys(images)[defaultValue], true);

  var controlPanel =
      ui.Panel({widgets: [label, select], style: {position: position}});

  mapToChange.add(controlPanel);
}
```


```
var splitPanel = ui.SplitPanel({
  firstPanel: leftMap,
  secondPanel: rightMap,
  wipe: true,
  style: {stretch: 'both'}
});

ui.root.widgets().reset([splitPanel]);
var linker = ui.Map.Linker([leftMap, rightMap]);
leftMap.centerObject(county);
```

Finally, we launch the application in the applications menu of the code editor. The result shows a search bar that allows us to move from city to city with ease.

###### Exploring
![](https://github.com/asrenninger/heat/raw/master/travelling.gif?raw=true)

# EXPLORATION
We can begin by looking at Philadelphia, which suffers from extreme during summer months, notably in July this year when in two separate weeks average surface temperature surpassed 90 degrees Fahrenheit.

Generally, the poorest areas in Philadelphia are the hottest. A corridor of marked with hot spots stretches along Roosevelt Boulevard, West and North Philadelphia also absorb heat. While the peripheral neighborhoods can either be above or below average, central Philadelphia is indeed a microcosm of the city, tracking along the average.

###### Probing
![](https://github.com/asrenninger/heat/raw/master/click.gif?raw=true)         

## Indices
Beginning to understand the contours of heat in a city requires measures of the built and natural environments that influence them. A popular technique is normalized difference, which takes the value of two bands and—of course—subtracts one from the before dividing by the total value of both. Using this simple technique can shed light on how lush, how built, or how wet a region is—NDVI (vegetation), NDBI (built, or built-up), and NDWI respectively. Compared to NDVI, the enhanced vegetation index (EVI) fails to meaningfully portray the character of the city and it thus falls out of the analysis.    

So why does NDVI vary over time—and why does it fall, rise and fall again over the span of a few winter months? A parsimonious explanation might be that changing seasons produce changing foliage, which is not always the lush, verdant chlorophyl that NDVI captures, but these images all come from a single year, which can be confounded by a prolonged damp period.   

We can view these indices simultaneously by modifying an [existing script](https://code.earthengine.google.com/?scriptPath=Examples:User+Interface/Linked+Maps) to create a similar one that uses our measures as layers, available [here](https://asrenninger.users.earthengine.app/).    

###### Various indices
![](https://raw.githubusercontent.com/asrenninger/heat/master/panels.PNG)
*Splitting maps across panels, as shown here, allows us to see relationships at once, rather toggling back and forth as earlier. We can see that there is no clear relationship between the built environment, as such, and temperature; rather vegetation and water explain and built character exists as a negative.*  


## Temperature


###### Temperature in Philadelphia
![](https://github.com/asrenninger/heat/blob/master/philly.gif?raw=true)

*The above animation shows average surface temperature in Philadelphia for each summer month over the past 10 years, beginning in May and ending in September. We can see that it begins cooler but even early on warmer spots emerge. The brightest spots show a temperature greater than 40 degree Celsius, or 105 degrees Fahrenheit; the darkest ones are lower than 18 degrees Celsius or 65 degrees Fahrenheit (see: code below to reproduce).*

One feature of note is that most business districts and the glass edifices that define them do much better than areas for industry or freight. This may be because reflective buildings repel heat or because tall buildings do let heat reach the ground. This holds for New York and Philadelphia—though not San Francisco, which has long slung buildings just off the ocean and high rise ones near the bay, likely influencing temperatures.

###### Zones over time
![](https://github.com/asrenninger/heat/raw/master/zones.png)

###### Vegetation over time
![](https://github.com/asrenninger/heat/raw/master/ndvi.png)

*These charts show NDVI and LST, with the two largely moving together; gaps exist where cloud cover limited measurement, but patterns emerge. Note that although vegetation and temperature rise and fall together, the areas with the lushest environments are those the coolest temperatures at any given time during the year.*

# INVESTIGATION
We can also test a clear hypothesis with hypothesis with this interface. New York City is defined by distinct built typologies, with towers in the park, houses on a block, low mixed use, high residential and higher commercial, and much more. The website [terrapattern](http://www.terrapattern.com/) allows any user to pick a square of urbanism in a select assemblage of cities and see similar forms and structures throughout the rest of that city. We can use this service to find points that correspond to features of natural and built environment, rather than selecting manually as the application facilitates.  

**Below we consider:**
- Commercial towers
- Residential towers
- Mixed neighborhoods
- Parks

###### Patterns of New York
![](https://github.com/asrenninger/heat/raw/master/pattern.gif?raw=true)

The ability to do this is built into the infrastructure constructed here: points grouped into geometries and comparisons across those geometries. We simply load the data from terrapattern after converting it from **geojson** to **shape**, using the asset manager in Earth Engine. Once we call features from the repository we need to transform them using the `geometry` command before binding them together. With this, the collection becomes a collection of point *collections* rather than a collection of multipoint *features*.    

```
var regions = ee.FeatureCollection([
  ee.Feature(
    ee.FeatureCollection('users/asrenninger/terra_parks').geometry(), {label: 'Parks'}),
  ee.Feature(
    ee.FeatureCollection('users/asrenninger/terra_blocks').geometry(), {label: 'Blocks'}),
  ee.Feature(
    ee.FeatureCollection('users/asrenninger/terra_towers').geometry(), {label: 'Towers'}),
  ee.Feature(
    ee.FeatureCollection('users/asrenninger/terra_business').geometry(), {label: 'Business'})
    ]);
```


## Guess and check

###### Typologies and heat
![](https://github.com/asrenninger/heat/raw/master/typologies.png)

###### Typologies and chlorophyl
![](https://github.com/asrenninger/heat/raw/master/comparison.png)

# EXPECTATION
The lion's share of time on this project went to developing a graphical interface; this necessarily meant that many analytical components did not reach fruition. Yet several germs for future development cropped up: the project began as a global comparison of urban heat islands but the computational power required to divide the world into cities, compute the average heat in those cities on a given day, and then remove that average from the measurements on the ground—so as to compare a city with itself rather than all cities across the globe—proved difficult. It was far easier to allow users to input a region and manage the calculations iteratively. Having said that, the concept succeed in contained geographies, like a single state or province. With improved parallel processing in future, the task may indeed be possible—if not with Landsat data than with MODIS.

###### Temperature against the average




# EXPLANATION
The following code executes this project. None of it is my own, though some combinations—amalgamations of snippets and ideas—are from my mind. For an introduction to Earth Engine, see

## How to Animate of Year of Surface Temperature

For a thorough explanation of this process, please see [this Justin Braaten tutorial](https://developers.google.com/earth-engine/tutorials/community/modis-ndvi-time-series-animation).
```
var collection = ee.ImageCollection("MODIS/006/MOD11A2").select('LST_Day_1km');
```

First we then set bounding box for the United States before transitioning into data processing.  
```
var mask = ee.FeatureCollection("TIGER/2016/States");

var region = ee.Geometry.Polygon(
  [[[-127.76347656249999, 49.42862428430654],
          [-127.76347656249999, 23.46798096552757],
          [-65.09746093749999, 23.46798096552757],
          [-65.09746093749999, 49.42862428430654]]],
          null, false
);
```
This section uses the `getRelative` function to determine the day of the year; we then select one year and then join the days from all other years to that list of days. We finish by taking the `median`, but could just have easily chosen the `mean`.

```
var collection = collection.map(function(img) {
  var doy = ee.Date(img.get('system:time_start')).getRelative('day', 'year');
  return img.set('doy', doy);
});

var distinctDOY = col.filterDate('2013-01-01', '2014-01-01');

var filter = ee.Filter.equals({leftField: 'doy', rightField: 'doy'});

var join = ee.Join.saveAll('doy_matches');

var joinCol = ee.ImageCollection(join.apply(distinctDOY, collection, filter));

var comp = joinCol.map(function(img) {
  var doyCol = ee.ImageCollection.fromImages(
    img.get('doy_matches')
  );
  return doyCol.reduce(ee.Reducer.median());
});
```
In order to plot and animate this, we simply pull in a palette from the Earth Engine community, as above, set parameters and then print the it.

```
var palettes = require('users/gena/packages:palettes');
var palette = palettes.kovesi.rainbow_bgyrm_35_85_c71[7];

var visParams = {
  min: 14000.0,
  max: 16000.0,
  palette: palette,
};

var rgbVis = comp.map(function(img) {
  return img.visualize(visParams).clip(mask);
});

var gifParams = {
  'region': region,
  'dimensions': 600,
  'crs': 'EPSG:5071',
  'framesPerSecond': 10
};

print(ui.Thumbnail(rgbVis, gifParams));
```
