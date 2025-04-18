<html>
<head>
    <title>Inverse Midpoint Calculator (Draggable)</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <style>
        #map { height: 500px; margin-bottom: 1em; }
        .input-group { margin: 10px 0; }
        label { display: inline-block; width: 120px; }
        body { font-family: Arial, sans-serif; }
         /* Override Leaflet's default marker paths */
    .leaflet-marker-icon {
      background-image: url('/images/marker-icon.png') !important;
    }
    .leaflet-marker-icon.leaflet-retina {
      background-image: url('/images/marker-icon-2x.png') !important;
    }
    .leaflet-marker-shadow {
      background-image: url('/images/marker-shadow.png') !important;
    }
    </style>
</head>
<body>
    <h2>Inverse Midpoint Calculator (Draggable Markers)</h2>
    <div class="input-group">
        <h3>Point A (Start)</h3>
        <label>Latitude:</label>
        <input type="number" id="a_lat" value="30.4079" step="0.0001"><br>
        <label>Longitude:</label>
        <input type="number" id="a_lon" value="-81.5823" step="0.0001">
    </div>
    <div class="input-group">
        <h3>Midpoint (M)</h3>
        <label>Latitude:</label>
        <input type="number" id="m_lat" value="33.6562" step="0.0001"><br>
        <label>Longitude:</label>
        <input type="number" id="m_lon" value="-80.8234" step="0.0001">
    </div>
    <button onclick="updateMarkers()">Calculate Inverse Midpoint</button>
    <div id="map"></div>
    <div id="result"></div>

    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
    <script>
        // Inverse midpoint function (spherical)
        function inverseMidpoint(a_lat, a_lon, m_lat, m_lon) {
            const toRad = deg => deg * Math.PI / 180;
            const toDeg = rad => rad * 180 / Math.PI;

            const φ1 = toRad(a_lat);
            const λ1 = toRad(a_lon);
            const φ2 = toRad(m_lat);
            const λ2 = toRad(m_lon);

            const Δφ = φ2 - φ1;
            const Δλ = λ2 - λ1;

            // Haversine distance (angular)
            const a = Math.sin(Δφ/2)**2 + Math.cos(φ1)*Math.cos(φ2)*Math.sin(Δλ/2)**2;
            const d = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

            // Initial bearing
            const y = Math.sin(Δλ) * Math.cos(φ2);
            const x = Math.cos(φ1)*Math.sin(φ2) - Math.sin(φ1)*Math.cos(φ2)*Math.cos(Δλ);
            const θ = Math.atan2(y, x);

            // Destination point (twice the angular distance)
            const δ = 2 * d;
            const φ3 = Math.asin(Math.sin(φ1)*Math.cos(δ) + Math.cos(φ1)*Math.sin(δ)*Math.cos(θ));
            const λ3 = λ1 + Math.atan2(Math.sin(θ)*Math.sin(δ)*Math.cos(φ1), Math.cos(δ) - Math.sin(φ1)*Math.sin(φ3));

            let lon = toDeg(λ3);
            lon = (lon + 540) % 360 - 180; // Normalize

            return [toDeg(φ3), lon];
        }

        // Initialize map
        const map = L.map('map').setView([32.5, -81.2], 6);
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors'
        }).addTo(map);

        // Markers
        let markerA, markerM, markerB;

        function addOrMoveMarker(marker, lat, lon, options, onDragEnd) {
            if (marker) {
                marker.setLatLng([lat, lon]);
            } else {
                marker = L.marker([lat, lon], options);
                if (onDragEnd) marker.on('dragend', onDragEnd);
                marker.addTo(map);
            }
            return marker;
        }

        function updateMarkers() {
            // Get input values
            const a_lat = parseFloat(document.getElementById('a_lat').value);
            const a_lon = parseFloat(document.getElementById('a_lon').value);
            const m_lat = parseFloat(document.getElementById('m_lat').value);
            const m_lon = parseFloat(document.getElementById('m_lon').value);

            // Calculate inverse midpoint
            const [b_lat, b_lon] = inverseMidpoint(a_lat, a_lon, m_lat, m_lon);

            // Add or move markers
            markerA = addOrMoveMarker(markerA, a_lat, a_lon, {draggable: true, title: "Point A"}, function(e) {
                const pos = e.target.getLatLng();
                document.getElementById('a_lat').value = pos.lat.toFixed(6);
                document.getElementById('a_lon').value = pos.lng.toFixed(6);
                updateMarkers();
            });
            markerA.bindPopup("Point A").openPopup();

            markerM = addOrMoveMarker(markerM, m_lat, m_lon, {draggable: true, title: "Midpoint"}, function(e) {
                const pos = e.target.getLatLng();
                document.getElementById('m_lat').value = pos.lat.toFixed(6);
                document.getElementById('m_lon').value = pos.lng.toFixed(6);
                updateMarkers();
            });
            markerM.bindPopup("Midpoint");

            markerB = addOrMoveMarker(markerB, b_lat, b_lon, {draggable: false, title: "Inverse Midpoint"});
            markerB.bindPopup("Inverse Midpoint");

            // Fit map to show all points
            const group = new L.featureGroup([markerA, markerM, markerB]);
            map.fitBounds(group.getBounds().pad(0.3));

            // Show result
            document.getElementById('result').innerHTML = `
                <h3>Results:</h3>
                <strong>Inverse Midpoint:</strong> ${b_lat.toFixed(6)}°N, ${b_lon.toFixed(6)}°W<br>
                <a href="https://www.google.com/maps/place/${b_lat},${b_lon}" target="_blank">View on Google Maps</a>
            `;
        }

        // Initialize markers on page load
        window.onload = updateMarkers;
    </script>
</body>
</html>
