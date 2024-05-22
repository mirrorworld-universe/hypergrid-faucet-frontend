<template>
  <header>
    <nav>
      <img src="@/assets/images/logo2.png" alt="" class="logo" />
    </nav>
  </header>
</template>

<script lang="ts" setup></script>

<style lang="scss" scoped>
header {
  width: 100%;
  height: 66.5px;
  position: fixed;
  z-index: 99;
  border-bottom: 1px solid #ffffff1a;
  background: #03081c;
}

nav {
  width: 100%;
  height: 100%;
  margin: 0 auto;
  padding: 0 16px;
  display: flex;
  align-items: center;
  .logo {
    width: auto;
    height: 40px;
  }
}

@media screen and (max-width: 750px) {
  nav {
    width: 90vw;
  }
}
</style>
