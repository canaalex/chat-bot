<script setup>
import { ref } from 'vue'


defineProps({
  msg: String,
})

const count = ref(0)
</script>

<template>

<div class="relative h-screen">
  <div className="bg-dark-blue px-10 h-500px rounded-xl w-96">
    <div className="text-white">
      Jack Ma
    </div>

  </div>
 
</div>
  
</template>

<style scoped>
.read-the-docs {
  color: #888;
}
</style>
