# Interactive Map QUALTRICS 👨🏻‍💻
Hello! I am Vicente Baier, a student of Industrial and Transport Engineering at the Pontifical Catholic University of Chile. During a research with the Professor Camila Balbontín, I developed an interactive map that allows users to select a location within Santiago. This map includes an integrated search bar and is built using the Leaflet library.

Below, I am sharing the code so you can incorporate this functionality into your surveys at the Qualtrics platform.

To ensure the code works correctly, you need to create an embedded data field to store the selected coordinates. The map is configured with the location of Santiago, Chile, but it can be adapted for other cities. Everything is programmed in JavaScript.

    // Open Source Code created by Vicente Baier
    
    // Remember to create an embedded data field __js_coordinates (to store coordinates)
    
    Qualtrics.SurveyEngine.addOnload(function() {
        // Load the Leaflet library
        var scriptLeaflet = document.createElement('script');
        scriptLeaflet.src = 'https://unpkg.com/leaflet@1.7.1/dist/leaflet.js';
        document.head.appendChild(scriptLeaflet);
    
        var cssLeaflet = document.createElement('link');
        cssLeaflet.rel = 'stylesheet';
        cssLeaflet.href = 'https://unpkg.com/leaflet@1.7.1/dist/leaflet.css';
        document.head.appendChild(cssLeaflet);
    
        // Add the Leaflet geocoding library
        var scriptGeocoder = document.createElement('script');
        scriptGeocoder.src = 'https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js';
        document.head.appendChild(scriptGeocoder);
    
        var cssGeocoder = document.createElement('link');
        cssGeocoder.rel = 'stylesheet';
        cssGeocoder.href = 'https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css';
        document.head.appendChild(cssGeocoder);
    
        // Wait for the Leaflet library to load
        scriptLeaflet.onload = function() {
            var map = L.map('map').setView([-33.4489, -70.6693], 13); // Santiago, Chile
    
            L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
                attribution: '© OpenStreetMap contributors',
                maxZoom: 18
            }).addTo(map);
    
            var marker;
    
            // Function to save and log coordinates
            function saveAndLogCoordinates(lat, lng) {
                Qualtrics.SurveyEngine.setJSEmbeddedData('coordinates', lat + ',' + lng);
    			Qualtrics.SurveyEngine.setJSEmbeddedData('lat_origen', lat);
    			Qualtrics.SurveyEngine.setJSEmbeddedData('long_origen', lng);
                console.log("Coordinates saved: " + lat + ", " + lng);
                console.log("Embedded Data (Qualtrics): " + Qualtrics.SurveyEngine.getJSEmbeddedData('coordinates'));
            }
    
            // Function for map click
            map.on('click', function(e) {
                var coords = e.latlng; // Get lat, lng coordinates
                var lat = coords.lat;
                var lng = coords.lng;
    
                // Remove existing marker before creating a new one
                if (marker) {
                    map.removeLayer(marker);
                }
    
                // Create a new marker at the selected location
                marker = L.marker([lat, lng]).addTo(map);
                marker.bindPopup('Selected coordinates: ' + lat + ', ' + lng).openPopup();
    
                // Call the function to save and log coordinates
                saveAndLogCoordinates(lat, lng);
            });
    
            // Wait for the geocoding library to load
            scriptGeocoder.onload = function() {
                var geocoder = L.Control.geocoder({
                    defaultMarkGeocode: false
                }).on('markgeocode', function(e) {
                    var lat = e.geocode.center.lat;
                    var lng = e.geocode.center.lng;
    
                    // Remove existing marker before creating a new one
                    if (marker) {
                        map.removeLayer(marker);
                    }
    
                    // Create a new marker at the searched location
                    marker = L.marker([lat, lng]).addTo(map);
                    marker.bindPopup('Selected coordinates: ' + lat + ', ' + lng).openPopup();
                    map.setView([lat, lng], 13);
    
                    // Call the function to save and log coordinates
                    saveAndLogCoordinates(lat, lng);
                }).addTo(map);
            };
        };
    
        // Error handling for library loading
        scriptLeaflet.onerror = function() {
            console.error("Error loading the Leaflet library");
        };
    
        scriptGeocoder.onerror = function() {
            console.error("Error loading the geocoding library");
        };
    });
    
    Qualtrics.SurveyEngine.addOnUnload(function() {
        setTimeout(function() {
            // Retrieve the embedded data __js_coordinates
            var coords = Qualtrics.SurveyEngine.getJSEmbeddedData('coordinates');
            if (coords) {
                console.log("Coordinates on page unload: " + coords);
                console.log("Embedded Data (Qualtrics) on Unload: " + coords);
            } else {
                console.warn("No coordinates saved on page unload.");
                console.log("Current value of coordinates: ", Qualtrics.SurveyEngine.getJSEmbeddedData('coordinates'));
            }
        }, 100); // Wait 100 ms to ensure the response is saved
    });
    
    // Open Source Code created by Vicente Baier

To ensure proper display on the platform, integrate the following code in HTML. I recommend using this configuration, which is the one I used.

    // Non-important text
    &iquest;En qu&eacute; parte de Santiago vive?<br />
    <span style="font-size:13px;"><em><strong>No es necesario que ponga su direcci&oacute;n exacta</strong>, nos basta con una referencia, como la intersecci&oacute;n de calles m&aacute;s cercana, para tener una idea aproximada de su localizaci&oacute;n.</em></span>
    // Important
    <div id="map" style="height: 400px; width: 100%;">&nbsp;</div>
    <input id="coordinates" name="coordinates" type="hidden" />
