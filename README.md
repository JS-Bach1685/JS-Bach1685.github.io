<html>
<head>
    <title>Team TPG Tools</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <style>
        #map { height: 500px; margin-bottom: 1em; }
        #midpoint_map { height: 500px; margin: 20px 0; }
        .input-group { margin: 10px 0; }
        label { display: inline-block; width: 120px; }
        body { font-family: Arial, sans-serif; max-width: 1000px; margin: 0 auto; padding: 20px; }
        .leaflet-marker-icon,
        .leaflet-marker-shadow {
            background-color: transparent !important;
            background: transparent !important;
        }
	    
	.leaflet-popup {
           width: max-content !important;
     		}
	.leaflet-popup-content-wrapper {
	  padding: 4px 6px !important;
	  border-radius: 8px !important;
	  background-color: rgba(255, 255, 255, 0.9) !important;
	}

	.leaflet-popup-content {
	  margin: 2px !important;
	  font-size: 12px !important;
	  line-height: 1.2 !important;
	  padding: 1px 9px 1px 1px !important;
	  max-width: 100% !important;
	}
	
 	.leaflet-popup-tip {
	  width: 5px !important;
	  height: 6px !important;
	  margin: -3px auto 0 !important;
	  transform: rotate(45deg) !important;
	}

        .tabs {
            display: flex;
	    overflow-x: auto;
	    white-space: nowrap;
            margin-bottom: 20px;
        }
	
	.tab-label {
	     flex: 0 0 auto;
	     padding: 12px 20px;
	     font-size: 16px;
	}

        .tab {
            padding: 10px 20px;
            background-color: #f0f0f0;
            cursor: pointer;
            border: 1px solid #ccc;
            border-bottom: none;
            margin-right: 5px;
        }
        .tab.active {
            background-color: #fff;
            border-bottom: 1px solid #fff;
            font-weight: bold;
        }
        .tab-content {
            display: none;
            border: 1px solid #ccc;
            padding: 20px;
        }
        .tab-content.active {
            display: block;
        }
        .result-box {
            margin-top: 20px;
            padding: 15px;
            background-color: #f9f9f9;
            border: 1px solid #ddd;
            border-radius: 5px;
        }
        .distance-label {
            white-space: nowrap;
            font-size: 12px;
            background-color: white;
            padding: 3px 6px;
            border: 1px solid #888;
            border-radius: 4px;
            box-shadow: 0 1px 3px rgba(0,0,0,0.2);
        }
        .warning-box {
            margin: 15px 0;
            padding: 12px 15px;
            background-color: #fff3cd;
            color: #856404;
            border: 1px solid #ffeeba;
            border-radius: 5px;
            font-weight: bold;
        }
	@media (max-width: 600px) {
 	 .tab-label {
   	 font-size: 14px;
    	padding: 10px 12px;
  	}
}
    </style>
</head>
<body>
    <h3>A variety of map tools to help with team TPG</h3>
        
    <div class="tabs">
        <div class="tab active" onclick="switchTab('reflect-tab')">Find Reflected Point</div>
        <div class="tab" onclick="switchTab('midpoint-tab')">Calculate Midpoint</div>
	<div class="tab" onclick="switchTab('batch-tab')">Photo Pair Finder</div>
    </div>
     
    <div id="reflect-tab" class="tab-content active">
        <h3>Enter a point and midpoint to find the reflected location on the map.</h3>
        <p>You can also drag & drop Point A or the Midpoint</p>
        <p>Refresh the page if the map looks wonky</p>
        <div class="input-group">
            <h3>Point A</h3>
            <label>Coordinates:</label>
            <input type="text" id="a_coords" value="41.02201, -83.92215" placeholder="lat, lon">
        </div>
        <div class="input-group">
            <h3>Midpoint</h3>
            <label>Coordinates:</label>
            <input type="text" id="m_coords" value="42.48927, -95.54477" placeholder="lat, lon">
        </div>
	<button onclick="updateMarkers()">Calculate Reflected Point</button>
        <div id="distance-warning"></div>
	<div id="map"></div>
        <div id="result" class="result-box"></div>
    </div>
    
    <div id="midpoint-tab" class="tab-content">
        <h3>Calculate the Midpoint Between Two Coordinates</h3>
        <div class="input-group">
            <h3>Point 1</h3>
            <label>Coordinates:</label>
            <input type="text" id="point1_coords" value="" placeholder="lat, lon">
        </div>
        <div class="input-group">
            <h3>Point 2</h3>
            <label>Coordinates:</label>
            <input type="text" id="point2_coords" value="" placeholder="lat, lon">
        </div>
        <div class="input-group">
            <h3>Custom Midpoint (Optional)</h3>
            <label>Coordinates:</label>
            <input type="text" id="custom_midpoint" value="" placeholder="lat, lon (optional)" onchange="updateMidpointMap()">
            <p><small>If provided, distance between calculated midpoint and this point will be shown</small></p>
        </div>
        <button onclick="calculateMidpointWithMap()">Calculate Midpoint</button>
        <div id="midpoint_map"></div>
        <div id="midpoint_result" class="result-box"></div>
    </div>

