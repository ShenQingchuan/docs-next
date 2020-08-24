<template>
  <SponsorsLayout extra-classes="sponsors__video-delegate">
    <template #heading>中国区视频托管赞助商</template>

    <template #content>
      <img class="sponsor-logo" src="https://cn.vuejs.org/images/youku.png" alt />
    </template>
  </SponsorsLayout>
</template>

<script>
import SponsorsLayout from '@theme/components/sponsors/SponsorsLayout.vue'

export default {
  components: {
    SponsorsLayout,
  },
}
</script>

<style lang="scss" scoped>
.sponsor-logo {
  width: 120px;
}
</style>
