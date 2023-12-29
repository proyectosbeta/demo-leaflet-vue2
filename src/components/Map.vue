<template>
  <div id="mapContainer" @mouseout="deselect"></div>
</template>

<script>
import "leaflet-routing-machine/dist/leaflet-routing-machine.css";
import L from "leaflet";
import "leaflet-routing-machine";
import states from "@/assets/India.json";

export default {
  name: "MapTest",
  data() {
    return {
      center: [22.21908650451422, 79.591992793990354],
      zoom: 5,
      selectedLayer: null,
      mapDiv: null,
      geojson: null,
    };
  },
  methods: {
    setupLeafletMap() {
      this.mapDiv = L.map("mapContainer").setView(this.center, this.zoom);
      this.addTileLayer();
      this.addGeoJSONLayer();
      this.addRouting();
    },

    addTileLayer() {
      L.tileLayer(
        "https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token={accessToken}",
        {
          attribution:
            'Map data (c) <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery (c) <a href="https://www.mapbox.com/">Mapbox</a>',
          maxZoom: 18,
          id: "mapbox/streets-v11",
          accessToken: process.env.VUE_APP_API_MAPBOX_KEY,
          crs: L.CRS.EPSG4326,
        }
      ).addTo(this.mapDiv);
    },

    addGeoJSONLayer() {
      this.geojson = L.geoJSON(states, {
        onEachFeature: this.onEachFeature,
        style: this.mapstyle(),
      }).addTo(this.mapDiv);
    },

    addRouting() {
      L.Routing.control({
        waypoints: [L.latLng(28.238, 83.9956), L.latLng(38.238, 93.9956)],
        routeWhileDragging: true,
        language: "de",
      }).addTo(this.mapDiv);
    },

    onEachFeature(feature, layer) {
      layer.bindPopup("<h1>" + feature.properties.name + "</h1>");

      layer.on({
        mouseover: (e) => this.highlight(e.target),
        mouseout: (e) => this.dehighlight(e.target),
        click: (e) => this.select(e.target),
      });
    },

    mapstyle() {
      return {
        fillColor: "#f56",
        color: "#000",
        weight: "1",
        fillOpacity: 0.5,
      };
    },

    highlight(layer) {
      layer.setStyle({
        weight: 2,
        dashArray: "",
        fillColor: "yellow",
      });
      if (!L.Browser.ie && !L.Browser.opera) {
        layer.bringToFront();
      }
    },

    dehighlight(layer) {
      if (this.selectedLayer === null || this.selectedLayer !== layer) {
        this.geojson.resetStyle(layer);
      }
    },

    select(layer) {
      if (this.selectedLayer !== null) {
        this.dehighlight(this.selectedLayer);
      }
      this.mapDiv.fitBounds(layer.getBounds());
      this.selectedLayer = layer;
    },

    deselect() {
      if (this.selectedLayer) {
        this.dehighlight(this.selectedLayer);
        this.selectedLayer.closePopup();
        this.selectedLayer = null;
      }
    },
  },
  mounted() {
    this.setupLeafletMap();
  },
};
</script>

<style scoped>
#mapContainer {
  width: 98vw;
  height: 98vh;
}
</style>
