<script>

export let mapVars = {
    "overlayImage": null,
    "imageWidth": 0
}

import {onMount} from "svelte";
import L from "leaflet";
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

function zoomCalculator(w) {
   let cutpoints = [
 {
   "zoom": 20,
   "width": 74.5
 },
 {
   "zoom": 19,
   "width": 149
 },
 {
   "zoom": 18,
   "width": 298
 },
 {
   "zoom": 17,
   "width": 596.5
 },
 {
   "zoom": 16,
   "width": 1193.5
 },
 {
   "zoom": 15,
   "width": 2386.5
 },
 {
   "zoom": 14,
   "width": 4773.5
 },
 {
   "zoom": 13,
   "width": 9546.5
 },
 {
   "zoom": 12,
   "width": 19093.5
 },
 {
   "zoom": 11,
   "width": 38186.5
 },
 {
   "zoom": 10,
   "width": 76373
 },
 {
   "zoom": 9,
   "width": 152746
 },
 {
   "zoom": 8,
   "width": 305492
 },
 {
   "zoom": 7,
   "width": 611000
 },
 {
   "zoom": 6,
   "width": 1222000
 },
 {
   "zoom": 5,
   "width": 2444000
 },
 {
   "zoom": 4,
   "width": 4888000
 },
 {
   "zoom": 3,
   "width": 9775500
 },
 {
   "zoom": 2,
   "width": 19551500
 },
 {
   "zoom": 1,
   "width": 39103000
 },
 {
   "zoom": 0,
   "width": 78206000
 }

]

for (let index = 0; index < cutpoints.length; index++) {
    if(cutpoints[index].width > w) {
        return cutpoints[index].zoom;
    }    
}

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
            console.log(map.setView(item.latlng, zoomCalculator(mapVars.imageWidth)));
        }
        });



    let map = L.map("map-inner-container", {
       center: [42.356613, -71.084876],
        zoom: zoomCalculator(mapVars.imageWidth)
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