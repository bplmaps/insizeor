<script>
export let mapVars = {
    "overlayImage": null,
    "imageWidth": 0
}

import {
    onMount
} from "svelte";

import autocomplete from "autocompleter";
import 'autocompleter/autocomplete.css'

import Map from 'ol/Map';
import View from 'ol/View';
import {
    XYZ,
} from 'ol/source';
import {
    Tile as TileLayer,
} from 'ol/layer';

import {
    toLonLat,
    fromLonLat
} from 'ol/proj';

import {
    ScaleLine,
    defaults as defaultControls
} from 'ol/control';

import {
    createEventDispatcher
} from 'svelte';

const dispatch = createEventDispatcher();

function startOver() {
    dispatch('startOver');
}

let rotation = 0;
let scale = 1.0;

function warpImage() {
    document.getElementById('image-overlay').style.transform = `translate(-50%,-50%) scale(${scale}) rotate(${rotation}deg)`;
}

let mapId = "map-inner-container";
var map;

function zoomCalculator(w) {
    console.log(w)
    let cutpoints = [{
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

    const f = cutpoints.find(x => x.width > w+200);
    return f.zoom;
}

onMount(() => {

  window.scrollTo(0,0);

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

    var raster = new TileLayer({
        source: new XYZ({
            url: "https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}",
            crossOrigin: "anonymous"
        })
    });

    map = new Map({
        layers: [raster],
        target: mapId,
        controls: defaultControls().extend([new ScaleLine({
            units: "metric",
        })]),
        view: new View({
            center: [-7911042.34, 5214298.54],
            zoom: zoomCalculator(mapVars.imageWidth),
            maxZoom: 19
        })
    });

    map.on("moveend", warpScale);

    function warpScale() {
        let v = map.getView();
        let ll = toLonLat(v.getCenter());
        let z = v.getZoom();

        let metersPerPx =
            (156543.03392 * Math.cos((ll[1] * Math.PI) / 180)) /
            Math.pow(2, z);
        scale = 1 / metersPerPx * (mapVars.imageWidth / 400);
        warpImage();
    }

});
</script>

<style>
#map-view {
    height: 900px;
    width: 100%;
    display: flex;
    flex-direction: column;
}

#map-view-top {
    padding: 5px;
}

#map-view-bottom {
    flex-grow: 1;
    display: flex;
    align-items: stretch;
    padding: 5px;
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

.autocomplete>div {
    z-index: 999;
    padding: 19px;
}
</style>

<div id="map-view">
    <div id="map-view-top">
        <div class="has-text-centered mb-2">
                <button class="button mb-1 is-danger is-outlined" on:click="{startOver}">Start Over</button>
                <button class="button mb-1 is-primary" on:click="{()=>{rotation = rotation + 10; warpImage(); }}">⤵️ Rotate right</button>
                <button class="button mb-1 is-primary" on:click="{()=>{rotation = rotation - 10; warpImage(); }}">⤴️ Rotate left</button>
        </div>
        <input type="text" class="input" id="nominatim-autocomplete" placeholder="Jump to location ...">


    </div>
    <div id="map-view-bottom">
        <div id="map-outer-container">
            <div id="{mapId}"></div>
            <div id="image-overlay">
                <img src="{mapVars.overlayImage}" alt="">
            </div>
        </div>
    </div>
    
    {#if mapVars.credit != null }
    <div class="notification is-secondary is-light my-3">
        <strong>This example is thanks to</strong>: {@html mapVars.credit}
    </div>
    {/if}

</div>
