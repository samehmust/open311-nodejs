<template>
  <div>
    {{loading}}
    <div class="form-group">
      <div v-if="loading" class="loader-wrapper">
        <div class="bar"></div>
      </div>

      <label for="location">Location:</label>
      <input v-model="location.address_string"
             type="text"
             id="location"
             disabled="disabled">
    </div>

    <h3 v-if="loading">Detecting your location.</h3>

    <div id="mapid"></div>
  </div>
</template>

<style type="text/css">
  .loader-wrapper {
    position: absolute;
    top: 42px;
  }

  #mapid {
    width: 100%;
    height: 500px;
    margin: 0 0 20px 0;
  }
</style>

<script>
import L from 'leaflet';
import axios from 'axios'

export default {
  head () {
    return {
      script: [
        { src: './leaflet/leaflet.js' }
      ],
      link: [
        { rel: 'stylesheet', href: './leaflet/leaflet.css' }
      ]
    }
  },
  data() {
    return {
      loading:          false,
      location: {
        lat:            null,
        long:           null,
        address_string: null
      },
      map:              null
    }
  },
  mounted() {
    var self = this;
    self.loading = true;

    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(displayLocationInfo);
    }

    function displayLocationInfo(position) {
      self.location.lat = position.coords.latitude;
      self.location.long = position.coords.longitude;
      self.initMap();
      console.log(`Original -- latitude:`
                  + `${self.location.lat} | `
                  + `longitude: ${self.location.long}`
      );
    }
  },
  watch: {
    locationUpdate: function() {
      this.updateAddressString(this.location.lat,this.location.long);
    }
  },
  methods: {
    updateAddressString(lat,long){
      axios.get(`${process.env.osmUrl}&lat=${lat}&lon=${long}`)
      .then(response => {
        this.loading = false;
        this.location.address_string = response.data.display_name;
      })
      .catch(function (error) {
        this.loading = false;
        alert(`Error happened: ${error}`);
      })
      .then(function () {});
    },
    initMap() {
      var self  = this;
      var mymap = L.map('mapid').setView([self.location.lat,self.location.long], 20);

      var crosshairIcon = L.icon({
        iconUrl:      '/crosshair.png',
        iconSize:     [150, 277],
        iconAnchor:   [70, 146],
      });

      var crosshair = new L.marker(mymap.getCenter(),{
        icon:       crosshairIcon,
        clickable:  false
      });
      crosshair.addTo(mymap);

      mymap.on('move', function(ev) {
        crosshair.setLatLng(mymap.getCenter());
      });

      mymap.on('moveend', function(ev) {
        self.location.lat = mymap.getCenter().lat;
        self.location.long = mymap.getCenter().lng;
        console.log(`Updated  -- latitude:`
                  + `${self.location.lat} | `
                  + `longitude: ${self.location.long}`
        );
      });

      L.tileLayer(process.env.mapBoxUrl,{
        attribution: 'Map data &copy; <a href="https://www.openstreetmap.org/">OpenStreetMap</a> contributors, <a href="https://creativecommons.org/licenses/by-sa/2.0/">CC-BY-SA</a>, Imagery © <a href="https://www.mapbox.com/">Mapbox</a>',
        maxZoom:      18,
        setView:      true,
        detectRetina: true,
        id:           'mapbox.streets',
        accessToken:  `${process.env.mapBoxKey}`
      }).addTo(mymap);
    }
  },
  computed: {
    locationUpdate() {
      return this.location.lat
      return this.location.long
      return this.location.address_string
    }
  }
}
</script>