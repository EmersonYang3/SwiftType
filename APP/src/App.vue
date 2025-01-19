<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import { ref, onMounted } from 'vue'

const Splash = ref(true)

onMounted(() => {
  setTimeout(() => {
    Splash.value = false
  }, 3000)
})
</script>

<template>
  <div class="relative">
    <RouterView />
    <transition name="fade">
      <div v-if="Splash" class="fixed inset-0 flex items-center justify-center bg-black z-50">
        <img
          src="https://media.discordapp.net/attachments/1281382118651461725/1330681645484740710/SwiftType_1.png?ex=678edd5b&is=678d8bdb&hm=f8837f6041f1b666697855bdd0b3caa7360341dec1511091a2c3a0305b79bee0&=&format=webp&quality=lossless&width=1244&height=700"
          alt="SwiftType Logo"
          class="w-1/2 h-auto pulsate"
          draggable="false"
        />
      </div>
    </transition>
  </div>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 1s;
}
.fade-enter,
.fade-leave-to {
  opacity: 0;
}

@keyframes pulsate {
  0% {
    opacity: 1;
  }
  50% {
    opacity: 0.8;
  }
  100% {
    opacity: 1;
  }
}

.pulsate {
  animation: pulsate 1s ease-in-out infinite;
}
</style>
