
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Inverse Midpoint Calculator (Draggable Markers)</title>
  <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 20px;
      background: none;
      background-color: transparent;
    }
    #map {
      width: 100%;
      height: 400px;
      margin-top: 20px;
      margin-bottom: 20px;
    }
    label {
      display: inline-block;
      width: 80px;
      margin-bottom: 5px;
    }
    input[type="text"] {
      width: 120px;
      margin-bottom: 10px;
    }
    button {
      margin-top: 10px;
      padding: 6px 16px;
      font-size: 1em;
    }
    h1, h2 {
      margin-bottom: 10px;
    }
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
  <button id="calcBtn">Calculate Inverse Midpoint</button>
  <div id="map"></div>
  <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
  <script>
    // Define the custom icon using your PNGs
    const customIcon = L.icon({
      iconUrl: 'images/marker-icon.png',
      shadowUrl: 'images/marker-shadow.png',
      iconSize:     [25, 41],
      iconAnchor:   [12, 41],
      popupAnchor:  [1, -34],
      shadowSize:   [41, 41]
    });

    // Initialize map and set default view
    var map = L.map('map').setView([51.5, -0.09], 13);

    // Add OpenStreetMap tile layer
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap'
    }).addTo(map);

    // Initialize markers with custom icon and make them draggable
    var markerA = L.marker([51.5, -0.09], {icon: customIcon, draggable: true}).addTo(map);
    var markerM = L.marker([51.505, -0.08], {icon: customIcon, draggable: true}).addTo(map);

    // Set input fields to initial marker positions
    document.getElementById('latA').value = markerA.getLatLng().lat.toFixed(6);
    document.getElementById('lonA').value = markerA.getLatLng().lng.toFixed(6);
    document.getElementById('latM').value = markerM.getLatLng().lat.toFixed(6);
    document.getElementById('lonM').value = markerM.getLatLng().lng.toFixed(6);

    // Update input fields when markers are dragged
    markerA.on('dragend', function(e) {
      var pos = markerA.getLatLng();
      document.getElementById('latA').value = pos.lat.toFixed(6);
      document.getElementById('lonA').value = pos.lng.toFixed(6);
    });
    markerM.on('dragend', function(e) {
      var pos = markerM.getLatLng();
      document.getElementById('latM').value = pos.lat.toFixed(6);
      document.getElementById('lonM').value = pos.lng.toFixed(6);
    });

    // Update markers when input fields are changed
    function updateMarkersFromInputs() {
      var latA = parseFloat(document.getElementById('latA').value);
      var lonA = parseFloat(document.getElementById('lonA').value);
      var latM = parseFloat(document.getElementById('latM').value);
      var lonM = parseFloat(document.getElementById('lonM').value);
      if (!isNaN(latA) && !isNaN(lonA)) {
        markerA.setLatLng([latA, lonA]);
      }
      if (!isNaN(latM) && !isNaN(lonM)) {
        markerM.setLatLng([latM, lonM]);
      }
    }
    document.getElementById('latA').addEventListener('change', updateMarkersFromInputs);
    document.getElementById('lonA').addEventListener('change', updateMarkersFromInputs);
    document.getElementById('latM').addEventListener('change', updateMarkersFromInputs);
    document.getElementById('lonM').addEventListener('change', updateMarkersFromInputs);

    // Calculate the inverse midpoint
    function calculateInverseMidpoint() {
      var latA = parseFloat(document.getElementById('latA').value);
      var lonA = parseFloat(document.getElementById('lonA').value);
      var latM = parseFloat(document.getElementById('latM').value);
      var lonM = parseFloat(document.getElementById('lonM').value);

      // Simple inverse midpoint calculation (for demonstration)
      var latB = 2 * latM - latA;
      var lonB = 2 * lonM - lonA;

      // Add or move a marker for the calculated point
      if (window.markerB) {
        markerB.setLatLng([latB, lonB]);
      } else {
        window.markerB = L.marker([latB, lonB], {icon: customIcon, draggable: true}).addTo(map)
          .bindPopup("Inverse Point B").openPopup();
        markerB.on('dragend', function(e) {
          var pos = markerB.getLatLng();
          // Optionally update inputs or recalculate as needed
        });
      }
      map.panTo([latM, lonM]);
    }

    document.getElementById('calcBtn').addEventListener('click', calculateInverseMidpoint);
  </script>
</body>
</html>
