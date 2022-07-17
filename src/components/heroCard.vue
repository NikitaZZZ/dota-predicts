<template>
  <div class="col mb-3">
    <div class="card" style="width: 12rem">
      <div class="card-body">
        <h5 class="card-title">{{ heroStats.heroName }}</h5>
        <p class="card-text">{{ heroStats.percentWins }}%</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'heroCard',
  props: {
    heroStats: {
      type: Object,
      required: false,
    },
  },
};
</script>

<style>
.card {
  cursor: pointer;
}
</style>
