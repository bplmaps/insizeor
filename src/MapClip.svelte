<script>
import {
    createEventDispatcher
} from 'svelte';

import Draw from 'ol/interaction/Draw';
import Map from 'ol/Map';
import View from 'ol/View';
import {
    XYZ,
    OSM,
    Vector as VectorSource
} from 'ol/source';
import {
    Tile as TileLayer,
    Vector as VectorLayer
} from 'ol/layer';
import {
    getVectorContext
} from 'ol/render';
import {
    Fill,
    Style
} from 'ol/style';

import {
    toLonLat,
    fromLonLat
} from 'ol/proj';
import {defaults} from 'ol/interaction';

import {
    onMount
} from 'svelte';

import autocomplete from "autocompleter";
import 'autocompleter/autocomplete.css'

import distance from "@turf/distance";

import 'ol/ol.css';

const dispatch = createEventDispatcher();

function complete(imgSrc, size) {
    dispatch('mapClipped', {
        imgSrc: imgSrc,
        size: size
    });
}

let mapId = 'mapDiv';
let map;

let drawingButtonText = "✏️ Click to start drawing";

var raster = new TileLayer({
    source: new XYZ({
        url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
        crossOrigin: "anonymous"
    })
});

var source = new VectorSource({
    wrapX: false
});

var dStyle = new Style({
    fill: null
})

var vector = new VectorLayer({
    source: source,
    style: dStyle
});

var style = new Style({
    fill: new Fill({
        color: 'white',
    }),
});

let drawn = false;
let drawing = false;

var draw; // global so we can remove it later
function addInteraction() {

    if (drawing) {
        drawing = false;
        drawingButtonText = "✏️ Click to start drawing";
        map.removeInteraction(draw);
        source.clear();
    } else {
        drawingButtonText = "🤚 Undo and start over";
        drawing = true;
        map.addInteraction(draw);
    }
}

onMount(() => {

    map = new Map({
        layers: [raster, vector],
        target: mapId,
        view: new View({
            center: [-7913132.21, 5213548.94],
            zoom: 15,
            maxZoom: 19
        }),
        interactions : defaults({doubleClickZoom :false}),
    });

    draw = new Draw({
        source: source,
        type: "Polygon",
    });

    raster.on('postrender', function(e) {
        var vectorContext = getVectorContext(e);
        e.context.globalCompositeOperation = 'destination-in';
        vector.getSource().forEachFeature(function(feature) {
            vectorContext.drawFeature(feature, style);
        });
        e.context.globalCompositeOperation = 'source-over';

    });

    source.on("addfeature", function() {
        map.getView().fit(source.getExtent())
        map.removeInteraction(draw);
        map.on('rendercomplete', () => {
            if (!drawn) {
                let cv = document.querySelector("canvas");
                let ig = cv.toDataURL("image/png");

                drawn = true;
                let ext = map.getView().calculateExtent();

                let y = (ext[1] + ext[3]) / 2;
                let left = toLonLat([ext[0], y]);

                let right = toLonLat([ext[2], y]);

                let w = distance(left, right) * 1000;
                complete(ig, w);

            }

        })

    });

    autocomplete({
        input: document.getElementById('nominatim-autocomplete'),
        "fetch": function(text, callback) {
            fetch(`https://nominatim.openstreetmap.org/?&q=${text}&format=json&limit=5`)
                .then(response => response.json())
                .then(j => {
                    callback(j.map(x => {
                        return {
                            label: x.display_name,
                            latlng: [+x.lat, +x.lon]
                        }
                    }));
                })
        },
        debounceWaitMs: 300,
        onSelect: function(item) {
            let ll = item.latlng.reverse();
            map.getView().setCenter(fromLonLat(ll));
        }
    });
});
</script>

<style>
#map-clip {
    width: 100%;
    background-color: rgba(246, 253, 214, 0.5);
    border: 2px solid rgb(213, 218, 190);
    padding: 10px;
}

.ol-map-clip-outer-holder {
    height: 500px;
    width: 100%;
    background-color: #333;
    position: relative;
}

.ol-map-clip-inner-holder {
    position: absolute;
    height: 500px;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
}

#drawing-button {
    position: absolute;
    z-index: 9999;
    top: 5px;
    right: 5px;
}

#drawing-instructions {
    position: absolute;
    bottom: 3px;
    z-index: 9999;
    right: 5px;
}
</style>

<div id="map-clip">

    <div class="pb-2">
        <input type="text" class="input mb-2" id="nominatim-autocomplete" placeholder="Jump to location ...">
    </div>

    <div class="ol-map-clip-outer-holder" id={mapId}>
        <div id={mapId} class="ol-map-clip-inner-holder"></div>
        <button class="button is-danger" id="drawing-button" on:click="{addInteraction}">{drawingButtonText}</button>
        {#if drawing}<div class="tag is-warning is-small" id="drawing-instructions">Click on corners. Double click to finish.</div>{/if}
    </div>

</div>