<div id="batch-tab" class="tab-content">
    <h3>Batch Process Coordinate Pairs to Find Closest Midpoint</h3>
    <p>Upload two CSV files with photo coordinates and find which pair produces a midpoint closest to the target.</p>
    <p><strong>Warning:</strong>This might ruin the fun of the game, use at your own risk! Talk to your partner instead, make a new friend!</p>
        
    <div class="input-group">
        <h3>File 1 (CSV with coordinates)</h3>
        <input type="file" id="file1" accept=".csv">
        <p><small>Format: Each line should contain "latitude,longitude" or "description,latitude,longitude"</small></p>
    </div>
    
    <div class="input-group">
        <h3>File 2 (CSV with coordinates)</h3>
        <input type="file" id="file2" accept=".csv">
        <p><small>Format: Same as File 1</small></p>
    </div>
    
    <div class="input-group">
        <h3>Target Midpoint</h3>
        <label>Coordinates:</label>
        <input type="text" id="target_midpoint" placeholder="lat, lon">
    </div>
    
    <button onclick="processBatchFiles()">Find Best Matches</button>
    
    <div id="batch_progress" style="margin-top: 15px; display: none;">
        <div style="width: 100%; background-color: #f0f0f0; height: 20px; border-radius: 4px; overflow: hidden;">
            <div id="progress_bar" style="width: 0%; background-color: #4CAF50; height: 100%;"></div>
        </div>
        <p id="progress_text">Processing...</p>
    </div>
    
    <div id="batch_map" style="height: 500px; margin: 20px 0; display: none;"></div>
    <div id="batch_result" class="result-box"></div>
</div>

    <!-- Load Leaflet -->
    <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"></script>
    
   <!-- This is just the modified part of the script section -->
<script>
    // Global variables for markers
    let markerA = null;
    let markerM = null;
    let markerB = null;
    let map = null;
    
    // Global variables for midpoint map
    let midpointMap = null;
    let point1Marker = null;
    let point2Marker = null;
    let calculatedMidpointMarker = null;
    let customMidpointMarker = null;
    let distancePolyline = null;
    let distanceLabel = null;
    
    // Constants for Earth dimensions
    const EARTH_RADIUS_MILES = 3958.8; // Earth's radius in miles
    const HALF_EARTH_CIRCUMFERENCE = Math.PI * EARTH_RADIUS_MILES; // Half of Earth's circumference in miles
    
    // Create a red icon for midpoint markers
    function createRedIcon() {
        return new L.Icon({
            iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
            shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
            iconSize: [25, 41],
            iconAnchor: [12, 41],
            popupAnchor: [1, -34],
            shadowSize: [41, 41]
        });
    }

     // Function to create a gold icon for custom midpoint
