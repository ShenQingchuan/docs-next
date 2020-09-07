<template>
  <main class="page">
    <slot name="top" />

    <VUEshopAd />
    <Content class="theme-default-content" />
    <iMoocFixedAd />
    
    <PageEdit />
    <PageNav v-bind="{ sidebarItems }" />

    <slot name="bottom" />
  </main>
</template>

<script>
import VUEshopAd from '@theme/components/sponsors/VUEshopAdvertisement.vue'
import iMoocFixedAd from '@theme/components/sponsors/iMoocFixedAd.vue'
import PageEdit from '@theme/components/PageEdit.vue'
import PageNav from '@theme/components/PageNav.vue'

export default {
  components: { PageEdit, PageNav, VUEshopAd, iMoocFixedAd },
  props: ['sidebarItems'],
}
</script>

<style lang="stylus">
@require '../styles/wrapper.styl';

.page {
  padding-bottom: 2rem;
  display: block;
}
</style>
