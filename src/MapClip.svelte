<script>

import { createEventDispatcher } from 'svelte';

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

import 'ol/ol.css';


const dispatch = createEventDispatcher();

function complete(imgSrc, size){ dispatch('mapClipped', {imgSrc: imgSrc, size: size}); }

let mapId = 'mapDiv';
let map;

let drawingButtonText = "Start Drawing";

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

var draw; // global so we can remove it later
function addInteraction() {
    draw = new Draw({
        source: source,
        type: "Polygon",
    });
    source.on("addfeature", function() {
        // console.log(source.getFeatures())

        map.getView().fit(source.getExtent())
        map.on('rendercomplete', () => {
            if (!drawn) {
                let cv = document.querySelector("canvas");
                let ig = cv.toDataURL("image/png");
                // let em = document.createElement("img")
                // document.body.appendChild(em);
                // em.src = ig;
                drawn = true;
                var ext = map.getView().calculateExtent();
                complete(ig, ext[3]-ext[1])
            }

        })

    });
    drawingButtonText = "Click on corners, double click to close";
    map.addInteraction(draw);

}

function initializeMap(node, _id) {
    let w = setInterval(() => {
        if (node.clientHeight > 0) {
            clearTimeout(w);
            map = new Map({
                layers: [raster, vector],
                target: mapId,
                view: new View({
                    center: [-7913132.21, 5213548.94],
                    zoom: 15,
                }),
            });

            raster.on('postrender', function(e) {
                var vectorContext = getVectorContext(e);
                e.context.globalCompositeOperation = 'destination-in';
                vector.getSource().forEachFeature(function(feature) {
                    vectorContext.drawFeature(feature, style);
                });
                e.context.globalCompositeOperation = 'source-over';

            });

        } else {
            console.log("waiting for sized box");
        }
    }, 100);
}
</script>

<style>
#map-clip {
    width: 100%;
}

.ol-map-clip-holder {
    height: 500px;
    width: 100%;
    background-color: #333;
}
</style>

<div id="map-clip">

    <div class="ol-map-clip-holder" id={mapId} use:initializeMap={mapId}>
    </div>

    <button class="button is-outlined mt-3" on:click="{addInteraction}" disabled="{drawingButtonText === 'Click on corners, double click to close'}">{drawingButtonText}</button>

</div>
