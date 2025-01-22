# Mapa Interactivo QUALTRICS 👨🏻‍💻

¡Hola! Soy Vicente Baier, estudiante de Ingeniería Industrial y Transporte en la Pontificia Universidad Católica de Chile. Durante una investigación con la profesora Camila Balbontín, desarrollé un mapa interactivo que permite seleccionar una ubicación dentro de Santiago. Este mapa incluye un buscador integrado y está construido utilizando la librería Leaflet.

A continuación, les comparto el código para que puedan incorporar esta funcionalidad en sus encuestas en la plataforma Qualtrics.

Para que el código funcione correctamente, deben crear un dato embebido para almacenar las coordenadas seleccionadas. El mapa está configurado con la ubicación de Santiago de Chile, pero es posible adaptarlo a otras ciudades. Todo está programado en JavaScript.

	// Codigo Abierto creado por Vicente Baier 
	
	// Recordar crear dato embebido __js_coordinates (para guardar coordenadas)
	
	Qualtrics.SurveyEngine.addOnload(function() {
	    // Cargamos la librería de Leaflet
	    var scriptLeaflet = document.createElement('script');
	    scriptLeaflet.src = 'https://unpkg.com/leaflet@1.7.1/dist/leaflet.js';
	    document.head.appendChild(scriptLeaflet);
	
	    var cssLeaflet = document.createElement('link');
	    cssLeaflet.rel = 'stylesheet';
	    cssLeaflet.href = 'https://unpkg.com/leaflet@1.7.1/dist/leaflet.css';
	    document.head.appendChild(cssLeaflet);
	
	    // Agregar la librería de geocodificación de Leaflet
	    var scriptGeocoder = document.createElement('script');
	    scriptGeocoder.src = 'https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js';
	    document.head.appendChild(scriptGeocoder);
	
	    var cssGeocoder = document.createElement('link');
	    cssGeocoder.rel = 'stylesheet';
	    cssGeocoder.href = 'https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css';
	    document.head.appendChild(cssGeocoder);
	
	    // Esperamos a que se cargue la librería de Leaflet
	    scriptLeaflet.onload = function() {
	        var map = L.map('map').setView([-33.4489, -70.6693], 13); // Santiago, Chile
	
	        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
	            attribution: '© OpenStreetMap contributors',
	            maxZoom: 18
	        }).addTo(map);
	
	        var marker;
	
	        // Función para guardar y registrar coordenadas
	        function guardarYRegistrarCoordenadas(lat, lng) {
	            Qualtrics.SurveyEngine.setJSEmbeddedData('coordinates', lat + ',' + lng);
				Qualtrics.SurveyEngine.setJSEmbeddedData('lat_origen', lat);
				Qualtrics.SurveyEngine.setJSEmbeddedData('long_origen', lng);
	            console.log("Coordenadas guardadas: " + lat + ", " + lng);
	            console.log("Embedded Data (Qualtrics): " + Qualtrics.SurveyEngine.getJSEmbeddedData('coordinates'));
	        }
	
	        // Funcion para click en el mapa
	        map.on('click', function(e) {
	            var coords = e.latlng; // Obtener coordenadas lat, lng
	            var lat = coords.lat;
	            var lng = coords.lng;
	
	            // Si ya existe un marcador, lo eliminamos antes de crear uno nuevo
	            if (marker) {
	                map.removeLayer(marker);
	            }
	
	            // Crear un nuevo marcador en la ubicación seleccionada
	            marker = L.marker([lat, lng]).addTo(map);
	            marker.bindPopup('Coordenadas seleccionadas: ' + lat + ', ' + lng).openPopup();
	
	            // Llamar a la función para guardar y registrar coordenadas
	            guardarYRegistrarCoordenadas(lat, lng);
	        });
	
	        // Esperar a que cargue la librería de geocodificación
	        scriptGeocoder.onload = function() {
	            var geocoder = L.Control.geocoder({
	                defaultMarkGeocode: false
	            }).on('markgeocode', function(e) {
	                var lat = e.geocode.center.lat;
	                var lng = e.geocode.center.lng;
	
	                // Si ya existe un marcador, lo eliminamos antes de crear uno nuevo
	                if (marker) {
	                    map.removeLayer(marker);
	                }
	
	                // Crear un nuevo marcador en la ubicación buscada
	                marker = L.marker([lat, lng]).addTo(map);
	                marker.bindPopup('Coordenadas seleccionadas: ' + lat + ', ' + lng).openPopup();
	                map.setView([lat, lng], 13);
	
	                // Llamar a la función para guardar y registrar coordenadas
	                guardarYRegistrarCoordenadas(lat, lng);
	            }).addTo(map);
	        };
	    };
	
	    // Manejo de errores al cargar bibliotecas
	    scriptLeaflet.onerror = function() {
	        console.error("Error al cargar la librería de Leaflet");
	    };
	
	    scriptGeocoder.onerror = function() {
	        console.error("Error al cargar la librería de geocodificación");
	    };
	});
	
	Qualtrics.SurveyEngine.addOnUnload(function() {
	    setTimeout(function() {
	        // Obtenemos el dato embebido __js_coordinates 
	        var coords = Qualtrics.SurveyEngine.getJSEmbeddedData('coordinates');
	        if (coords) {
	            console.log("Coordenadas al descargar la página: " + coords);
	            console.log("Embedded Data (Qualtrics) on Unload: " + coords);
	        } else {
	            console.warn("No hay coordenadas guardadas al descargar la página.");
	            console.log("Valor actual de coordinates: ", Qualtrics.SurveyEngine.getJSEmbeddedData('coordinates'));
	        }
	    }, 100); // Esperar 100 ms para asegurarnos que se guarda la respuesta
	});

 	// Codigo Abierto creado por Vicente Baier

Para garantizar una correcta visualización dentro de la plataforma, deben integrar el siguiente código en HTML. Les recomiendo usar esta configuración, que es la que utilicé.

	&iquest;En qu&eacute; parte de Santiago vive?<br />
	<span style="font-size:13px;"><em><strong>No es necesario que ponga su direcci&oacute;n exacta</strong>, nos basta con una referencia, como la intersecci&oacute;n de calles m&aacute;s cercana, para tener una idea aproximada de su localizaci&oacute;n.</em></span>
	<div id="map" style="height: 400px; width: 100%;">&nbsp;</div>
	<input id="coordinates" name="coordinates" type="hidden" />
