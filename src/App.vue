<template>
  <a-config-provider
    :theme="{
      algorithm: theme.darkAlgorithm,
      token: {
        colorPrimary: '#0000ff'
      }
    }">
    <a-app>
      <HeaderView />
      <main>
        <router-view></router-view>
      </main>
    </a-app>
  </a-config-provider>
</template>

<script setup lang="ts">
import HeaderView from '@/layout/HeaderView.vue';
import { theme } from 'ant-design-vue';
</script>

<style lang="scss">
main {
  padding-top: 100px;
  min-height: 100vh;
  position: relative;
  z-index: 9;
  section {
    width: 100%;
    height: 100%;
  }
}

@media screen and (max-width: 750px) {
  main {
  }
}
</style>
