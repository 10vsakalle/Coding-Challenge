<template>
  <div class="home h-screen bg-green-200">
    <div class="bg-green-200 h-auto">
      <Header @onHeaderClick="gasStationList = null" class="mb-4" />

      <div class="px-4 justify-start">
        <p id="userLocationStatus"></p>

        <DropDown
          v-if="shouldShowStation"
          v-model="selectedBrand"
          :options="brandList"
          placeholder="Select Brand"
          close-on-select
          class="mb-2"
        />

        <!-- For quick dropdown list, vue-next-select component is installed. For more information (https://www.npmjs.com/package/vue-next-select) -->
        <DropDown
          v-model="selectedFuelType"
          :options="fuelTypeList"
          close-on-select
          placeholder="Select Fuel"
          class="mb-2"
        ></DropDown>

        <input
          type="number"
          id="radius"
          name="radius"
          min="1"
          max="25"
          v-model="radius"
          class="text-center rounded-md justify-start mr-2"
        />
        <label for="radius" class="mr-2">Enter search area in KM</label>

        <button
          @click="getStations"
          class="
            bg-red-300
            text-blue-800
            mt-2
            w-full
            mb-4
            rounded-lg
            py-2
            border-blue-800 border-solid border-2
          "
        >
          Find the nearest Gas-Station
          <!-- By default, Stations are sorted by Distance. 
               If user selects Fuel Type, stations will get sort by price.
           -->
        </button>

        <div v-if="gasStationList">
          <StationCard
            v-for="(station, index) in gasStationList"
            v-bind:key="index"
            :station="station"
            :selectedFuelType="selectedFuelType"
            class="mx-auto mb-4"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "@/components/Header";
import VueSelect from "vue-next-select";
import StationCard from "@/components/StationCard";
export default {
  components: {
    Header,
    DropDown: VueSelect,
    StationCard,
  },
  data() {
    return {
      gasStationList: null,
      shouldShowStation: false,
      brandList: [],
      fuelTypeList: ["e10", "e5", "diesel"],
      selectedBrand: "",
      selectedFuelType: "",
      lat: "", //lattitude
      lon: "", //longitude
      url: "",
      radius: 25, //(in KM)
    };
  },
  methods: {
    getStations() {
      this.getUserLocation();
    },

    //getUserLocation() -> To access geographical location information associated with the hosting device
    //For more info : https://w3c.github.io/geolocation-api/#dom-navigator-geolocation
    getUserLocation() {
      const status = document.querySelector("#userLocationStatus");
      if (navigator.geolocation) {
        navigator.geolocation.getCurrentPosition(
          this.showPosition,
          this.showError
        );
      } else {
        status.textContent = "Geolocation is not supported by this browser.";
      }
    },

    //showPosition() & showError() - callback functions for navigator.geolocation.getCurrentPosition
    showPosition(position) {
      this.lat = parseFloat(this.round(position.coords.latitude, 3)); //rounding floating point numbers to 3 decimal places
      this.lon = parseFloat(this.round(position.coords.longitude, 3));
      console.log(typeof this.lat, typeof this.lon);
      this.createURL();
    },
    showError(error) {
      const status = document.querySelector("#userLocationStatus");
      switch (error.code) {
        case error.PERMISSION_DENIED:
          status.textContent = "User denied the request for Geolocation.";
          break;
        case error.POSITION_UNAVAILABLE:
          status.textContent = "Location information is unavailable.";
          break;
        case error.TIMEOUT:
          status.textContent = "The request to get user location timed out.";
          break;
        case error.UNKNOWN_ERROR:
          status.textContent = "An unknown error occurred.";
          break;
      }
    },

    //Once location is received, createURL() functions creates the url with parameters(brand, fuel type and radius) selected by user to fetch data
    createURL() {
      //To check if radius has a valid value.
      //If not then radius will be 25 by default
      if (this.radius <= 0) {
        //To check if fuel type is selected.
        //If yes, stations will get sort by fuel price in given radius
        if (this.selectedFuelType !== "") {
          this.url = `https://creativecommons.tankerkoenig.de/json/list.php?lat=${this.lat}&lng=${this.lon}&rad=25&sort=price&type=${this.selectedFuelType}&apikey=ac543711-c782-4dfb-c0d8-2dc703eaac9b`;
          //If No, stations will get sort by distance in given radius
        } else {
          this.url = `https://creativecommons.tankerkoenig.de/json/list.php?lat=${this.lat}&lng=${this.lon}&rad=25&sort=dist&type=all&apikey=ac543711-c782-4dfb-c0d8-2dc703eaac9b`;
        }
        //If radius has a valid value
      } else {
        if (this.selectedFuelType !== "") {
          this.url = `https://creativecommons.tankerkoenig.de/json/list.php?lat=${this.lat}&lng=${this.lon}&rad=${this.radius}&sort=price&type=${this.selectedFuelType}&apikey=ac543711-c782-4dfb-c0d8-2dc703eaac9b`;
        } else {
          this.url = `https://creativecommons.tankerkoenig.de/json/list.php?lat=${this.lat}&lng=${this.lon}&rad=${this.radius}&sort=dist&type=all&apikey=ac543711-c782-4dfb-c0d8-2dc703eaac9b`;
        }
      }
      this.fetchDataFromAPI(this.url); //To fetch data from API
    },
    async fetchDataFromAPI(url) {
      const status = document.querySelector("#userLocationStatus");
      const res = await fetch(url).then((res) => {
        if (!res.ok) {
          status.textContent = res.message;
        }
        return res;
      });
      const data = await res.json();
      const tempStationList = data.stations;
      //To check if Brand is selected
      if (this.selectedBrand !== "") {
        //If yes, filter gas stations for selected brand.
        //Also no need to create filter list for brand again since it already has a value.
        //No call to createFilterList()
        this.gasStationList = tempStationList.filter(
          (el) => el.brand === this.selectedBrand
        );
        this.shouldShowStation = true;
      } else {
        //If not selected, show all the stations
        //Create list of brands available in the selected radius using createFilterList
        this.gasStationList = tempStationList;
        this.shouldShowStation = true;
        this.createFilterList();
      }
      //resetting the filter value after fetch
      this.selectedBrand = "";
      this.selectedFuelType = "";
    },
    //To round floating point numbers - lat & lon
    round(value, decimals) {
      return Number(Math.round(value + "e" + decimals) + "e-" + decimals);
    },
    //to create brand filter list
    createFilterList() {
      if (this.gasStationList !== null && this.brandList !== "") {
        const tempBrandList = this.gasStationList
          .map((a) => a.brand) //creating array of brands
          .filter(Boolean) // removing empty array elements
          .map(function (e) {
            return e.toUpperCase(); // converting each element to uppercase to remove duplicate values in next step
          });
        const seen = new Set();
        this.brandList = tempBrandList.filter((el) => {
          const duplicate = seen.has(el);
          seen.add(el);
          return !duplicate;
        });
      }
    },
  },
};
</script>
<style>
.vue-input input {
  padding-left: 10px;
  width: 100%;
  margin-bottom: 16px;
  border-radius: 0.375rem;
}
</style>
