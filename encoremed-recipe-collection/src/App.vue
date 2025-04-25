<template>
  <main>
    <NavBar />
    <div class="container">
      <router-view />
    </div>
  </main>
</template>



<script>
import NavBar from './components/NavBar.vue'

export default {
  components: { NavBar }
}
</script>