function createGoldIcon() {
    return new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-gold.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });
}

    
    // Fix Leaflet's default icon paths once for all maps
    function fixLeafletIconPaths() {
        delete L.Icon.Default.prototype._getIconUrl;
        L.Icon.Default.mergeOptions({
            iconRetinaUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon-2x.png',
            iconUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-icon.png',
            shadowUrl: 'https://unpkg.com/leaflet@1.7.1/dist/images/marker-shadow.png'
        });
    }
    
    // Initialize map
    function initMap() {
        if (map !== null) return; // Only initialize once
        
        map = L.map('map', {
            worldCopyJump: true // Helps with the wrapping behavior
        }).setView([32.5, -81.2], 6);
        
        // Add tile layer
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors',
            noWrap: false // Allow the map to repeat horizontally
        }).addTo(map);

        // Fix icon paths
        fixLeafletIconPaths();
    }
    
    // Initialize midpoint map
    function initMidpointMap() {
        if (midpointMap !== null) return; // Only initialize once
        
        midpointMap = L.map('midpoint_map', {
            worldCopyJump: true // Helps with the wrapping behavior
        }).setView([32.5, -81.2], 3);
        
        // Add tile layer
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors',
            noWrap: false // Allow the map to repeat horizontally
        }).addTo(midpointMap);

        // Fix icon paths
        fixLeafletIconPaths();
    }

    // Tab switching function
    function switchTab(tabId) {
        // Hide all tab contents
        document.querySelectorAll('.tab-content').forEach(content => {
            content.classList.remove('active');
        });
        
        // Remove active class from all tabs
        document.querySelectorAll('.tab').forEach(tab => {
            tab.classList.remove('active');
        });
        
        // Show the selected tab content
        document.getElementById(tabId).classList.add('active');
        
        // Add active class to the clicked tab
        Array.from(document.querySelectorAll('.tab')).find(tab => 
            tab.textContent.toLowerCase().includes(tabId.split('-')[0])
        ).classList.add('active');
        
        // Initialize appropriate map
        if (tabId === 'reflect-tab') {
            setTimeout(() => {
                initMap();
                updateMarkers();
            }, 100);
        } else if (tabId === 'midpoint-tab') {
            setTimeout(() => {
                initMidpointMap();
                // Update the map if we already have data
                if (document.getElementById('point1_coords').value && 
                    document.getElementById('point2_coords').value) {
                    calculateMidpointWithMap();
                }
            }, 100);
        } else if (tabId === 'batch-tab') {
            setTimeout(() => {
                // We'll initialize batch map only when needed
                document.getElementById('batch_map').style.display = 'none';
            }, 100);
        }
    }

    // Function to parse coordinates from input string
    function parseCoordinates(coordString) {
        if (!coordString || coordString.trim() === '') {
            throw new Error("Coordinates cannot be empty");
        }
        
        const parts = coordString.split(',').map(v => parseFloat(v.trim()));
        
        if (parts.length !== 2 || isNaN(parts[0]) || isNaN(parts[1])) {
            throw new Error("Invalid coordinates format. Please use 'latitude, longitude'");
        }
        
        return parts;
    }

    // Function to normalize coordinates
    function normalizeCoordinates(lat, lng) {
        // Constrain latitude to -90 to 90
        lat = Math.max(-90, Math.min(90, lat));
        
        // Normalize longitude to -180 to 180
        lng = ((lng + 540) % 360) - 180;
        
        return [lat, lng];
    }

    // Calculate midpoint using the geomidpoint.com method (center of gravity)
    function calculateGeographicMidpoint(coords) {
        const toRad = deg => deg * Math.PI / 180;
        const toDeg = rad => rad * 180 / Math.PI;
        
        // Convert to Cartesian coordinates
        let x = 0, y = 0, z = 0;
        
        for (const [lat, lon] of coords) {
            const phi = toRad(lat);
            const lambda = toRad(lon);
            
            // Convert to Cartesian coordinates
            x += Math.cos(phi) * Math.cos(lambda);
            y += Math.cos(phi) * Math.sin(lambda);
            z += Math.sin(phi);
        }
        
        // Get averages
        x /= coords.length;
        y /= coords.length;
        z /= coords.length;
        
        // Convert back to spherical coordinates
        const lambda = Math.atan2(y, x);
        const hyp = Math.sqrt(x * x + y * y);
        const phi = Math.atan2(z, hyp);
        
        // Convert to degrees
        const midLat = toDeg(phi);
        const midLon = toDeg(lambda);
        
        return normalizeCoordinates(midLat, midLon);
    }

    // Function to calculate distance between two points (in miles)
    function calculateDistance(lat1, lon1, lat2, lon2) {
        const R = EARTH_RADIUS_MILES; // Earth's radius in miles
        const dLat = (lat2 - lat1) * Math.PI / 180;
        const dLon = (lon2 - lon1) * Math.PI / 180;
        
        const a = Math.sin(dLat/2) * Math.sin(dLat/2) +
                  Math.cos(lat1 * Math.PI / 180) * Math.cos(lat2 * Math.PI / 180) * 
                  Math.sin(dLon/2) * Math.sin(dLon/2);
        
        const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
        const distance = R * c;
        
        return distance;
    }
    
    // Function to check if distance exceeds half of Earth's circumference
    function isDistanceTooLarge(lat1, lon1, lat2, lon2) {
        const distance = calculateDistance(lat1, lon1, lat2, lon2);
        return {
            isTooLarge: distance > HALF_EARTH_CIRCUMFERENCE / 2, // Half of half circumference (quarter of full)
            distance: distance
        };
    }
    
   // Function to add or update a marker on the midpoint map
