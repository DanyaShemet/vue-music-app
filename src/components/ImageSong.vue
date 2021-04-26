<template>
  <div class="top_block_img">
    <img :src="src" alt="" class="background__artist_img" ref="image">
    <img :src="src" alt="" class="artist__img">
  </div>
</template>

<script>

export default {
  name: "ImageSong",
  props: ['src'],

}
</script>

<style scoped>

</style>