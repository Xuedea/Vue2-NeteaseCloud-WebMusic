<template>
  <div class="github">
    <ay-svg-icon class="icon" icon="github" @click.stop="openGithub" />
  </div>
</template>

<script>
export default {
  name: "aGithub",
  methods: {
    openGithub() {
      window.open("https://github.com/Xuedea/Vue2-NeteaseCloud-WebMusic");
    },
  },
};
</script>

<style lang="scss" scoped>
.github {
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  .icon {
    font-size: 24px;
    cursor: pointer;
  }
}
</style>
