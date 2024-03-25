# WEEK - 1
## Task 
 Propose an example algorithm that summarises a geospatial time series and describes its time and memory complexity.

## Approach Followed:
1. Load Sentinel-2 Image Collection: The code begins by loading the Sentinel-2 Harmonized dataset using ee.ImageCollection('COPERNICUS/S2_HARMONIZED').
2. Define Time Range: The start and end dates for the analysis are specified using ee.Date.fromYMD() to create startDate and endDate variables representing January 1, 2020, and January 1, 2023, respectively.
3. Functions for NDVI Calculation and Cloud Masking:
	• addNDVI(): This function calculates the Normalized Difference Vegetation Index (NDVI) by taking the normalized difference between the near-infrared (B8) and red (B4) bands and adds it as a new band to each image in the collection.
	• maskS2clouds(): This function applies a cloud mask to Sentinel-2 images using the 'QA60' band to filter out cloudy and cirrus pixels.
4. Preprocess Sentinel-2 Image Collection:
	• The original Sentinel-2 image collection is filtered by date, cloud cover, and a specified geometry (region of interest).
	• Clouds are masked using the maskS2clouds() function, and NDVI bands are added to each image using the addNDVI() function.
	• The result is stored in the originalCollection variable.
5. Display Time-Series Chart:
	• A time-series chart is created using ui.Chart.image.series() to visualize the mean NDVI values over time within the specified geometry.
	• The chart is customized with options for title, axis labels, line color, and point size.
6. Export NDVI Time-Series as Video:
	• The NDVI time-series is visualized as a video using a predefined color palette and the visualizeImage() function.
	• The visualization is exported to Google Drive as a video file using Export.video.toDrive().
This approach involves loading, preprocessing, visualizing, and exporting Sentinel-2 imagery data to analyze NDVI time-series trends over the specified time range and geographical area of interest (northern India). The code aims to provide insights into vegetation dynamics and land cover changes over time.



## Psuedocode 
1. Load Sentinel-2 Image Collection:
   - Load the Sentinel-2 Harmonized dataset as an Image Collection.

2. Define Time Range:
   - Define the start and end dates for the analysis period.

3. Define Functions:
   - Define a function to calculate NDVI:
     - Input: Sentinel-2 image
     - Output: Image with NDVI band added

   - Define a function to mask clouds:
     - Input: Sentinel-2 image
     - Output: Cloud-masked image

4. Preprocess Sentinel-2 Image Collection:
   - Filter the original Sentinel-2 Image Collection by:
     - Date range
     - Cloud cover percentage
     - Geographic region of interest (e.g., northern India)
   - Apply cloud masking to filter out cloudy and cirrus pixels.
   - Calculate NDVI for each image in the filtered collection.

5. Visualize Time-Series Data:
   - Display a time-series chart to visualize the mean NDVI values over time:
     - Input: Filtered and processed Sentinel-2 Image Collection
     - Output: Time-series chart showing NDVI trends over time

6. Export Time-Series as Video:
   - Visualize the NDVI time-series as a video:
     - Input: Filtered and processed Sentinel-2 Image Collection
     - Output: Video file showing NDVI variations over time

7. End 






## Code 
var s2 = ee.ImageCollection('COPERNICUS/S2_HARMONIZED');

var startDate = ee.Date.fromYMD(2020, 1, 1);
var endDate = ee.Date.fromYMD(2023, 1, 1);

// Function to add a NDVI band to an image
function addNDVI(image) {
  var ndvi = image.normalizedDifference(['B8', 'B4']).rename('ndvi');
  return image.addBands(ndvi);
} 

// Function to mask clouds
function maskS2clouds(image) {
  var qa = image.select('QA60')
  var cloudBitMask = 1 << 10;
  var cirrusBitMask = 1 << 11;
  var mask = qa.bitwiseAnd(cloudBitMask).eq(0).and(
             qa.bitwiseAnd(cirrusBitMask).eq(0))
  return image.updateMask(mask).divide(10000)
      .select("B.*")
      .copyProperties(image, ["system:time_start"])
}