function addOrUpdateMidpointMarker(marker, lat, lon, title, useRedIcon = false, customIcon = null) {
    let options = {};
    
    // Use specified icon if provided, otherwise use red icon if specified
    if (customIcon) {
        options.icon = customIcon;
    } else if (useRedIcon) {
        options.icon = createRedIcon();
    }
    
    if (marker) {
        marker.setLatLng([lat, lon]);
        
        // Update icon if we're changing the icon
        if ((useRedIcon && !marker.options.icon) || customIcon) {
            marker.setIcon(customIcon || createRedIcon());
        }
    } else {
        marker = L.marker([lat, lon], options);
        marker.addTo(midpointMap);
    }
    marker.bindPopup(title);
    return marker;
}
    
    // Improved function to create a distance label that's always on one line
    function createDistanceLabel(midLat, midLon, distance) {
        // Remove existing distance label if it exists
        if (distanceLabel) {
            midpointMap.removeLayer(distanceLabel);
        }
        
        // Format the distance with appropriate precision
        const formattedDistance = distance.toFixed(2);
        
        // Create a div icon with nowrap styling
        const labelIcon = L.divIcon({
            className: 'custom-label', // This is ignored, we'll use the html styling
            html: `<div class="distance-label">${formattedDistance} miles</div>`,
            iconSize: [null, null], // Auto-size based on content
            iconAnchor: [50, 10] // Centered horizontally
        });
        
        // Create the marker with the label
        distanceLabel = L.marker([midLat, midLon], {
            icon: labelIcon,
            interactive: false, // Make it non-interactive (can't be clicked)
            keyboard: false
        }).addTo(midpointMap);
        
        return distanceLabel;
    }
    
    // Function to update the midpoint map display
    function updateMidpointMap() {
        try {
            // Make sure the midpoint map is initialized
            initMidpointMap();
                            
            // Check if we have the necessary data
            const point1Value = document.getElementById('point1_coords').value.trim();
            const point2Value = document.getElementById('point2_coords').value.trim();
            
            if (!point1Value || !point2Value) {
                return; // Not enough data to update map
            }
            
            // Parse point coordinates
            let [lat1, lon1] = parseCoordinates(point1Value);
            let [lat2, lon2] = parseCoordinates(point2Value);
            
            // Normalize coordinates
            [lat1, lon1] = normalizeCoordinates(lat1, lon1);
            [lat2, lon2] = normalizeCoordinates(lat2, lon2);
            
            // Calculate midpoint
            const [midLat, midLon] = calculateGeographicMidpoint([[lat1, lon1], [lat2, lon2]]);
            
            // Update or add markers - use red icon for midpoint
            point1Marker = addOrUpdateMidpointMarker(point1Marker, lat1, lon1, "Point 1", false);
            point2Marker = addOrUpdateMidpointMarker(point2Marker, lat2, lon2, "Point 2", false);
            calculatedMidpointMarker = addOrUpdateMidpointMarker(calculatedMidpointMarker, midLat, midLon, "Calculated Midpoint", true);
            
            // Check for custom midpoint
            const customMidpointValue = document.getElementById('custom_midpoint').value.trim();
            
            // Remove existing polyline
            if (distancePolyline) {
                midpointMap.removeLayer(distancePolyline);
                distancePolyline = null;
            }
            
            // Remove existing distance label
            if (distanceLabel) {
                midpointMap.removeLayer(distanceLabel);
                distanceLabel = null;
            }
            
            if (customMidpointValue) {
                try {
                    let [customLat, customLon] = parseCoordinates(customMidpointValue);
                    [customLat, customLon] = normalizeCoordinates(customLat, customLon);
                    
                    // Add or update custom midpoint marker with gold icon
customMidpointMarker = addOrUpdateMidpointMarker(customMidpointMarker, customLat, customLon, "Custom Midpoint", false, createGoldIcon());
                    
                    // Calculate distance
                    const distanceMiles = calculateDistance(midLat, midLon, customLat, customLon);
                    
                    // Add polyline
                    distancePolyline = L.polyline(
                        [[midLat, midLon], [customLat, customLon]], 
                        { color: 'red', weight: 3 }
                    ).addTo(midpointMap);
                    
                    // Add improved distance label
                    const midPoint = [
                        (midLat + customLat) / 2,
                        (midLon + customLon) / 2
                    ];
                    
                    createDistanceLabel(midPoint[0], midPoint[1], distanceMiles);
                    
                } catch (err) {
                    console.error("Error with custom midpoint:", err);
                    if (customMidpointMarker) {
                        midpointMap.removeLayer(customMidpointMarker);
                        customMidpointMarker = null;
                    }
                }
            } else {
                // If no custom midpoint is provided, remove the custom midpoint marker
                if (customMidpointMarker) {
                    midpointMap.removeLayer(customMidpointMarker);
                    customMidpointMarker = null;
                }
            }
            
            // Create a group with all valid markers to fit the map view
            const markersToInclude = [point1Marker, point2Marker, calculatedMidpointMarker];
            if (customMidpointMarker) markersToInclude.push(customMidpointMarker);
            
            const group = new L.featureGroup(markersToInclude);
            midpointMap.fitBounds(group.getBounds().pad(0.3));
            
        } catch (error) {
            console.error("Error updating midpoint map:", error);
            document.getElementById('midpoint_result').innerHTML = `
                <h3>Error:</h3>
                <strong>${error.message}</strong>
            `;
        }
    }

    // Function to handle midpoint calculation with map update
    function calculateMidpointWithMap() {
        try {
            // This calls the original calculation function
            calculateMidpoint();
            
            // Then updates the map
            updateMidpointMap();
            
        } catch (error) {
            console.error("Error calculating midpoint with map:", error);
        }
    }

    // Function to handle midpoint calculation
    function calculateMidpoint() {
        try {
            // Get input values
            let [lat1, lon1] = parseCoordinates(document.getElementById('point1_coords').value);
            let [lat2, lon2] = parseCoordinates(document.getElementById('point2_coords').value);
            
            // Normalize coordinates
            [lat1, lon1] = normalizeCoordinates(lat1, lon1);
            [lat2, lon2] = normalizeCoordinates(lat2, lon2);
            
            // Update input fields with normalized values
            document.getElementById('point1_coords').value = `${lat1.toFixed(6)}, ${lon1.toFixed(6)}`;
            document.getElementById('point2_coords').value = `${lat2.toFixed(6)}, ${lon2.toFixed(6)}`;
            
            // Calculate midpoint
            const [midLat, midLon] = calculateGeographicMidpoint([[lat1, lon1], [lat2, lon2]]);
            
            let resultHTML = `
                <h3>Midpoint Results:</h3>
                <strong>Calculated Midpoint:</strong> ${midLat.toFixed(6)}, ${midLon.toFixed(6)}<br>
                <a href="https://www.google.com/maps/place/${midLat},${midLon}" target="_blank">View on Google Maps</a><br><br>
                <strong>Distance from Point 1 to Point 2:</strong> ${calculateDistance(lat1, lon1, lat2, lon2).toFixed(2)} miles<br>
            `;
            
            // Check if custom midpoint was provided
            const customMidpointValue = document.getElementById('custom_midpoint').value.trim();
            if (customMidpointValue) {
                try {
                    let [customLat, customLon] = parseCoordinates(customMidpointValue);
                    
                    [customLat, customLon] = normalizeCoordinates(customLat, customLon);
                    document.getElementById('custom_midpoint').value = `${customLat.toFixed(6)}, ${customLon.toFixed(6)}`;
                    
                    const customDistance = calculateDistance(midLat, midLon, customLat, customLon);
                    resultHTML += `<br><strong>Distance between calculated midpoint and custom midpoint:</strong> ${customDistance.toFixed(2)} miles`;
                } catch (err) {
                    resultHTML += `<br><span style="color: red;">Error with custom midpoint: ${err.message}</span>`;
                }
            }
            
            document.getElementById('midpoint_result').innerHTML = resultHTML;
            
        } catch (error) {
            document.getElementById('midpoint_result').innerHTML = `
                <h3>Error:</h3>
                <strong>${error.message}</strong>
            `;
        }
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
            
            // Update icon if specified in options and different from current
            if (options.icon && (!marker.options.icon || marker.options.icon !== options.icon)) {
                marker.setIcon(options.icon);
            }
        } else {
            marker = L.marker([lat, lon], options);
            if (onDragEnd) marker.on('dragend', onDragEnd);
            marker.addTo(map);
        }
        return marker;
    }

    // Update markers for the reflection calculator
    function updateMarkers() {
        try {
            // Initialize map if not already initialized
            initMap();
            
            // Clear any existing warning
            document.getElementById('distance-warning').innerHTML = '';
            
            // Get input values
            try {
                var [a_lat, a_lon] = parseCoordinates(document.getElementById('a_coords').value);
                var [m_lat, m_lon] = parseCoordinates(document.getElementById('m_coords').value);
            } catch (error) {
                document.getElementById('result').innerHTML = `
                    <h3>Error:</h3>
                    <strong>${error.message}</strong>
                `;
                return;
            }
            
            // Normalize the input coordinates
            [a_lat, a_lon] = normalizeCoordinates(a_lat, a_lon);
            [m_lat, m_lon] = normalizeCoordinates(m_lat, m_lon);
            
            // Update the input fields with normalized values
            document.getElementById('a_coords').value = `${a_lat.toFixed(6)}, ${a_lon.toFixed(6)}`;
            document.getElementById('m_coords').value = `${m_lat.toFixed(6)}, ${m_lon.toFixed(6)}`;
            
            // Check if distance is too large (more than a quarter of Earth's circumference)
            const { isTooLarge, distance } = isDistanceTooLarge(a_lat, a_lon, m_lat, m_lon);
            
            if (isTooLarge) {
                // Show warning
                document.getElementById('distance-warning').innerHTML = `
                    <div class="warning-box">
                        <strong>Warning:</strong> The distance between Point A and Midpoint (${distance.toFixed(0)} miles) 
                        is very large. For points this far apart, the reflected point calculation may not be what you expect.
                        The true shortest path midpoint would be along a different great circle path.
                    </div>
                `;
            }
            
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
            
            // Create a red icon for the midpoint
            const redIcon = createRedIcon();
            
            // Add or move markers - use red icon for midpoint
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
            
            markerM = addOrMoveMarker(markerM, m_lat, m_lon, {draggable: true, title: "Midpoint", icon: redIcon}, function(e) {
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

// Global variables for batch processing
let batchMap = null;
let batchResults = [];
let batchMarkers = [];

// Initialize batch processing map
function initBatchMap() {
    if (batchMap !== null) return; // Only initialize once
    
    batchMap = L.map('batch_map', {
        worldCopyJump: true
    }).setView([32.5, -81.2], 3);
    
    // Add tile layer
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
        attribution: '&copy; OpenStreetMap contributors',
        noWrap: false
    }).addTo(batchMap);

    // Fix icon paths
    fixLeafletIconPaths();
}

// Function to parse CSV file content
function parseCSVCoordinates(csvContent) {
    const lines = csvContent.split(/\r?\n/);
    const coordinates = [];
    
    for (let i = 0; i < lines.length; i++) {
        const line = lines[i].trim();
        if (!line) continue;
        
        // Try to extract coordinates - handle both formats:
        // "lat,lon" or "name,lat,lon"
        const parts = line.split(',').map(p => p.trim());
        
        try {
            let lat, lon, name;
            
            if (parts.length === 2) {
                // Just coordinates: "lat,lon"
                [lat, lon] = [parseFloat(parts[0]), parseFloat(parts[1])];
                name = `Point ${i+1}`;
            } else if (parts.length >= 3) {
                // Format with name: "name,lat,lon"
                name = parts[0];
                lat = parseFloat(parts[1]);
                lon = parseFloat(parts[2]);
            } else {
                continue; // Skip invalid lines
            }
            
            if (isNaN(lat) || isNaN(lon)) continue;
            
            // Normalize coordinates
            [lat, lon] = normalizeCoordinates(lat, lon);
            
            coordinates.push({
                name: name,
                lat: lat,
                lon: lon
            });
        } catch (e) {
            console.warn(`Error parsing line ${i+1}: ${line}`);
        }
    }
    
    return coordinates;
}

// Function to process batch files
function processBatchFiles() {
    // Clear previous results
    batchResults = [];
    batchMarkers.forEach(marker => {
        if (batchMap) batchMap.removeLayer(marker);
    });
    batchMarkers = [];
    
    document.getElementById('batch_result').innerHTML = '';
    
    // Get files and target coordinates
    const file1 = document.getElementById('file1').files[0];
    const file2 = document.getElementById('file2').files[0];
    const targetMidpointStr = document.getElementById('target_midpoint').value.trim();
    
    // Validate inputs
    if (!file1 || !file2) {
        document.getElementById('batch_result').innerHTML = '<strong>Error:</strong> Please upload both CSV files.';
        return;
    }
    
    if (!targetMidpointStr) {
        document.getElementById('batch_result').innerHTML = '<strong>Error:</strong> Please enter target midpoint coordinates.';
        return;
    }
    
    try {
        var [targetLat, targetLon] = parseCoordinates(targetMidpointStr);
        [targetLat, targetLon] = normalizeCoordinates(targetLat, targetLon);
    } catch (error) {
        document.getElementById('batch_result').innerHTML = `<strong>Error:</strong> ${error.message}`;
        return;
    }
    
    // Show progress bar
    document.getElementById('batch_progress').style.display = 'block';
    document.getElementById('progress_bar').style.width = '0%';
    document.getElementById('progress_text').innerText = 'Reading files...';
    
    // Process files
    Promise.all([
        readFileAsText(file1),
        readFileAsText(file2)
    ]).then(([content1, content2]) => {
        // Parse coordinates from both files
        const coords1 = parseCSVCoordinates(content1);
        const coords2 = parseCSVCoordinates(content2);
        
        if (coords1.length === 0 || coords2.length === 0) {
            throw new Error('Could not parse coordinates from one or both files. Check format.');
        }
        
        document.getElementById('progress_text').innerText = `Processing ${coords1.length} × ${coords2.length} = ${coords1.length * coords2.length} possible pairs...`;
        
        // Process in batches to avoid freezing the UI
        processPairsInBatches(coords1, coords2, targetLat, targetLon, 0, 0, 100);
    }).catch(error => {
        document.getElementById('batch_progress').style.display = 'none';
        document.getElementById('batch_result').innerHTML = `<strong>Error:</strong> ${error.message}`;
    });
}

// Helper function to read file as text
function readFileAsText(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = event => resolve(event.target.result);
        reader.onerror = error => reject(error);
        reader.readAsText(file);
    });
}

