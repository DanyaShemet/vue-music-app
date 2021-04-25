<template>
  <div class="middle_block_title song-title">

    <h2>{{ current.title }}</h2>
    <p>{{ current.artist }}</p>
  </div>
</template>

<script>
export default {
  name: "TitleSong",
  props: ['current']
}
</script>

<style scoped>

</style>