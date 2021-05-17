<template>
  <header class="py-8 bg-blue-800 text-green-200">
    <button @click="toInitialPage">
      <h1 class="font-bold">Welcome to Ryd-Pay</h1>
    </button>
  </header>
</template>
<script>
export default {
  methods: {
    toInitialPage() {
      this.$emit("onHeaderClick");
    },
  },
};
</script>