// Process pairs in batches to avoid UI freeze
function processPairsInBatches(coords1, coords2, targetLat, targetLon, i, j, batchSize) {
    const totalPairs = coords1.length * coords2.length;
    const startTime = performance.now();
    let pairsProcessed = 0;
    
    while (i < coords1.length && pairsProcessed < batchSize) {
        while (j < coords2.length && pairsProcessed < batchSize) {
            const coord1 = coords1[i];
            const coord2 = coords2[j];
            
            // Calculate midpoint
            const [midLat, midLon] = calculateGeographicMidpoint([
                [coord1.lat, coord1.lon],
                [coord2.lat, coord2.lon]
            ]);
            
            // Calculate distance to target
            const distanceToTarget = calculateDistance(midLat, midLon, targetLat, targetLon);
            
            batchResults.push({
                point1: coord1,
                point2: coord2,
                midpoint: { lat: midLat, lon: midLon },
                distance: distanceToTarget
            });
            
            pairsProcessed++;
            j++;
        }
        
        if (j >= coords2.length) {
            j = 0;
            i++;
        }
    }
    
    // Update progress
    const processedSoFar = Math.min(i * coords2.length + j, totalPairs);
    const percentComplete = Math.round((processedSoFar / totalPairs) * 100);
    document.getElementById('progress_bar').style.width = percentComplete + '%';
    document.getElementById('progress_text').innerText = `Processed ${processedSoFar} of ${totalPairs} pairs (${percentComplete}%)`;
    
    if (i < coords1.length) {
        // Continue processing in the next batch
        setTimeout(() => {
            processPairsInBatches(coords1, coords2, targetLat, targetLon, i, j, batchSize);
        }, 0);
    } else {
        // All done, show results
        displayBatchResults(targetLat, targetLon);
    }
}

