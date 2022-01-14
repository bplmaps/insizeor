<script>
  import './bulma-imports.scss';


  import MapView from './MapView.svelte';
  import MapClip from './MapClip.svelte';

  let view = "splash";
  let inputImage;
  let loadedImageSrc;

  let mapVars = {};

  let measurementUnit = "meters";
  let inputWidthDimension;

  let examples = [
    {label: "Evergiven Ship", image: "https://cdn.glitch.com/40d83f73-34cc-401e-90f3-d6b2f92f2b01%2Fevergiven.png?v=1616949352560", width: 400},
    {label: "BPL McKim Building", image: "https://s3.us-east-2.wasabisys.com/lmec-public-files/insizeor-examples/mckim.png", width: 72},
    {label: "Fenway Park", image: "https://s3.us-east-2.wasabisys.com/lmec-public-files/insizeor-examples/fenway.png", width: 235},
    {label: "Rhode Island", image: "https://s3.us-east-2.wasabisys.com/lmec-public-files/insizeor-examples/ri.png", width: 60000}

  ];

  function checkImageUrl() {
    if(!inputImage || inputImage.length < 7 || inputImage.slice(0,8) != "https://") {
      window.alert("The image URL must begin with https:// ");
    }
    else {
      fetch(inputImage)
        .then(response => {
          if(response.status === 200) {
            loadedImageSrc = inputImage;
            view = "loadImageInterstitial";
          } else {
            window.alert("Something seems wrong with that URL. Try entering it again.")
          }
        })
        .catch(()=>{ window.alert("Something seems wrong with that URL. The server may not allow images to be shared to other sites."); });
    }
  }

  function startOver() {
    inputImage = "";
    loadedImageSrc = null;
    view = "splash";
  }

  function finishLoadingImage() {
    if(isNaN(inputWidthDimension)){
      window.alert("The size must be a number");
    }
    else {
      let w = measurementUnit === "feet" ? (inputWidthDimension * 0.3048) : inputWidthDimension;
      enterMap(loadedImageSrc, +w);
    }
  }

  function enterMap(img,width) {
    mapVars.overlayImage = img;
    mapVars.imageWidth = width;
    view = "main";
  }

</script>

<style>
  @import url("https://use.typekit.net/ros1unq.css");
  
  :global(html) {
    height: 100%;
  }

  :global(body) {
    margin: 0;
    font-family: azo-sans-web, Helvetica, sans-serif;
    background: rgb(255,255,240);
    background: linear-gradient(135deg, rgba(255,255,240,1) 0%, rgba(230,251,255,1) 100%);
    min-height: 100%;
  }
  .wraps-all {
    text-align: center;
    padding: 2rem;
  }

  h1.title {
    text-transform: uppercase;
    color: #373fff;
    text-shadow: 2px 2px 0px coral;
    font-size: 2.5rem;
    font-family: azo-sans-uber;
    font-weight: 400;
    letter-spacing: 3px;
  }

  .column-bumper {
    border: 3px inset rgba(157, 100, 167, 0.27);
    background-color: rgba(255,255,255,0.4);
  }

  .column-bumper-a {
    box-shadow: -3px -3px 0 rgba(255, 188, 81, 0.5);
  }

  .column-bumper-b {
    box-shadow: -3px -3px 0 rgba(235, 154, 101, 0.5);
  }

  .preview-image {
    max-width: 100%;
    max-height: 500px;
  }

  .footer-logo {
    width: 200px;
    height: auto;
  }
  
</style>

<svelte:head>
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.7.1/dist/leaflet.css"
    integrity="sha512-xodZBNTC5n17Xt2atTPuE1HxjVMSvLVW9ocqUKLsCC5CXdbqCmblAshOMAS6/keqq/sMZMZ19scR4PsZChSR7A=="
    crossorigin=""/>
</svelte:head>


