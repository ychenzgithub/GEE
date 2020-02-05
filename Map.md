## Base map

### Default options
There are four default map options 'roadmap', 'satellite', 'hybrid', and 'terrain'. Use setOptions to set these styles.
```
Map.setOptions('satellite');
```
### Customize base map
The Google Maps API (and by extension, Earth Engine) gives you the ability to control a large number of map features and elements.
The full list of elements that you can modify can be found in the [Google Maps documentation](https://developers.google.com/maps/documentation/javascript/style-reference)
```
var roadNetwork = [
  {stylers: [{saturation: -100}]}, {
    featureType: 'road.highway',
    elementType: 'geometry.fill',
    stylers: [{color: '#000055'}, {weight: 2.5}]
  },
  {
    featureType: 'road.highway',
    elementType: 'geometry.stroke',
    stylers: [{color: '#000000'}, {weight: 2}]
  },
  {
    featureType: 'road.arterial',
    elementType: 'geometry',
    stylers: [{color: '#FF0000'}, {weight: 1.8}]
  },
  {
    featureType: 'road.local',
    elementType: 'geometry',
    stylers: [{color: '#00FF55'}, {weight: 1.5}]
  }
];
Map.setOptions(
    'roadNetwork', {iconChange: iconChange, roadNetwork: roadNetwork});
```

### Tools of base map customization
Some community projects provide nice map options. You can copy the code from [Snazzy maps](https://snazzymaps.com/) and use for map options.  
You may also use this [map style tool](https://mapstyle.withgoogle.com) to customize the style.

