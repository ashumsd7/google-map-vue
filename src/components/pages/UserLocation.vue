<template>
<div class="">
      <section class="ui two column grid " style="position:relative; z-index:1;">
    <div class="column">
      <form class="ui segment form large">
        <div class="ui message red" v-show="error">
          {{ error }}
        </div>
        <div class="ui segment">
          <div class="field">
            <div class="ui right icon input large">
              <input
                type="text"
                name="inout"
                placeholder="Enter Location"
                id="autocomplete"
                v-model="address"
              />
              <!-- <i class="circle icon"   style="background: pink; color: white"></i> -->
            </div>
          </div>
          <button class="ui button my-2">
            Search <i class="mark icon"></i>
          </button>
          <br />
          <button @click="fetchLocation" class="ui button">
            <i class="circle icon" @click="fetchLocation"></i>
          </button>
        </div>
      </form>
    </div>
  </section>
  <section id="map"></section>
</div>
</template>

<script>
import axios from "axios";
export default {
  data() {
    return {
      // axios:axios
      address: "",
      error: "",
    };
  },

  // init auto complete

  mounted() {
  let autocomplete=  new google.maps.places.Autocomplete(
      document.getElementById("autocomplete"),
    // this.$refs['autocomplete'],
      {
        bounds: new google.maps.LatLngBounds(new google.maps.LatLng(12.58, 77.35)),
      }
    );


    autocomplete.addListener('place_changed', ()=>{
     let place=   autocomplete.getPlace()
     console.log(place);
     this.showAddressOnMap(place.geometry.location.lat(),place.geometry.location.lng())
    })
  },

  methods: {
    fetchLocation(e) {
      e.preventDefault();
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          this.successFunction,
          this.errorFunction
        );
      } else {
        console.warn(
          "Browser doesn't support location feature. Try different browser. "
        );
      }
    },

    successFunction(pos) {
      console.log(pos.coords.latitude);
      console.log(pos.coords.longitude);

      this.getAddress(pos.coords.latitude, pos.coords.longitude);


      this.showAddressOnMap(pos.coords.latitude, pos.coords.longitude)
    },
    errorFunction(err) {
      console.error(err.message);
    },


// show marker ton map
// init map
    showAddressOnMap(lat,lng){
    
    let map = new google.maps.Map(document.getElementById('map'),{
        zoom:15,
        center: new google.maps.LatLng(lat,lng),
    mapTypeId:google.maps.MapTypeId.ROADMAP
    })


    new google.maps.Marker(
        {
            position: new google.maps.LatLng(lat,lng),
            map:map,
        }
    )

    },

    // AJAX geocoding API : Street address to lat and long
    // we want street address from lat and  long which is reverse geo codings

    getAddress(lat, lng) {
      axios
        .get(
          `https://maps.googleapis.com/maps/api/geocode/json?latlng=${lat},${lng}&key=AIzaSyBOqEO_vBg6U-GKE-FUQfx1MWJ-I1mVSzQ`
        )
        .then((res) => {
          if (res.data.error_message) {
            console.warn("Error occurred", res.data.error_message);
            this.error = res.data.error_message;
          } else {
            this.error = "";
            console.log("res is", res.data.results[0].formatted_address);
            this.address=res.data.results[0].formatted_address
          }
        })
        .catch((err) => {
          console.log(err);
          this.error = err.message;
        });
    },

    // place auto complete
  },
};
</script>

<style>
.ui button,
.dot.circle.icon {
  background-color: #ff5a5f;
  color: white;
}
.pac-icon{
    display: none;
}
.pac-item{
    padding: 10px;
    font-size: 14px;
    cursor: pointer;
}
.pac-item:hover{
    background-color: #eee;
}
.pac-item-query{
       font-size: 16px;
}
#map{
    position: absolute;
    top:0;
    left: 0;
    bottom: 0;
    right: 0;
    /* background-color: red; */
}
</style>
