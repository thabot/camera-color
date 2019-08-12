<template>
  <div class="home">
    <!-- <img alt="Vue logo" src="../assets/logo.png"> -->
    <ShowImage/>
  </div>
</template>

<script>
// @ is an alias to /src
import HelloWorld from '@/components/HelloWorld.vue'
import ShowImage from '@/components/ShowImage.vue'

export default {
  name: 'home',
  components: {
    ShowImage
  }
}
</script>