<div class="wraps-all">


  {#if view === "splash" }
  <section id="splash">
    <div class="container">
    <h1 class="title">Insizeor</h1>
    <div class="is-size-5 mb-4">
    <p>How big is a cargo ship compared to Boston Common? What would the White House look like in Glacier National Park?</p>
    <p class="notification is-success is-light my-2 has-text-weight-bold">Use this tool to drop any image to scale on top of an aerial map. There are three different ways to get started.</p>
  </div>
    

    <div class="has-text-centered py-5">

      <div class="columns is-vcentered column-bumper column-bumper-a">
        <div class="column is-one-third is-size-4">Enter the URL to any image</div>
        <div class="column">
          <div class="field has-addons">
          <div class="control is-expanded">
            <input bind:value={inputImage} class="input" type="text" placeholder="https://.../image.png" aria-label="Field to enter an image URL">
          </div>
          <div class="control">
            <button class="button is-primary has-text-weight-bold" on:click="{checkImageUrl}">
              Load image
            </button>
          </div>
         
        </div>
        <p class="is-size-7">🤷 Need somewhere to upload an image? Try <a href="https://imgur.com">Imgur</a> or <a href="https://postimages.org">Postimage</a>.</p>
      </div>
      </div>

      <div class="columns is-vcentered column-bumper column-bumper-b mt-3">
        <div class="column is-one-third is-size-4">Choose an example from our gallery</div>
        <div class="column">
            {#each examples as example}
              <button class="button is-primary has-text-weight-bold mr-4 mb-1" on:click={()=>{enterMap(example.image, example.width)}}>{example.label}</button>
            {/each}
        </div>
      </div>

      <div class="columns is-vcentered column-bumper column-bumper-b mt-3">
        <div class="column is-one-third is-size-4">Draw a shape on the map to clip out</div>
        <div class="column">
            <MapClip on:mapClipped={(event)=>{ enterMap(event.detail.imgSrc, event.detail.size) }}></MapClip>
        </div>
      </div>

   
    </div>

    </div>
      
  </section>

  {:else if view==="loadImageInterstitial"}

  <section id="imageInterstitial">
    <div class="container">
      <p>OK, here's the image we'll be using. If it doesn't look right to you, you should <button class="button is-primary is-small has-text-weight-bold is-outlined" on:click="{startOver}">Start Over</button></p>
      <div class="preview-image-holder box my-4">
        <img class="preview-image" src={loadedImageSrc} alt="The image that you loaded">
      </div>
      <p>To scale it, we need to know how big it is in the real world <strong>from the left side of the image to the right</strong>.</p>
      <div class="field has-addons has-addons-centered py-2">
        <p class="control">
          <input class="input" type="text" bind:value={inputWidthDimension} placeholder="Real size of this image">
        </p>
        <p class="control">
          <span class="select">
            <select bind:value="{measurementUnit}">
              <option value="meters">meters</option>
              <option value="feet">feet</option>
            </select>
          </span>
        </p>
        <p class="control">
          <button class="button is-primary has-text-weight-bold" on:click="{finishLoadingImage}">
            Enter
          </button>
        </p>
      </div>
    </div>
  </section>
  

  {:else if view==="main" }
  <section id="main">
    <div class="notification is-success is-light">
      <p class="has-text-weight-bold">Here's your image dropped down on top of Boston Common. Grab and move the map, or type in the search bar to move elsewhere. You can use the rotate buttons to adjust the positioning of the scaled image.</p></div>
    <MapView bind:mapVars={mapVars} on:startOver={startOver} />
  </section>
  {/if}

  <footer>
    <p class="mb-2">
      <a class="button is-small is-link is-outlined" href="https://leventhalmap.org/donate">💌 Support free programs at LMEC</a>
    </p>
    <a href="https://leventhalmap.org"><img src="https://s3.us-east-2.wasabisys.com/lmec-public-files/images/MapCenter-small.png" alt="LMEC Logo" class="footer-logo"></a>
    <p class="is-size-7"><a href="https://www.arcgis.com/home/item.html?id=10df2279f9684e4a9f6a7f08febac2a9">Map imagery source</a>: Esri, Maxar, GeoEye, Earthstar Geographics, CNES/Airbus DS, USDA, USGS, AeroGRID, IGN, and the GIS User Community</p>
  </footer>




</div>
