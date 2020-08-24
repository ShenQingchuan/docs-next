<template>
  <div class="ad-pagetop">
    <a
      href="http://www.vueshop.com.cn/index.php?http_referer=vuejs"
      target="_blank"
      rel="noopener noreferrer"
    >
      <span class="name">VUEshop</span>
      <span class="description">中国免费商城系统 - 搜豹商城系统 - 免费50小时 Vue 视频教程</span>
      <span class="action">立即查看 &gt;</span>
    </a>
  </div>
</template>

<script>
export default {}
</script>

<style lang="stylus" scoped>
$info = #1c90f3;

.ad-pagetop {
  background: #f8f8f8;
  padding: 8px 16px;
  border-radius: 4px;
  clear: both;
  position: relative;
  top: 58px;

  a {
    display: block;
  }

  span {
    color: $info;
    display: inline-block;

    &.name {
      color: $info;
      font-size: 1.1em;
      font-weight: 600; 
    }

    &.description {
      margin-left: 0.8em;
      font-size: 0.9em;
      color: #666;
      font-weight: normal;
    }

    &.action {
      font-size: 0.75em;
      color: #888;
      border: 1px solid #888;
      border-radius: 3px;
      padding: 2px 6px;
      margin-left: 1em;
    }
  }

  &:after {
    content: '广告';
    font-size: 10px;
    position: absolute;
    top: 50%;
    color: #aaa;
    padding: 3px 6px;
    border: 1px solid #ccc;
    border-radius: 3px;
    margin-top: -9px;
    right: 10px;
  }
}

@media screen and (max-width: 560px) {
  .ad-pagetop {
    display: none;
  }
}
</style>