// Display batch processing results
function displayBatchResults(targetLat, targetLon) {
    document.getElementById('batch_progress').style.display = 'none';
    
    if (batchResults.length === 0) {
        document.getElementById('batch_result').innerHTML = '<strong>No valid coordinate pairs found.</strong>';
        return;
    }
    
    // Sort results by distance
    batchResults.sort((a, b) => a.distance - b.distance);
    
    // Take top 5 results
    const topResults = batchResults.slice(0, 5);
    
    // Initialize map
    document.getElementById('batch_map').style.display = 'block';
    initBatchMap();
    
    // Create result HTML
    let resultHTML = `
       <h3>Top 5 Closest Matches:</h3>
        <table style="width: 100%; border-collapse: collapse; margin-top: 10px; font-size: 14px;">
            <tr>
                <th style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">Rank</th>
                <th style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">Point 1</th>
                <th style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">Point 2</th>
                <th style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">Midpoint</th>
                <th style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">Distance to Target</th>
            </tr>
    `;
   
    // Add markers for target point
    const targetIcon = new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });
    
    const targetMarker = L.marker([targetLat, targetLon], { icon: targetIcon })
        .bindPopup("Target Midpoint")
        .addTo(batchMap);
    
    batchMarkers.push(targetMarker);
    
    // Add markers for top results
    topResults.forEach((result, index) => {
        const { point1, point2, midpoint, distance } = result;
        
        resultHTML += `
            <tr>
                <td style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">${index + 1}</td>
                <td style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">
                    <div>${point1.name}</div>
                    <div><small>(${point1.lat.toFixed(6)}, ${point1.lon.toFixed(6)})</small></div>
                </td>
                <td style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">
                    <div>${point2.name}</div>
                    <div><small>(${point2.lat.toFixed(6)}, ${point2.lon.toFixed(6)})</small></div>
                </td>
                <td style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">${midpoint.lat.toFixed(6)}, ${midpoint.lon.toFixed(6)}</td>
                <td style="padding: 8px; text-align: left; border-bottom: 1px solid #ddd;">${distance.toFixed(2)} miles</td>
            </tr>
        `;
        
        // Only add markers for top 3 results to avoid clutter
        if (index < 3) {
            // Use different colors for top 3
            const colors = ['gold', 'grey', 'orange'];
            const color = colors[index];
            
            // Point 1 marker
            const marker1 = L.marker([point1.lat, point1.lon])
                .bindPopup(`${point1.name} (Pair #${index + 1})`)
                .addTo(batchMap);
            
            // Point 2 marker
            const marker2 = L.marker([point2.lat, point2.lon])
                .bindPopup(`${point2.name} (Pair #${index + 1})`)
                .addTo(batchMap);
            
            // Midpoint marker
            const midpointIcon = new L.Icon({
                iconUrl: `https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-${color}.png`,
                shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
                iconSize: [25, 41],
                iconAnchor: [12, 41],
                popupAnchor: [1, -34],
                shadowSize: [41, 41]
            });
            
            const midMarker = L.marker([midpoint.lat, midpoint.lon], { icon: midpointIcon })
                .bindPopup(`Midpoint #${index + 1} - ${distance.toFixed(2)} miles from target`)
                .addTo(batchMap);
            
            // Add lines connecting the points
            const polyline = L.polyline([
                [point1.lat, point1.lon],
                [midpoint.lat, midpoint.lon],
                [point2.lat, point2.lon]
            ], { color: color, weight: 2 }).addTo(batchMap);
            
            // Add distance line to target
            const targetLine = L.polyline([
                [midpoint.lat, midpoint.lon],
                [targetLat, targetLon]
            ], { color: 'red', weight: 2, dashArray: '5, 5' }).addTo(batchMap);
            
            batchMarkers.push(marker1, marker2, midMarker, polyline, targetLine);
        }
    });
    
    resultHTML += '</>';
    document.getElementById('batch_result').innerHTML = resultHTML;
    
    // Fit map to markers
    const group = new L.featureGroup(batchMarkers);
    batchMap.fitBounds(group.getBounds().pad(0.3));
}

