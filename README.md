<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inverse Midpoint Calculator (Draggable Markers)</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <style>
    html, body, #map {
      height: 100%;
      margin: 0;
      padding: 0;
      background: none !important;
      background-color: transparent !important;
      border: none !important;
    }
    #map { width: 100%; height: 400px; }
  </style>
</head>
<body>
  <h1>Inverse Midpoint Calculator (Draggable Markers)</h1>
  <div>
    <h2>Point A (Start)</h2>
    <label for="latA">Latitude:</label>
    <input id="latA" type="text"><br>
    <label for="lonA">Longitude:</label>
    <input id="lonA" type="text"><br>
  </div>
  <div>
    <h2>Midpoint (M)</h2>
    <label for="latM">Latitude:</label>
    <input id="latM" type="text"><br>
    <label for="lonM">Longitude:</label>
    <input id="lonM" type="text"><br>
  </div>
  <button>Calculate Inverse Midpoint</button>
  <div id="map"></div>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script>
    // Point Leaflet to the correct marker image paths
    L.Icon.Default.mergeOptions({
      iconRetinaUrl: 'images/marker-icon-2x.png',
      iconUrl: 'images/marker-icon.png',
      shadowUrl: 'images/marker-shadow.png'
    });

    // Initialize the map
    var map = L.map('map').setView([51.5, -0.09], 13);

    // Add a tile layer (OpenStreetMap)
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap'
    }).addTo(map);

    // Example marker to show default icon now works
    L.marker([51.5, -0.09]).addTo(map)
      .bindPopup("Default marker with correct transparency.");
  </script>
</body>
</html>
