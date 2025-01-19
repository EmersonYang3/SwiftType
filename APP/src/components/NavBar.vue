<template>
  <div class="navbar bg-base-100">
    <div class="flex-1">
      <a class="btn btn-ghost text-xl" @click="routeHome">SwiftType</a>
    </div>
    <div class="flex-none">
      <ul class="menu menu-horizontal px-1">
        <li><a @click="routeToContributer">Contributers</a></li>
      </ul>
    </div>
  </div>
  <div class="w-full h-[1px] bg-white"></div>
</template>

<script setup lang="ts">
import router from '@/router'

function routeToContributer() {
  router.push({ name: 'ContributerView' })
}

function routeHome() {
  router.push({ name: 'Home' })
}
</script>
