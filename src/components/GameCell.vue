<template>
  <div
    class="game-cell"
    @click="$emit('click')"
    :style="{
      backgroundImage: owner ? `url('imgs/stone_${owner}.svg')` : 'none',
      border: put ? `3px solid khaki` : '1px solid gray'
    }"
  ></div>
</template>

<script>
export default {
  props: {
    owner: { type: Number },
    put: { type: Boolean }
  }
};
</script>

<style lang="scss" scoped>
.game-cell {
  border: 1px solid gray;
  background-color: darkgreen;
  background-image: url();
}
</style>
