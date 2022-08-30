# LULC_classification
var ROI = ee.FeatureCollection(ROI);
Map.addLayer(ROI);
Map.centerObject(ROI);
Map.centerObject(ROI,9);

var S2Collection = ee.ImageCollection('COPERNICUS/S2_SR')
    .filterDate('2020-01-01','2020-01-30')
    .filterBounds(ROI);

// var composite = S2Collection.clip(ROI);
Map.addLayer(S2Collection, {bands: ['B4', 'B3', 'B2'], max: 0.5, gamma: 2}, 'L8 Image', false);

var newfc = BuiltUp_Areas.merge(Waterbodies).merge(Bare_Land).merge(Trees).merge(Grassland_n_shrubs).merge(Clouds);
print(newfc, 'newfc')

var bands = ['B2', 'B3', 'B4', 'B5', 'B6', 'B7'];

var training = S2Collection.select(bands).sampleRegions({
  collection: newfc,
  properties: ['LandCover'],
  scale: 30
});

var classifier = ee.Classifier.smileRandomForest(10).train({
  features: training,
  classProperty: 'LandCover',
  inputProperties: bands
});

var classified = composite.select(bands).classify(classifier);

var palette = [
  'd63000', // BuiltUp_Areas (0)  // red
  '98ff00', // Trees (1)  // green
  '0b4a8b' //  WaterBodies (2) // blue
];

Map.centerObject(ROI,10);
Map.addLayer(classified, {min: 0, max: 2, palette: palette}, 'Land Use Classification');
