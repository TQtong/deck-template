<template>
  <!-- <div id="test"></div> -->
</template>

<script setup lang="ts">
import { Deck, SolidPolygonLayer, TerrainLayer } from 'deck.gl'
import { onMounted } from 'vue'
import { TerrainLayerProps } from '@deck.gl/geo-layers'

const MAPBOX_TOKEN =
  'pk.eyJ1IjoieHNqY29kaW5nIiwiYSI6ImNscmtsdGdhZjAyNHAycXJha2NxaXE4aDIifQ.spJXyacE9bfThdQ4oOqkVQ' // eslint-disable-line

const TERRAIN_IMAGE = `https://api.mapbox.com/v4/mapbox.terrain-rgb/{z}/{x}/{y}.png?access_token=${MAPBOX_TOKEN}`

// https://docs.mapbox.com/help/troubleshooting/access-elevation-data/#mapbox-terrain-rgb
// Note - the elevation rendered by this example is greatly exagerated!
const ELEVATION_DECODER: TerrainLayerProps['elevationDecoder'] = {
  rScaler: 6553.6,
  gScaler: 25.6,
  bScaler: 0.1,
  offset: -10000,
}

/*
 * https://deck.gl/docs/api-reference/layers/solid-polygon-layer
 */

const layer = new SolidPolygonLayer({
  id: 'SolidPolygonLayer',
  data: 'https://raw.githubusercontent.com/visgl/deck.gl-data/master/website/sf-zipcodes.json',

  /* props from SolidPolygonLayer class */

  // elevationScale: 1,
  extruded: true,
  filled: true,
  getElevation: () => 1000,
  getFillColor: (d) => [d.population / d.area / 60, 140, 0],
  getLineColor: [80, 80, 80],
  getPolygon: (d) => d.contour,
  material: true,
  wireframe: true,

  /* props inherited from Layer class */

  autoHighlight: false,
  coordinateOrigin: [0, 0, 0],
  // coordinateSystem: COORDINATE_SYSTEM.LNGLAT,
  highlightColor: [0, 0, 128, 128],
  modelMatrix: null,
  opacity: 1,
  pickable: true,
  // visible: true,
  // wrapLongitude: false,
})

onMounted(() => {
  new Deck({
    initialViewState: {
      longitude: -122.4,
      latitude: 37.74,
      zoom: 11,
      maxZoom: 20,
      pitch: 30,
      bearing: 0,
    },
    controller: { maxPitch: 90 },
    getTooltip: ({ object }) =>
      object &&
      `${object.zipcode}
Population: ${object.population}`,
    layers: [
      layer,
      // new TerrainLayer({
      //   id: 'terrain-source',
      //   // Data source: https://www.mapzen.com/blog/terrain-tile-service/
      //   elevationData:
      //     'https://s3.amazonaws.com/elevation-tiles-prod/terrarium/{z}/{x}/{y}.png',
      //   elevationDecoder: {
      //     rScaler: 256,
      //     gScaler: 1,
      //     bScaler: 1 / 256,
      //     offset: -32768,
      //   },
      //   texture: null,
      //   maxZoom: 14,
      //   material: {
      //     diffuse: 1,
      //   },
      //   operation: 'terrain+draw',
      // }),
      new TerrainLayer({
        id: 'terrain',
        minZoom: 0,
        maxZoom: 23,
        strategy: 'no-overlap',
        elevationDecoder: ELEVATION_DECODER,
        elevationData: TERRAIN_IMAGE,
        texture: `https://api.mapbox.com/v4/mapbox.satellite/{z}/{x}/{y}@2x.png?access_token=${MAPBOX_TOKEN}`,
        color: [255, 255, 255],
      }),
    ],
  })
})
</script>

<style>
#test {
  height: 100vh;
  width: 100vw;
}
</style>
