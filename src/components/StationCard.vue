<template>
  <div class="bg-blue-800 text-green-200 p-2 rounded-lg">
    <div class="text-left">
      <p>{{ station.name }} <br /></p>
      <p>Distance : {{ station.dist }}</p>
    </div>
    <div v-if="!station.price && !selectedFuelType" class="flex justify-left">
      <p class="pr-2">diesel: {{ station.diesel }}</p>
      <p class="pr-2">e5 : {{ station.e5 }}</p>
      <p class="pr-2">e10 : {{ station.e10 }}</p>
    </div>
    <div v-if="station.price" class="text-left">
      <p>Price : {{ station.price }}</p>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    station: {
      type: Object,
      default() {
        return {};
      },
    },
    selectedFuelType: {
      type: String,
      default: "",
    },
  },
};
</script>
