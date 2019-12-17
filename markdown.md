# An Application to Discover and Map Heat Islands
###### Andrew Renninger

# CONTENTS
- [INSPIRATION](#INSPIRATION)
- [EXPLORATION](#EXPLORATION)
- [INVESTIGATION](#INVESTIGATION)
- [EXPECTATION](#EXPECTATION)
- [EXPLANATION](#EXPLANATION)

# INSPIRATION
The goal of this project is to create a process by which we can understand urban heat islands—the phenomenon wherein built and natural conditions conspire to raise temperature in urban areas during periods of hot weather—with readily accessibly data from satellite images. During a spell of hot weather in late July 2019, mercury levels in Philadelphia remained above 90 degrees Fahrenheit during the day and even held above 80 over night. Yet this swelter was not felt equally across the city: the hottest and coolest—comparably—portions of the city differed by as much as 20 degrees. A park could be bearable; a parking lot could be insufferable. Each neighborhood sees unique effects from sun and heat and each season is different. Because of this, this following project is an application that can be used by all. While this application represents a high cost to build, it yields a low cost for experimentation. The subsequent exploration is modest, but folding new cities into it and layering new methods onto only becomes easier with such an interface. Additionally, because it lacks advanced statistical analysis, forgoing the statistical for the graphical, it serves as a means to travel the globe by satellite—more qualitatively than quantitatively and with breadth rather than depth, although by allowing for such travel is by no means trivial.         

###### Temperature in the United States
![](https://github.com/asrenninger/tinkering/blob/master/viz/temp_kovesi.gif?raw=true)

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

![](https://github.com/asrenninger/heat/blob/master/sharpening.gif?raw=true)

*The value of sharpening satellite imagery, as above, comes from this improved clarity, as many blurred masses become defined buildings and the relationship between features of built environment become clear (see: code below to reproduce).*

## Build up from there
With this map as a base, we can then layer on surface temperature along with a battery of explanatory variables—various indices to describe the landscape.


## Aesthetic considerations
As open source software often does, Earth Engine has a strong community behind it. This project uses one particular contribution—a collection of palettes—from Gennadii Donchyts, Fedor Baart & Justin Braaten to improve its look. For more information and to see what other palettes are on offer, see [this repository](https://github.com/gee-community/ee-palettes).    

To pull in a palette without needing to code each hexadecimal string manually, load the package with the `require` function, then choose a palette. The example below shows the scheme used to produce the above animation. Although it saves time in scripting different palettes on the front end, calling the package does slow down processing on the back end.  

```
var palettes = require('users/gena/packages:palettes');
var palette  = palettes.misc.kovesi.rainbow_bgyrm_35_85_c71[7];

```

# EXPLORATION
We can begin by looking at Philadelphia, which suffers from extreme during summer months, notably in July this year when in two separate weeks average surface temperature surpassed 90 degrees Fahrenheit.     

## Indices
Beginning to understand the contours of heat in a city requires measures of the built and natural environments that influence them. A popular technique is normalized difference, which takes the value of two bands and—of course—subtracts one from the before dividing by the total value of both. Using this simple technique can shed light on how lush, how built, or how wet a region is—NDVI (vegetation), NDBI (built, or built-up), and NDWI respectively. Compared to NDVI, the enhanced vegetation index (EVI) fails to meaningfully portray the character of the city and it thus falls out of the analysis.    

So why does NDVI vary over time—and why does it fall, rise and fall again over the span of a few winter months? A parsimonious explanation might be that changing seasons produce changing foliage, which is not always the lush, verdant chlorophyl that NDVI captures, but these images all come from a single year, which can be confounded by a prolonged damp period.   

###### Various indices
![]()

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

Below we consider:
- Commercial towers
- Residential towers
- Mixed neighborhoods
- Parks

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

var collection = collection.map(function(img) {
  var doy = ee.Date(img.get('system:time_start')).getRelative('day', 'year');
  return img.set('doy', doy);
});

var distinctDOY = col.filterDate('2013-01-01', '2014-01-01');

var filter = ee.Filter.equals({leftField: 'doy', rightField: 'doy'});

var join = ee.Join.saveAll('doy_matches');

var joinCol = ee.ImageCollection(join.apply(distinctDOY, col, filter));

var comp = joinCol.map(function(img) {
  var doyCol = ee.ImageCollection.fromImages(
    img.get('doy_matches')
  );
  return doyCol.reduce(ee.Reducer.median());
});

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

7. **Fill out your sections as needed.**  Give each section a unique name in the section `id` property. This will become the HTML `div` `id`, so avoid spaces in the name. The `title`, `description` properties are optional. The `description` supports HTML tags. If you have an image that goes with that section of the story, add the path to the image in the `image` property.

8. For `location`, you can use the `helper.html` file to help you determine the map's position. This tool prints the location settings of the map on the screen in a format ready for copy/paste into the template.

9. Repeat until you have the location entered for each of your sections.

10. Open `index.html` in a browser, and scroll. Voila!

#### Generate Map Position Using `Helper.html`

Using the `helper.html` file, you can search for places, zoom, pan, tilt, and rotate the map to get the desired map position (_Hint_: To tilt and rotate the map, right-click and drag the map).

Notice the location parameters are updated in the upper left corner with everytime you move the map. You can copy the location definition from that page into the `config.js` `location` property section.

There is also a hosted version of this file at [https://demos.mapbox.com/location-helper/](https://demos.mapbox.com/location-helper/)

![location helper screen capture](assets/locationhelper.gif)
