<html lang="en">
<head>
  <meta charset="utf-8">

  <title>Simple Map</title>
  <meta name="description" content="Testing out basic Leaflet">

  <!-- YOUR STYLES HERE -->
  <!-- Step 1. Paste Leaflet CSS here -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css"
  integrity="sha512-xodZBNTC5n17Xt2atTPuE1HxjVMSvLVW9ocqUKLsCC5CXdbqCmblAshOMAS6/keqq/sMZMZ19scR4PsZChSR7A=="
  crossorigin=""/>

  <!-- Step 2. Paste Leaflet JS here -->
  <script src="https://unpkg.com/leaflet@1.7.1/dist/leaflet.js"
   integrity="sha512-XQoYMqMTK8LvdxXYG3nZ448hOEQiglfqkJs1NOQV44cWnUrBc8PkAOcXy20w0vlaXaVUearIOBhiXZ5V3ynxwA=="
   crossorigin=""></script>

   <style>
    .container {
        margin-left: 15px;
        margin-right: 15px;
        text-align: center;
    }

    .centerobject {
        display: block;
        margin-left: auto;
        margin-right: auto;
    }

    /* Step 3. Define the map window by ID. */

    #mapid { 
        height: 400px;
        width: 700px;
    }

    /* Custom Information Control Styles */
    .info {
        padding: 6px 8px;
        font: 14px/16px Arial, Helvetica, sans-serif;
        background: white;
        background: rgba(255,255,255,0.8);
        box-shadow: 0 0 15px rgba(0,0,0,0.2);
        border-radius: 5px;
    }
    .info h4 {
        margin: 0 0 5px;
        color: #777;
    }

    /* Custom Legend Control Styles */
    .legend {
        line-height: 18px;
        color: #555;
    }

    .legend i {
        width: 18px;
        height: 18px;
        float: left;
        margin-right: 8px;
        opacity: 0.7;
    }
   </style>
</head>

<body>
    <div class="container">
        <h1>Map Sample - U.S.A. Population Density - Choropleth</h1>
    </div>

    <!-- Step 4. Create the div to contain the map. -->
    <!-- Name of the ID doesn't matter, so long as it's kept consistent. -->
    <div id="mapid" class="centerobject">
    </div>

  <!-- YOUR JAVASCRIPT HERE -->
  <!-- Step 5. US-state.js is a GeoJSON file defining data for each state, along with the graphical shape for each state that will be its outline.-->
  <script type="text/javascript" src="us-states.js"></script>

  <script type="text/javascript">
  // Step 6. Fetch the Leaflet map from the "L" library, set its view using [coordinates] and initial zoom (4 in this case). Assign the map to the div with the ID 'mapid'.
    let map = L.map('mapid').setView([37.8, -96], 4);

    // This function determines the colors of the specific data property values that get fed into it.
    function getColor(d) {
        return d > 1000 ? '#91003f' :
            d > 500 ? '#ce1256' :
            d > 200 ? '#e7298a' :
            d > 100 ? '#df65b0' :
            d > 50  ? '#c994c7' :
            d > 20  ? '#d4b9da' :
            d > 10  ? '#facd64' :
        '#f7f699';
    }   

    // This variable will define GeoJSON for use in the event listeners
    let geojson;

    // This function determines the style of the GeoJSON overlay
    function style(feature) {
        return {
            fillColor: getColor(feature.properties.density),
            weight: 1,
            opacity: 1,
            color: 'white',
            dashArray: '3',
            fillOpacity: 0.8
        };
    }

    // This function is for the MOUSEOVER event
    function highlightFeature(e) {
        let layer = e.target;

        layer.setStyle({
            fillColor: '#64f754',
            weight: 5,
            color: '#154a10',
            dashArray: '',
            fillOpacity: 0.7
        });

        if (!L.Browser.ie && !L.Browser.opera && L.Browser.edge) {
            layer.bringToFront();
        }

        info.update(layer.feature.properties); // This passes the props for the Custom Information Control function
    }

    // This function is for the MOUSEOUT event
    function resetHighlight(e) {
        geojson.resetStyle(e.target);

        info.update(); // This resers the Custom Information Control to the default informationless state
    }

    // This function is for the CLICK event
    function zoomToFeature(e) {
        map.firBounds(e.target.getBounds());
    }

    // This function will add the event listeners to the U.S. state layers
    function onEachFeature(feature, layer) {
        layer.on({
            mouseover: highlightFeature,
            mouseout: resetHighlight,
            click: zoomToFeature
        });
    }

    // Custom Information Control
    // See above for the styles of this segment
    let info = L.control();

    info.onAdd = function (map) {
        this._div = L.DomUtil.create('div', 'info'); // This creates a div with a class "info"
        this.update();
        return this._div;
    };

    // This method will be used to update the control based on feature properties passed
    info.update = function (props) {
        this._div.innerHTML = '<h4>US Population Density</h4>' + (props ?
        '<b>' + props.name + '</b><br />' + props.density + ' people / mi<sup>2</sup>' : 'Hover over a state');
    };

    info.addTo(map);

    // Custom Legend Control
    // See above for the styles of this segment
    // Creating a static legend for the map
    let legend = L.control({position: 'bottomright'});

    legend.onAdd = function (map) {

        let div = L.DomUtil.create('div', 'info legend'),
        grades = [0, 10, 20, 50, 100, 200, 500, 1000],
        labels = [];

        // Looping through the density intervals and generating a label with a colored square for each interval
        for (let i = 0; i <grades.length; i++) {
            div.innerHTML += '<i style="background:' + getColor(grades[i] + 1) + '"></i> ' + grades[i] + (grades[i + 1] ? '&ndash;' + grades[i + 1] + '<br>' : '+');
        }

        return div;
    };

    legend.addTo(map);

    // Reused the map from the simple-map example.
    // Note that the addTo function adds the tiling to the "map" variable above.
    // Do not confuse the "map" variable with the "mapid" class indicated in the <div>.
    L.tileLayer('https://tile.thunderforest.com/neighbourhood/{z}/{x}/{y}.png?apikey=e6af7839f94a43428e202e1290e0a2be', {
        attribution: 'Map data &copy; <a href="https://manage.thunderforest.com">Thunderforest</a> contributors, Imagery ?? <a href="https://manage.thunderforest.com/">Thunderforest</a>',
        maxZoom: 18,
        tileSize: 512,
        zoomOffset: -1,
        accessToken: 'e6af7839f94a43428e202e1290e0a2be'
    }).addTo(map);

    // GeoJSON is also added to the "map" variable.
	geojson = L.geoJson(statesData, {
        style: style,
        onEachFeature: onEachFeature
    }).addTo(map); // The colour style determined with functions getColor and style is added here. So are the event listeners.
  </script>

</body>
</html>
