<html>
<head>
    <title>Team TPG Tools</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css" />
    <style>
        #map { 
	     height: 500px;
	     margin-top: 1em 
	     margin-bottom: 1em; 
	}
	#map.warning {
	  border: 4px solid #ff4d4f;
	  box-shadow: 0 0 15px 6px rgba(255, 77, 79, 0.5);
	  transition: box-shadow 0.3s ease, border 0.3s ease;
	}
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
    <script src="https://cdn.jsdelivr.net/npm/papaparse@5.4.1/papaparse.min.js"></script>
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
	<p>Endpoints can be dragged</p>
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
    <h3>Photo Pair Finder - Batch Process Coordinate Pairs</h3>
    <p>Upload two CSV files with photo coordinates and find which pair produces a midpoint closest to the target.</p>
    <p>Now accepts exported Voronoi Map CSVs!</p>
    <p><strong>Warning:</strong> Both CSVs should have the same format to avoid errors.</p>
    
    <div class="input-group">
        <h4>CSV Format Settings</h4>
        <div style="display: flex; align-items: center; margin-bottom: 10px;">
            <input type="checkbox" id="auto_detect" checked style="margin-right: 10px;">
            <label for="auto_detect" style="width: auto; margin-right: 15px;"><strong>Auto-detect coordinate columns</strong></label>
            <span style="font-size: 12px; color: #666;">(Recommended for most files)</span>
        </div>
        
        <div id="manual_columns" style="display: none;">
            <div style="margin-bottom: 10px;">
                <label for="lat_column">Latitude Column Index:</label>
                <input type="number" id="lat_column" value="0" min="0" style="width: 60px;">
                <span style="margin-left: 15px; font-size: 12px; color: #666;">(First column is index 0)</span>
            </div>
            <div>
                <label for="lon_column">Longitude Column Index:</label>
                <input type="number" id="lon_column" value="1" min="0" style="width: 60px;">
            </div>

<p><small>Specify which columns contain the latitude and longitude values. Default is 0,1 for the voronoi map exported csv.</small></p>

<div style="display: flex; align-items: center; margin-bottom: 10px; margin-top: 10px;">
    <input type="checkbox" id="has_headers" checked style="margin-right: 10px;">
    <label for="has_headers" style="width: auto; margin-right: 15px;"><strong>Files have headers</strong></label>
    <span style="font-size: 12px; color: #666;">(Uncheck if your CSV files don't have column names in the first row)</span>
</div>

	<div style="margin-bottom: 10px;">
    <label for="desc_column">Description Column Index:</label>
    <input type="number" id="desc_column" value="2" min="0" style="width: 60px;">
    <span style="margin-left: 15px; font-size: 12px; color: #666;">(Column to use for point description)</span>
</div>
            
        </div>
    </div>
        
    <div class="input-group">
        <h3>File 1 (CSV with coordinates)</h3>
        <input type="file" id="file1" accept=".csv">
        <p><small>CSV can include additional columns for descriptions, rounds, URLs, etc.</small></p>
    </div>
    
    <div class="input-group">
        <h3>File 2 (CSV with coordinates)</h3>
        <input type="file" id="file2" accept=".csv">
        <p><small>Same format as File 1</small></p>
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
    const tabContent = document.getElementById(tabId);
    if (tabContent) tabContent.classList.add('active');

    // Add active class to the correct tab using a matching ID-to-label map
    const tabMap = {
        'reflect-tab': 'Find Reflected Point',
        'midpoint-tab': 'Calculate Midpoint',
        'batch-tab': 'Photo Pair Finder'
    };

    document.querySelectorAll('.tab').forEach(tab => {
        if (tab.textContent.trim() === tabMap[tabId]) {
            tab.classList.add('active');
        }
    });

    // Initialize the appropriate map with a delay
    setTimeout(() => {
        if (tabId === 'reflect-tab') {
            initMap();
            updateMarkers();
        } else if (tabId === 'midpoint-tab') {
            initMidpointMap();
            if (document.getElementById('point1_coords').value && document.getElementById('point2_coords').value) {
                calculateMidpointWithMap();
            }
        } else if (tabId === 'batch-tab') {
            document.getElementById('batch_map').style.display = 'none';
        }
    }, 100);
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
    let options = {
        draggable: false
    };

   // Make points 1 and 2 draggable, but not the midpoints
    if (title === "Point 1" || title === "Point 2") {
        options.draggable = true;
    }
    
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

 // Update draggable property if it changed
        if (marker.options.draggable !== options.draggable) {
            // We need to recreate the marker if draggable property changes
            marker.remove();
            marker = L.marker([lat, lon], options);
            marker.addTo(midpointMap);
        }

    } else {
        marker = L.marker([lat, lon], options);
        marker.addTo(midpointMap);
    }
    marker.bindPopup(title);
    return marker;
}
    
