<script setup>
import { ref, computed, watch } from "vue";
import { GoogleMap, Marker } from "vue3-google-map";

const geocoder = ref(null);

const clickMap = (e) => {
  console.log("clickMap");
  lng.value = e.latLng.lng();
  lat.value = e.latLng.lat();
  console.log(markerRef.value);

  console.log("lng", lng.value);
  console.log("lat", lat.value);
  markerRef.value.marker.setPosition({ lat: lat.value, lng: lng.value });

  geocoder.value.geocode(
    { location: { lat: lat.value, lng: lng.value } },
    (results, status) => {
      if (status === "OK") {
        if (results[0]) {
          address.value = results[0].formatted_address;
          console.log("address", address.value);
        } else {
          window.alert("No results found");
        }
      } else {
        window.alert("Geocoder failed due to: " + status);
      }
    }
  );

};
const address = ref('');

const callbackAutocomplete = (place) => {
  console.log(place);
  lng.value = place.geometry.location.lng();
  lat.value = place.geometry.location.lat();

  markerRef.value.marker.setPosition({ lat: lat.value, lng: lng.value });
  address.value = place.formatted_address;
  console.log("lng", lng.value);
  console.log("lat", lat.value);
  console.log("address", address.value);
}


const mapRef = ref(null);
const markerRef = ref(null);
const autocompleteRef = ref(null);

const markerPosition = ref({ lat: 0, lng: 0 });

const lng = computed({
  get: () => markerPosition.value.lng,
  set: (v) => {
    if (!Number.isFinite(v)) {
      markerPosition.value.lng = 0;
    } else if (v > 180) {
      markerPosition.value.lng = 180;
    } else if (v < -180) {
      markerPosition.value.lng = -180;
    } else {
      markerPosition.value.lng = v;
    }
  },
});

const lat = computed({
  get: () => markerPosition.value.lat,
  set: (v) => {
    if (!Number.isFinite(v)) {
      markerPosition.value.lat = 0;
    } else if (v > 90) {
      markerPosition.value.lat = 90;
    } else if (v < -90) {
      markerPosition.value.lat = -90;
    } else {
      markerPosition.value.lat = v;
    }
  },
});

// Third pattern: watch for "ready" then do something with the API or map instance
watch(() => mapRef.value?.ready, (ready) => {
      if (!ready) return

      console.log('The map is ready!')
      console.log(mapRef.value.api)


      const options = {
          fields: ["formatted_address", "geometry", "name"],
          strictBounds: false,
        };

        const autocomplete = new mapRef.value.api.places.Autocomplete(
          autocompleteRef.value,
          options
        );


        geocoder.value = new mapRef.value.api.Geocoder();

        autocomplete.addListener("place_changed", () => {
          const place = autocomplete.getPlace();
          if (!place.geometry || !place.geometry.location) {
            console.log("No details available for input: '" + place.name + "'");
            return;
          }
          callbackAutocomplete(place);
        });
      // do something with the api using `mapRef.value.api`
      // or with the map instance using `mapRef.value.map`
    })

</script>

<template>
  <div>
    <h1>Maps</h1>
    <GoogleMap ref="mapRef" api-key="AIzaSyDztKIwCgUXyASJK_YYYzz8xKk_1b2osVo" class="map" :center="markerPosition" :zoom="2"
      @click="clickMap">
      <Marker ref="markerRef" :options="{ position: markerPosition }" />
    </GoogleMap>
    <input v-model="address" type="text" ref="autocompleteRef" />
    <label for="lng">Longitude</label>
    <input v-model.number="lng" id="lng" type="number" min="-180" max="180" step="10" />
    <label for="lat">Latitude</label>
    <input v-model.number="lat" id="lat" type="number" min="-90" max="90" step="10" />
  </div>
</template>

<style scoped>
.map {
  position: relative;
  width: 100%;
  height: 500px;
}

/* .map::after {
  position: absolute;
  content: "";
  width: 1px;
  height: 100%;
  top: 0;
  left: 50%;
  background: red;
} */

input[type="number"] {
  width: 200px;
  margin-top: 20px;
  margin-left: 10px;
}
</style>
