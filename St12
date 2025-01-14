// Define the area of interest (North Dakota)
var northDakota = ee.FeatureCollection('TIGER/2018/States')
  .filter(ee.Filter.eq('NAME', 'North Dakota'));

// Define the date range for summer 2024
var startDate = '2024-06-01';
var endDate = '2024-08-31';

// Load Sentinel-2 surface reflectance dataset
var s2 = ee.ImageCollection('COPERNICUS/S2_SR')
  .filterBounds(northDakota)
  .filterDate(startDate, endDate)
  .filter(ee.Filter.lt('CLOUDY_PIXEL_PERCENTAGE', 20))
  .map(function(image) {
    return image.clip(northDakota); // Clip to North Dakota
  });

// Select relevant Sentinel-2 bands
var s2Bands = ['B2', 'B3', 'B4', 'B8']; // Blue, Green, Red, NIR
s2 = s2.select(s2Bands).median();

// Load Sentinel-1 radar data
var s1 = ee.ImageCollection('COPERNICUS/S1_GRD')
  .filterBounds(northDakota)
  .filterDate(startDate, endDate)
  .filter(ee.Filter.eq('orbitProperties_pass', 'DESCENDING')) // Descending orbit
  .filter(ee.Filter.eq('instrumentMode', 'IW')) // Interferometric Wide Swath
  .map(function(image) {
    return image.clip(northDakota); // Clip to North Dakota
  });

// Select VV and VH polarization bands
var s1Bands = ['VV', 'VH'];
s1 = s1.select(s1Bands).median();

// Combine Sentinel-2 and Sentinel-1 data
var fusion = s2.addBands(s1);

// Visualize the fusion layer
Map.centerObject(northDakota, 7);
Map.addLayer(fusion, {
  bands: ['B4', 'B3', 'B2'], // Display RGB from Sentinel-2
  min: 0,
  max: 3000
}, 'Sentinel-2 RGB');
Map.addLayer(fusion, {
  bands: ['VV', 'VH'], // Display radar backscatter
  min: -20,
  max: 0
}, 'Sentinel-1 VV/VH');

// Export the fusion layer
Export.image.toDrive({
  image: fusion,
  description: 'Sentinel1_2_Fusion_ND_Summer2024',
  folder: 'GEE_Exports',
  scale: 10, // Export at 10m resolution
  region: northDakota.geometry(),
  maxPixels: 1e13
});