// Throttle function to reduce number of updates during dragging
function throttle(func, limit) {
    let inThrottle;
    return function() {
        const args = arguments;
        const context = this;
        if (!inThrottle) {
            func.apply(context, args);
            inThrottle = true;
            setTimeout(() => inThrottle = false, limit);
        }
    }
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
   
// Add drag event listeners to recalculate midpoint
        if (point1Marker) {
            // Remove existing listeners first to avoid duplicates
            point1Marker.off('dragend');
            point1Marker.off('drag');
            
            // Add new listeners
            point1Marker.on('dragend', function(e) {
                const pos = e.target.getLatLng();
                document.getElementById('point1_coords').value = `${pos.lat.toFixed(6)}, ${pos.lng.toFixed(6)}`;
                calculateMidpointWithMap();
            });
            
            point1Marker.on('drag', throttle(function(e) {
                const pos = e.target.getLatLng();
                document.getElementById('point1_coords').value = `${pos.lat.toFixed(6)}, ${pos.lng.toFixed(6)}`;
            }, 100));
        }
        
        if (point2Marker) {
            // Remove existing listeners first to avoid duplicates
            point2Marker.off('dragend');
            point2Marker.off('drag');
            
            // Add new listeners
            point2Marker.on('dragend', function(e) {
                const pos = e.target.getLatLng();
                document.getElementById('point2_coords').value = `${pos.lat.toFixed(6)}, ${pos.lng.toFixed(6)}`;
                calculateMidpointWithMap();
            });
            
            point2Marker.on('drag', throttle(function(e) {
                const pos = e.target.getLatLng();
                document.getElementById('point2_coords').value = `${pos.lat.toFixed(6)}, ${pos.lng.toFixed(6)}`;
            }, 100));
        }

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

const warningBox = document.getElementById('distance-warning');
const mapElement = document.getElementById('map');

if (isTooLarge) {
    // Show warning
    warningBox.innerHTML = `
        <div class="warning-box">
            <strong>Warning:</strong> The distance between Point A and the Midpoint (${distance.toFixed(0)} miles) 
            is very large. For points this far apart the true shortest path midpoint would be the antipode of the current midpoint.
        </div>
    `;

    // Add red glow to the map
    mapElement.classList.add('warning');

    // Optional: vibrate on mobile
    if ('vibrate' in navigator) navigator.vibrate(200);

} else {
    // Clear warning
    warningBox.innerHTML = '';
    
    // Remove red glow
    mapElement.classList.remove('warning');
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

// Add this code to your existing JavaScript section
document.addEventListener('DOMContentLoaded', function() {
    // Add event listener for auto-detect checkbox
    const autoDetectCheckbox = document.getElementById('auto_detect');
    const manualColumnsDiv = document.getElementById('manual_columns');
    
    if (autoDetectCheckbox && manualColumnsDiv) {
        autoDetectCheckbox.addEventListener('change', function() {
            manualColumnsDiv.style.display = this.checked ? 'none' : 'block';
        });
    }
});
/**
 * More robust function to determine if a CSV has headers
 * @param {string} csvContent - The content of the CSV file
 * @returns {boolean} - True if the CSV likely has headers, false otherwise
 */
function detectCSVHeaders(csvContent) {
    const lines = csvContent.split(/\r?\n/).filter(line => line.trim());
    if (lines.length < 2) return false; // Need at least 2 lines to compare
    
    const firstLine = lines[0].split(',').map(item => item.trim());
    const secondLine = lines[1].split(',').map(item => item.trim());
    
    // If first line has different number of columns than second, it's likely a header
    if (firstLine.length !== secondLine.length) return true;
    
    // Check if first line has any non-numeric values while second line is all numeric
    const firstLineHasNonNumeric = firstLine.some(item => isNaN(parseFloat(item)));
    const secondLineAllNumeric = secondLine.every(item => !isNaN(parseFloat(item)));
    
    if (firstLineHasNonNumeric && secondLineAllNumeric) return true;
    
    // Check for common header words in first line
    const headerKeywords = ['name', 'id', 'title', 'label', 'description', 'desc', 'lat', 'lon', 'lng', 
                           'latitude', 'longitude', 'location', 'address', 'city', 'state', 'country', 
                           'zip', 'postal', 'code', 'date', 'time', 'year', 'month', 'day'];
    
    const firstLineHasHeaderWords = firstLine.some(item => 
        headerKeywords.some(keyword => 
            item.toLowerCase().includes(keyword.toLowerCase())
        )
    );
    
    // If we detect header keywords in the first line, it's likely a header
    if (firstLineHasHeaderWords) return true;
    
    // Compare data patterns between first row and other rows
    // If first row pattern is significantly different, it might be a header
    let firstRowNumericCount = 0;
    let otherRowsNumericCount = 0;
    let rowsChecked = 0;
    
    firstLine.forEach(item => {
        if (!isNaN(parseFloat(item))) firstRowNumericCount++;
    });
    
    // Check up to 5 more rows
    for (let i = 1; i < Math.min(lines.length, 6); i++) {
        const rowItems = lines[i].split(',').map(item => item.trim());
        let rowNumericCount = 0;
        
        rowItems.forEach(item => {
            if (!isNaN(parseFloat(item))) rowNumericCount++;
        });
        
        otherRowsNumericCount += rowNumericCount;
        rowsChecked++;
    }
    
    const avgOtherRowsNumeric = otherRowsNumericCount / rowsChecked;
    
    // If first row has significantly less numeric values than others, likely a header
    if (firstRowNumericCount < avgOtherRowsNumeric * 0.7) return true;
    
    // Default to assuming no headers if none of the above checks passed
    return false;
}
  
/**
 * Automatically detects the latitude and longitude columns in a CSV file.
 * @param {string} csvContent - The content of the CSV file as a string
 * @returns {Object} - Object containing the detected latitude and longitude column indices
 */
function autoDetectCoordinateColumns(csvContent) {
    if (!csvContent || typeof csvContent !== 'string') {
        return { latIndex: 0, lonIndex: 1 }; // Default values if input is invalid
    }

    // Split content into lines and get header row if it exists
    const lines = csvContent.split(/\r?\n/);
    if (lines.length < 2) {
        return { latIndex: 0, lonIndex: 1 }; // Not enough data, return defaults
    }

    // Check if we have headers (first attempt to parse first line as numbers)
    const firstLineParts = lines[0].split(',').map(p => p.trim());
    const hasHeaders = firstLineParts.some(part => isNaN(parseFloat(part)));

    // Get column headers if they exist, otherwise use indices as headers
    const headers = hasHeaders ? firstLineParts : Array.from({ length: firstLineParts.length }, (_, i) => `Column ${i}`);
    
    // Header-based detection - common names for lat/lon columns
    const latKeywords = ['lat', 'latitude', 'latitud'];
    const lonKeywords = ['lon', 'lng', 'long', 'longitude', 'longitud'];
    
    let latCandidates = [];
    let lonCandidates = [];
    
    // First check headers for common lat/lon names
    headers.forEach((header, index) => {
        const headerLower = header.toLowerCase();
        
        // Check for latitude keywords
        if (latKeywords.some(keyword => headerLower.includes(keyword))) {
            latCandidates.push({ index, confidence: 0.9 }); // High confidence based on header
        }
        
        // Check for longitude keywords
        if (lonKeywords.some(keyword => headerLower.includes(keyword))) {
            lonCandidates.push({ index, confidence: 0.9 }); // High confidence based on header
        }
    });
    
    // If we couldn't find by headers, analyze data in columns
    if (latCandidates.length === 0 || lonCandidates.length === 0) {
        // Get sample rows for analysis (skip header if it exists)
        const startRow = hasHeaders ? 1 : 0;
        const sampleRows = lines.slice(startRow, Math.min(startRow + 10, lines.length))
            .filter(line => line.trim() !== '');
        
        // Extract columns from sample
        const columns = [];
        for (let i = 0; i < headers.length; i++) {
            columns[i] = [];
        }
        
        // Parse column values
        sampleRows.forEach(line => {
            const parts = line.split(',').map(p => p.trim());
            for (let i = 0; i < Math.min(parts.length, headers.length); i++) {
                const parsedValue = parseFloat(parts[i]);
                if (!isNaN(parsedValue)) {
                    columns[i].push(parsedValue);
                }
            }
        });
        
        // Analyze each column for lat/lon characteristics
        columns.forEach((values, index) => {
            if (values.length === 0) return; // Skip empty columns
            
            // Calculate value range and average
            const min = Math.min(...values);
            const max = Math.max(...values);
            const avg = values.reduce((sum, val) => sum + val, 0) / values.length;
            const allNumeric = values.length === sampleRows.length;
            
            // Latitude should be between -90 and 90
            if (allNumeric && min >= -90 && max <= 90) {
                // Higher confidence if values are in typical ranges
                let confidence = 0.6;
                
                // Most populated areas have latitudes between -60 and 75
                if (min >= -60 && max <= 75) {
                    confidence += 0.1;
                }
                
                // Non-zero latitudes increase confidence
                if (Math.abs(avg) > 1) {
                    confidence += 0.1;
                }
                
                latCandidates.push({ index, confidence });
            }
            
            // Longitude should be between -180 and 180
            if (allNumeric && min >= -180 && max <= 180) {
                // Higher confidence if values are in typical ranges
                let confidence = 0.6;
                
                // Most longitudes will have some significant value
                if (Math.abs(avg) > 5) {
                    confidence += 0.1;
                }
                
                lonCandidates.push({ index, confidence });
            }
        });
    }
    
    // If we still have no candidates, look for any numeric columns
    if (latCandidates.length === 0 || lonCandidates.length === 0) {
        // Get the first few numeric columns
        const numericColumns = [];
        
        // Check first few rows
        const startRow = hasHeaders ? 1 : 0;
        const checkRows = Math.min(lines.length, startRow + 3);
        
        for (let i = startRow; i < checkRows; i++) {
            const parts = lines[i].split(',').map(p => p.trim());
            
            parts.forEach((part, index) => {
                if (!numericColumns.includes(index) && !isNaN(parseFloat(part))) {
                    numericColumns.push(index);
                }
            });
        }
        
        // If we have at least two numeric columns, use the first two as lat/lon
        if (numericColumns.length >= 2) {
            if (latCandidates.length === 0) {
                latCandidates.push({ index: numericColumns[0], confidence: 0.3 });
            }
            if (lonCandidates.length === 0) {
                lonCandidates.push({ index: numericColumns[1], confidence: 0.3 });
            }
        }
    }
    
    // Sort candidates by confidence
    latCandidates.sort((a, b) => b.confidence - a.confidence);
    lonCandidates.sort((a, b) => b.confidence - a.confidence);
    
    // Ensure lat and lon are different columns
    if (latCandidates.length > 0 && lonCandidates.length > 0) {
        if (latCandidates[0].index === lonCandidates[0].index) {
            // If the same column is detected for both, take the second best for lon
            if (lonCandidates.length > 1) {
                lonCandidates[0] = lonCandidates[1];
            } else if (latCandidates.length > 1) {
                // Or take the second best for lat
                latCandidates[0] = latCandidates[1];
            } else {
                // If no alternatives, use consecutive columns
                lonCandidates[0] = { index: latCandidates[0].index + 1, confidence: 0.1 };
            }
        }
    }
    
    // Set default values if no candidates were found
    const latIndex = latCandidates.length > 0 ? latCandidates[0].index : 0;
    const lonIndex = lonCandidates.length > 0 ? lonCandidates[0].index : 1;
    
    return { 
        latIndex, 
        lonIndex,
        latConfidence: latCandidates.length > 0 ? latCandidates[0].confidence : 0,
        lonConfidence: lonCandidates.length > 0 ? lonCandidates[0].confidence : 0
    };
}

// Update the CSV parsing function to use autodetected columns
function parseCSVCoordinates(csvContent, latIndex = null, lonIndex = null, descIndex = null, explicitHasHeader = null){
    // Auto-detect columns if not specified
    let columnIndices;
    if (latIndex === null || lonIndex === null) {
        columnIndices = autoDetectCoordinateColumns(csvContent);
        latIndex = columnIndices.latIndex;
        lonIndex = columnIndices.lonIndex;
    }
    
    const lines = csvContent.split(/\r?\n/).filter(line => line.trim());
    const coordinates = [];
    
    // Skip empty file
    if (lines.length === 0) {
        return coordinates;
    }
    
    // Use the improved header detection function
    const hasHeader = typeof explicitHasHeader === 'boolean' ? explicitHasHeader : detectCSVHeaders(csvContent);
    const startRow = hasHeader ? 1 : 0;
    
    // Get headers only if we actually have headers
    const headers = hasHeader ? lines[0].split(',').map(h => h.trim()) : null;
    
    // If no specific description column is provided, try to find a good candidate
    // Typically the first non-lat/lon column
    if (descIndex === null) {
        // Start from column 2 (typically the third column, index 2) if lat/lon are 0 and 1
        let startIdx = (latIndex === 0 && lonIndex === 1) ? 2 : 0;
        
        for (let i = startIdx; i < (lines[0]?.split(',').length || 0); i++) {
            if (i !== latIndex && i !== lonIndex) {
                descIndex = i;
                break;
            }
        }
    }
    
    for (let i = startRow; i < lines.length; i++) {
        const line = lines[i].trim();
        if (!line) continue;
        
        // Split the line into parts
        const parts = line.split(',').map(p => p.trim());
        
        try {
            // Need at least enough parts to include lat and lon
            const maxIndex = Math.max(latIndex, lonIndex);
            if (parts.length <= maxIndex) continue;
            
            // Extract lat and lon from specified indices
            const lat = parseFloat(parts[latIndex]);
            const lon = parseFloat(parts[lonIndex]);
            
            if (isNaN(lat) || isNaN(lon)) continue;
            
            // Normalize coordinates
            const [normalizedLat, normalizedLon] = normalizeCoordinates(lat, lon);
            
            // Build a name/description from the selected description column
            let name = `Point ${i+1}`;
            
            // If we have a valid description column and it exists in this row
            if (descIndex !== null && descIndex < parts.length) {
                const descValue = parts[descIndex];
                if (descValue && descValue.trim()) {
                    // If we have headers, include the header name
                    if (headers && hasHeader) {
                        name = `${headers[descIndex]}: ${descValue}`;
                    } else {
                        // For files without headers, just use the raw description value
                        name = descValue;
                    }
                }
            }
            
            coordinates.push({
                name: name,
                lat: normalizedLat,
                lon: normalizedLon,
                // Store all original parts in case needed for display
                originalData: parts
            });
        } catch (e) {
            console.warn(`Error parsing line ${i+1}: ${line}`);
        }
    }
    
    return coordinates;
}

// Update the processBatchFiles function to include description column selection
function parseCSVFileWithPapa(file, hasHeaders) {
    return new Promise((resolve, reject) => {
        Papa.parse(file, {
            header: hasHeaders,
            skipEmptyLines: true,
            complete: results => resolve(results.data),
            error: err => reject(err)
        });
    });
}

function extractCoordinatesFromRows(rows, latIndex, lonIndex, descIndex, hasHeaders, autoDetect) {
    if (autoDetect && rows.length > 0) {
        const sampleContent = Object.values(rows[0]).join(",");
        const auto = autoDetectCoordinateColumns(sampleContent);
        latIndex = auto.latIndex;
        lonIndex = auto.lonIndex;
    }

    return rows.map((row, i) => {
        const parts = hasHeaders ? Object.values(row) : row;
        const lat = parseFloat(parts[latIndex]);
        const lon = parseFloat(parts[lonIndex]);
        const desc = descIndex !== null && descIndex < parts.length ? parts[descIndex] : `Point ${i + 1}`;
        if (isNaN(lat) || isNaN(lon)) return null;
        const [normLat, normLon] = normalizeCoordinates(lat, lon);
        return {
            name: hasHeaders && descIndex !== null ? `${Object.keys(row)[descIndex]}: ${desc}` : desc,
            lat: normLat,
            lon: normLon,
            originalData: parts
        };
    }).filter(p => p !== null);
}

function processBatchFiles() {
    batchResults = [];
    batchMarkers.forEach(marker => {
        if (batchMap) batchMap.removeLayer(marker);
    });
    batchMarkers = [];
    document.getElementById('batch_result').innerHTML = '';

    const file1 = document.getElementById('file1').files[0];
    const file2 = document.getElementById('file2').files[0];
    const targetMidpointStr = document.getElementById('target_midpoint').value.trim();
    const useAutoDetect = document.getElementById('auto_detect').checked;
    const hasHeaders = document.getElementById('has_headers').checked;

    if (!file1 || !file2 || !targetMidpointStr) {
        document.getElementById('batch_result').innerHTML = '<strong>Please provide both files and a target midpoint.</strong>';
        return;
    }

    let [targetLat, targetLon] = parseCoordinates(targetMidpointStr);
    [targetLat, targetLon] = normalizeCoordinates(targetLat, targetLon);

    let latIndex = null, lonIndex = null, descIndex = null;
    if (!useAutoDetect) {
        latIndex = parseInt(document.getElementById('lat_column').value);
        lonIndex = parseInt(document.getElementById('lon_column').value);
        descIndex = parseInt(document.getElementById('desc_column').value);
    }

    document.getElementById('batch_progress').style.display = 'block';
    document.getElementById('progress_bar').style.width = '0%';
    document.getElementById('progress_text').innerText = 'Reading files...';

    Promise.all([
        parseCSVFileWithPapa(file1, hasHeaders),
        parseCSVFileWithPapa(file2, hasHeaders)
    ]).then(([rows1, rows2]) => {
        const coords1 = extractCoordinatesFromRows(rows1, latIndex, lonIndex, descIndex, hasHeaders, useAutoDetect);
        const coords2 = extractCoordinatesFromRows(rows2, latIndex, lonIndex, descIndex, hasHeaders, useAutoDetect);

        document.getElementById('progress_text').innerText = `Processing ${coords1.length} × ${coords2.length} pairs...`;
        processPairsInBatches(coords1, coords2, targetLat, targetLon, 0, 0, 100);
    }).catch(error => {
        document.getElementById('batch_progress').style.display = 'none';
        document.getElementById('batch_result').innerHTML = '<strong>Error:</strong> ' + error.message;
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
    
    // Take top 10 results
    const topResults = batchResults.slice(0, 10);
    
    // Initialize map
    document.getElementById('batch_map').style.display = 'block';
    initBatchMap();
    
    // Create result HTML
    let resultHTML = `
       <h3>Top 10 Closest Matches:</h3>
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
