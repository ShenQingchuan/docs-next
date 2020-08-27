<template>
  <div class="dcloud-video">
    <a :href="href" target="_blank" rel="sponsored noopener">{{ title }}</a>
  </div>
</template>

<script>
export default {
  props: {
    title: String,
    href: String
  }
}
</script>

<style lang="scss">
.dcloud-video {
  background-color: #e7ecf3;
  padding: 1em 1.25em;
  border-radius: 2px;
  position: relative;
  margin: 20px 0;

  a {
    margin-left: 34px;
    color: #486491;
  }
  a:hover {
    text-decoration: none !important;
  }

  a:before {
    content: '';
    position: absolute;
    display: block;
    width: 30px;
    height: 30px;
    top: 12px;
    left: 14px;
    border-radius: 50%;
    background-color: #73abfe;
  }

  a:after {
    content: '';
    position: absolute;
    display: block;
    width: 0;
    height: 0;
    top: 22px;
    left: 26px;
    border-top: 5px solid transparent;
    border-bottom: 5px solid transparent;
    border-left: 8px solid #fff;
  }
}
</style>
