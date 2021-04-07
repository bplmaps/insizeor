<script>

export let mapVars = {
    "overlayImage": null,
    "imageWidth": 0
}

import {onMount} from "svelte";
import L from "Leaflet";
import autocomplete from "autocompleter";
import 'autocompleter/autocomplete.css'

import { createEventDispatcher } from 'svelte';

const dispatch = createEventDispatcher();

function startOver() {
    dispatch('startOver');
}

let rotation = 0;
let scale = 1.0;

function warpImage() {
    document.getElementById('image-overlay').style.transform = `translate(-50%,-50%) scale(${scale}) rotate(${rotation}deg)`;
}

onMount(()=>{

    autocomplete({
        input: document.getElementById('nominatim-autocomplete'),
        "fetch": function(text,callback) {
            fetch(`https://nominatim.openstreetmap.org/?&q=${text}&format=json&limit=5`)
                .then(response => response.json())
                .then(j => { callback(j.map(x => {return { label: x.display_name, latlng: [+x.lat, +x.lon] }})); })
        },
        debounceWaitMs: 300,
        onSelect: function(item) {
            console.log(map.setView(item.latlng, 17));
        }
        });

    let map = L.map("map-inner-container", {
       center: [42.356613, -71.084876],
        zoom: 17
    });

    function warpScale() {
        let metersPerPx = (156543.03392 * Math.cos((map.getCenter().lat * Math.PI) / 180)) / Math.pow(2, map.getZoom());
        scale = (1 / metersPerPx) * (mapVars.imageWidth/400);
        warpImage();
    }

    L.tileLayer(
    "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
    {
        attribution:
        "Source: Esri, Maxar, GeoEye, Earthstar Geographics, CNES/Airbus DS, USDA, USGS, AeroGRID, IGN, and the GIS User Community"
    }
    ).addTo(map);

    warpScale();

    map.on("zoomend", warpScale);

})


</script>

<style>
    #map-view {
        position: absolute;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        display: flex;
        flex-direction: column;
    }

    #map-view-top {
        padding: 20px;
    }

    #map-view-bottom
    {
        flex-grow: 1;
        display: flex;
        align-items: stretch;
        padding: 10px;
    }

    #map-outer-container {
        position: relative;
        width: 100%;
    }

    #map-inner-container {
        height: 100%;
        width: 100%;
        z-index: 1;
    }

    #image-overlay {
        position: absolute;
        top: 50%;
        left: 50%;
        height: auto;
        width: 400px;
        z-index: 999;
        transform: translate(-50%, -50%);
        -webkit-filter: drop-shadow(5px 5px 5px #222);
        filter: drop-shadow(5px 5px 5px #222);
        pointer-events: none;
    }


    .autocomplete > div {
        z-index: 999;
        padding: 19px;
    }

</style>


<div id="map-view">
    <div id="map-view-top">
        <div class="field is-grouped is-grouped-centered">
        <p class="control">
            <button class="button is-primary is-outlined" on:click="{startOver}">Start Over</button>
        </p>
        <p class="control">
            <button class="button is-primary" on:click="{()=>{rotation = rotation + 10; warpImage(); }}">⤵️ Rotate right</button>
        </p>
        <p class="control">
            <button class="button is-primary" on:click="{()=>{rotation = rotation - 10; warpImage(); }}">⤴️ Rotate left</button>
        </p>
        <p class="control">
            <input type="text" class="input" id="nominatim-autocomplete" placeholder="Jump to location ...">
        </p>
        </div>

    </div>
    <div id="map-view-bottom">
        <div id="map-outer-container">
            <div id="map-inner-container"></div>
            <div id="image-overlay">
                <img src="{mapVars.overlayImage}" alt="">
            </div>
        </div>
    </div>


</div>