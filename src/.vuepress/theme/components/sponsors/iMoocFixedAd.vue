<template>
  <div id="imooc-ad3">
    <span style="float: right;">广告</span>
    <a
      href="https://coding.imooc.com/?c=vuejs&amp;mc_marking=2243e4b1af42da32ee6e9f088e646c7c&amp;mc_channel=vuezwsq"
      target="_blank"
      rel="nofollow"
    >
      <img src="/images/imooc-ad3.png" alt="Vue.js实战项目开发教程" />
    </a>
  </div>
</template>

<style lang="scss" scoped>
#imooc-ad3 {
	width: 125px;
	position: fixed;
	z-index: 19;
	bottom: 10px;
	right: 10px;
	padding: 10px;
	background-color: #fff;
	border-radius: 3px;
	font-size: 13px;

	a {
		display: inline-block;
		color: #4f5959;
		font-weight: normal;
	}

	img {
		width: 125px;
	}

	span {
		color: #4f5959;
		display: block;

		img {
			width: 125px;
		}
	}

	.carbon-img,
	.carbon-text {
		display: block;
		margin-bottom: 4px;
		font-weight: normal;
		color: #304455;
	}
	.carbon-poweredby {
		color: #aaa;
		font-size: 10px;
		letter-spacing: 0.8px;
		text-transform: uppercase;
		font-weight: normal;
	}
}
</style>