// Update tab switching function to include batch tab
function switchTab(tabId) {
    // Hide all tab contents
    document.querySelectorAll('.tab-content').forEach(content => {
        content.classList.remove('active');
    });
    
    // Remove active class from all tabs
    document.querySelectorAll('.tab').forEach(tab => {
        tab.classList.remove('active');
    });
    
    // Show the selected tab content
    document.getElementById(tabId).classList.add('active');
    
    // Add active class to the clicked tab
    if (tabId === 'reflect-tab') {
        document.querySelector('.tab:nth-child(1)').classList.add('active');
    } else if (tabId === 'midpoint-tab') {
        document.querySelector('.tab:nth-child(2)').classList.add('active');
    } else if (tabId === 'batch-tab') {
        document.querySelector('.tab:nth-child(3)').classList.add('active');
    }
    
    // Initialize appropriate map
    if (tabId === 'reflect-tab') {
        setTimeout(() => {
            initMap();
            updateMarkers();
        }, 100);
    } else if (tabId === 'midpoint-tab') {
        setTimeout(() => {
            initMidpointMap();
            // Update the map if we already have data
            if (document.getElementById('point1_coords').value && 
                document.getElementById('point2_coords').value) {
                calculateMidpointWithMap();
            }
        }, 100);
    } else if (tabId === 'batch-tab') {
        setTimeout(() => {
            // We'll initialize batch map only when needed
            document.getElementById('batch_map').style.display = 'none';
        }, 100);
    }
}

        // Initialize tabs and default functionality on page load
        window.onload = function() {
            // Fix Leaflet icon paths
            fixLeafletIconPaths();
            
            // Initialize the default tab
            switchTab('reflect-tab');
        };
    </script>
        <footer style="text-align: center; padding: 20px; font-size: 12px;">
  <p>Found a bug or have a feature idea? Message me on Discord:
  <a href="https://discord.com/users/g3mbert" target="_blank" style="text-decoration: none; color: inherit;"> 
  @g3mbert </a>
     </p>
</footer>
</body>
</html>
