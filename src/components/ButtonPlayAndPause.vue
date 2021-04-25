<template>
  <button class="main-btn play" @click="$emit('resume')">
    <svg width="35" height="43" viewBox="0 0 31 43" fill="none" xmlns="http://www.w3.org/2000/svg" v-if="!isPlaying">
      <path
          d="M5.61646 0.939207C2.5147 -1.26362 0 0.540962 0 4.96664V37.6262C0 42.0563 2.5147 43.8585 5.61646 41.6578L28.6729 25.2868C31.7757 23.0832 31.7757 19.513 28.6729 17.3099L5.61646 0.939207Z"
          fill="black"/>
    </svg>

    <svg width="31" height="38" viewBox="0 0 31 38" fill="none" xmlns="http://www.w3.org/2000/svg" v-else>
      <path
          d="M10.5858 32.7071C10.5858 35.6301 8.21589 38 5.29288 38C2.36986 38 0 35.6301 0 32.7071V5.29288C0 2.36986 2.36986 0 5.29288 0C8.21589 0 10.5858 2.36986 10.5858 5.29288V32.7071Z"
          fill="black"/>
      <path
          d="M30.4498 32.7071C30.4498 35.6301 28.0799 38 25.1569 38C22.2339 38 19.8641 35.6301 19.8641 32.7071V5.29288C19.8649 2.36986 22.2347 0 25.1569 0C28.0799 0 30.4498 2.36986 30.4498 5.29288V32.7071Z"
          fill="black"/>
    </svg>
  </button>
</template>

<script>
export default {
  name: "ButtonPlayAndResume",
  props: ['isPlaying']
}
</script>

<style scoped>

</style>