var originalCollection = s2
  .filter(ee.Filter.date(startDate, endDate))
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 30))
  .filter(ee.Filter.bounds(geometry))
  .map(maskS2clouds)
  .map(addNDVI);


// Display a time-series chart
var chart = ui.Chart.image.series({
  imageCollection: originalCollection.select('ndvi'),
  region: geometry,
  reducer: ee.Reducer.mean(),
  scale: 20
}).setOptions({
      title: 'Original NDVI Time Series',
      interpolateNulls: false,
      vAxis: {title: 'NDVI', viewWindow: {min: 0, max: 1}},
      hAxis: {title: '', format: 'YYYY-MM'},
      lineWidth: 1,
      pointSize: 4,
      series: {
        0: {color: '#238b45'},
      },

    })
print(chart);

// Let's export the NDVI time-series as a video

var palette = ['#d73027','#f46d43','#fdae61','#fee08b',
  '#ffffbf','#d9ef8b','#a6d96a','#66bd63','#1a9850'];
var ndviVis = {min:-0.2, max: 0.8,  palette: palette}

Map.centerObject(geometry, 16);
var bbox = Map.getBounds({asGeoJSON: true});

var visualizeImage = function(image) {
  return image.visualize(ndviVis).clip(bbox).selfMask()
}

var visCollectionOriginal = originalCollection.select('ndvi')
  .map(visualizeImage)


Export.video.toDrive({
  collection: visCollectionOriginal,
  description: 'Original_Time_Series',
  folder: 'earthengine',
  fileNamePrefix: 'original',
  framesPerSecond: 2,
  dimensions: 800,
  region: bbox})



## Time and space complexities 

Analyzing the time and space complexity of the implemented algorithm involves evaluating how the algorithm's performance scales with the size of the input data (in this case, the Sentinel-2 image collection) and the computational resources required for its execution. Let's break down the time and space complexity for each major component of the algorithm:

1. Loading and Filtering Image Collection:
   - Time Complexity: O(n)
     - The time complexity for loading and filtering the Sentinel-2 image collection depends on the number of images in the collection (`n`). Filtering by date, cloud cover, and geographic region typically involves iterating over the image collection once.
   - Space Complexity: O(1)
     - The space complexity for loading and filtering the image collection is constant, as it does not require additional memory allocation proportional to the size of the input data.

2. NDVI Calculation and Cloud Masking:
   - Time Complexity: O(n)
     - Calculating NDVI and applying cloud masking involves iterating over each image in the filtered collection (`n`). These operations typically involve pixel-wise computations, which are linear with respect to the number of pixels in each image.
   - Space Complexity: O(1)
     - The space complexity for NDVI calculation and cloud masking is constant, as it does not require additional memory allocation proportional to the size of the input data.

3. Time-Series Visualization:
   - Time Complexity: O(n)
     - Generating the time-series chart involves processing the filtered image collection (`n`) to compute summary statistics for visualization. The time complexity depends on the number of images in the collection.
   - Space Complexity: O(1)
     - The space complexity for time-series visualization is constant, as it does not require additional memory allocation proportional to the size of the input data.

4. Exporting Video:
   - Time Complexity: O(n)
     - Exporting the NDVI time-series as a video involves processing the filtered image collection (`n`) to create individual frames for the video. The time complexity depends on the number of images in the collection.
   - Space Complexity: O(n)
     - The space complexity for exporting the video is linear with respect to the number of images in the collection, as it requires storing the frames temporarily before exporting.

Overall, the time complexity of the implemented algorithm is primarily determined by the number of images in the Sentinel-2 image collection (`n`). The space complexity remains constant for most operations, except for exporting the video, which requires temporary storage of image frames proportional to the size of the input data.



