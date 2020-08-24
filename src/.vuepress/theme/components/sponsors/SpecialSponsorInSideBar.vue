<template>
  <div class="special-sponsors-in-sidebar">
    <div class="title">特别赞助商</div>

    <div class="sponsor-logo" v-for="sponsor of sponsors.special_sponsors">
      <a target="_blank">
        <img :src="`/images/sponsors/${sponsor.sidebar_img}`" :alt="sponsor.name" width="200" />
      </a>
    </div>
  </div>
</template>

<script>
import sponsors from '@theme/data/patreon-sponsors.js'

export default {
  data: () => ({
    sponsors,
  }),
}
</script>

<style lang="stylus" scoped>
.special-sponsors-in-sidebar {
  color: #4f5959;
  font-size: 0.85em;
  padding: 20px 20px 0 20px;

  .title {
    margin-bottom: 16px;
  }

  .sponsor-logo {
    a {
      margin: 10px auto;
    }
  }
}
</style>
