<html>
<head>
    <title>Map Point Reflector</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <style>
        #map { height: 500px; margin-bottom: 1em; }
        .input-group { margin: 10px 0; }
        label { display: inline-block; width: 120px; }
        body { font-family: Arial, sans-serif; }
        .leaflet-marker-icon,
        .leaflet-marker-shadow {
            background-color: transparent !important;
            background: transparent !important;
        }
    </style>
</head>
<body>
    <h2>Map Point Reflector</h2>
    <h3>Enter a point and midpoint to find the reflected location on the map.</h3>
    <h4>You can also drag & drop Point A or the Midpoint</h4>
    <div class="input-group">
        <h3>Point A</h3>
        <label>Coordinates:</label>
        <input type="text" id="a_coords" value="37.95792, -121.29413" placeholder="lat, lon">
    </div>
    <div class="input-group">
        <h3>Midpoint</h3>
        <label>Coordinates:</label>
        <input type="text" id="m_coords" value="39.60798, -116.37683" placeholder="lat, lon">
    </div>
    <button onclick="updateMarkers()">Calculate Reflected Point</button>
    <div id="map"></div>
    <div id="result"></div>

    <!-- Load Leaflet -->
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
    
    <script>
        // Global variables for markers
        let markerA = null;
        let markerM = null;
        let markerB = null;
        
        // Initialize map
        const map = L.map('map', {
            worldCopyJump: true // Helps with the wrapping behavior
        }).setView([32.5, -81.2], 6);
        
        // Add tile layer
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors',
            noWrap: false // Allow the map to repeat horizontally
        }).addTo(map);

        // Fix Leaflet's default icon paths
        delete L.Icon.Default.prototype._getIconUrl;
        L.Icon.Default.mergeOptions({
            iconRetinaUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon-2x.png',
            iconUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon.png',
            shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png'
        });

        // Function to normalize coordinates
        function normalizeCoordinates(lat, lng) {
            // Constrain latitude to -90 to 90
            lat = Math.max(-90, Math.min(90, lat));
            
            // Normalize longitude to -180 to 180
            lng = ((lng + 540) % 360) - 180;
            
            return [lat, lng];
        }

        // Improved inverse midpoint function using vector-based calculation
        function improvedInverseMidpoint(a_lat, a_lon, m_lat, m_lon) {
            const toRad = deg => deg * Math.PI / 180;
            const toDeg = rad => rad * 180 / Math.PI;

            const φ1 = toRad(a_lat);
            const λ1 = toRad(a_lon);
            const φ2 = toRad(m_lat);
            const λ2 = toRad(m_lon);

            // Calculate angular distance between point A and midpoint
            const Δφ = φ2 - φ1;
            const Δλ = λ2 - λ1;
            const a = Math.sin(Δφ/2)**2 + Math.cos(φ1)*Math.cos(φ2)*Math.sin(Δλ/2)**2;
            const angularDistance = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));

            // Check for antipodal points (distance close to π radians or 180 degrees)
            if (Math.abs(angularDistance - Math.PI) < 1e-10) {
                return [null, null]; // Reflection is undefined for antipodal points
            }

            // Check if points are identical or very close
            if (angularDistance < 1e-10) {
                return [null, null]; // Cannot determine reflection direction
            }

            // Calculate initial bearing from point A to midpoint
            const y = Math.sin(Δλ) * Math.cos(φ2);
            const x = Math.cos(φ1)*Math.sin(φ2) - Math.sin(φ1)*Math.cos(φ2)*Math.cos(Δλ);
            const θ = Math.atan2(y, x);

            // Continue along the same great circle for the same distance to get reflected point
            // Double the angular distance from A to M
            const δ = 2 * angularDistance;
            
            // If this exceeds 180 degrees, we need to warn about potential issues
            if (δ > Math.PI) {
                console.warn("Warning: Reflected point is more than 180° from the original point");
            }

            // Calculate the destination point
            const φ3 = Math.asin(Math.sin(φ1)*Math.cos(δ) + Math.cos(φ1)*Math.sin(δ)*Math.cos(θ));
            const λ3 = λ1 + Math.atan2(Math.sin(θ)*Math.sin(δ)*Math.cos(φ1), 
                                      Math.cos(δ) - Math.sin(φ1)*Math.sin(φ3));

            // Convert back to degrees
            let b_lat = toDeg(φ3);
            let b_lon = toDeg(λ3);
            
            // Normalize longitude to -180 to 180
            b_lon = ((b_lon + 540) % 360) - 180;

            return [b_lat, b_lon];
        }

        // Helper function to add or move markers
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

        // Update markers
        function updateMarkers() {
            try {
                // Get input values and normalize them
                let [a_lat, a_lon] = document.getElementById('a_coords').value.split(',').map(v => parseFloat(v.trim()));
                let [m_lat, m_lon] = document.getElementById('m_coords').value.split(',').map(v => parseFloat(v.trim()));
                
                // Validate inputs
                if (isNaN(a_lat) || isNaN(a_lon) || isNaN(m_lat) || isNaN(m_lon)) {
                    throw new Error("Invalid coordinates. Please enter valid numbers.");
                }
                
                // Normalize the input coordinates
                [a_lat, a_lon] = normalizeCoordinates(a_lat, a_lon);
                [m_lat, m_lon] = normalizeCoordinates(m_lat, m_lon);
                
                // Update the input fields with normalized values
                document.getElementById('a_coords').value = `${a_lat.toFixed(6)}, ${a_lon.toFixed(6)}`;
                document.getElementById('m_coords').value = `${m_lat.toFixed(6)}, ${m_lon.toFixed(6)}`;
                
                // Calculate inverse midpoint
                const [b_lat, b_lon] = improvedInverseMidpoint(a_lat, a_lon, m_lat, m_lon);
                
                if (b_lat === null || b_lon === null) {
                    document.getElementById('result').innerHTML = `
                        <h3>Error:</h3>
                        <strong>Cannot calculate reflected point:</strong> 
                        Points are either too close or antipodal (opposite sides of Earth).
                        Please choose different points.
                    `;
                    return;
                }
                
                // Add or move markers
                markerA = addOrMoveMarker(markerA, a_lat, a_lon, {draggable: true, title: "Point A"}, function(e) {
                    const pos = e.target.getLatLng();
                    // Normalize the coordinates when marker is dragged
                    const [normalizedLat, normalizedLng] = normalizeCoordinates(pos.lat, pos.lng);
                    document.getElementById('a_coords').value = `${normalizedLat.toFixed(6)}, ${normalizedLng.toFixed(6)}`;
                    // Update the marker position with normalized coordinates
                    e.target.setLatLng([normalizedLat, normalizedLng]);
                    updateMarkers();
                });
                markerA.bindPopup("Point A").openPopup();
                
                markerM = addOrMoveMarker(markerM, m_lat, m_lon, {draggable: true, title: "Midpoint"}, function(e) {
                    const pos = e.target.getLatLng();
                    // Normalize the coordinates when marker is dragged
                    const [normalizedLat, normalizedLng] = normalizeCoordinates(pos.lat, pos.lng);
                    document.getElementById('m_coords').value = `${normalizedLat.toFixed(6)}, ${normalizedLng.toFixed(6)}`;
                    // Update the marker position with normalized coordinates
                    e.target.setLatLng([normalizedLat, normalizedLng]);
                    updateMarkers();
                });
                markerM.bindPopup("Midpoint");
                
                markerB = addOrMoveMarker(markerB, b_lat, b_lon, {draggable: false, title: "Reflected Point"});
                markerB.bindPopup("Reflected Point");
                
                // Fit map to show all points
                const group = new L.featureGroup([markerA, markerM, markerB]);
                map.fitBounds(group.getBounds().pad(0.3));
                
                // Show result
                document.getElementById('result').innerHTML = `
                    <h3>Results:</h3>
                    <strong>Reflected Point Coordinates:</strong> ${b_lat.toFixed(6)}, ${b_lon.toFixed(6)}<br>
                    <a href="https://www.google.com/maps/place/${b_lat},${b_lon}" target="_blank">View on Google Maps</a>
                `;
            } catch (error) {
                document.getElementById('result').innerHTML = `
                    <h3>Error:</h3>
                    <strong>${error.message}</strong>
                `;
            }
        }

        // Initialize markers on page load
        window.onload = updateMarkers;
    </script>
</body>
</html>
