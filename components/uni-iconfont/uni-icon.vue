<template>
	<view :class="'uni-icon-' + type" :style="{ color: color, 'font-size': size + 'px' }" class="uni-icon" @click="_onClick" />
</template>

<script>
	export default {
		name: 'UniIconfont',
		props: {
			type: {
				type: String,
				default: ''
			},
			color: {
				type: String,
				default: '#333333'
			},
			size: {
				type: [Number, String],
				default: 16
			}
		},
		methods: {
			_onClick() {
				this.$emit('click')
			}
		}
	}
</script>

<style>
	@font-face {
		font-family: uniiconfont;
		font-weight: normal;
		font-style: normal;
		src: url(data:application/font-woff2;charset=utf-8;base64,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) format('woff2');
	}

	uni-icon {
		line-height: 1;
	}

	.uni-icon {
		font-family: uniiconfont;
		font-weight: normal;
		font-style: normal;
		line-height: 1;
		display: inline-block;
		text-decoration: none;
		-webkit-font-smoothing: antialiased;
	}

	.uni-icon.uni-active {
		color: #007aff;
	}

	.uni-icon-shdz:before {
	  content: "\e9fb";
	}
	
	.uni-icon-qbdd:before {
	  content: "\e615";
	}
	
	.uni-icon-dsh:before {
	  content: "\e60a";
	}
	
	.uni-icon-qbdd2:before {
	  content: "\e604";
	}
	
	.uni-icon-dfk:before {
	  content: "\e649";
	}
	
	.uni-icon-shdd2:before {
	  content: "\e611";
	}
	
	.uni-icon-shdd3:before {
	  content: "\e601";
	}
	
	.uni-icon-gwd:before {
	  content: "\e652";
	}
	
	.uni-icon-car:before {
	  content: "\e607";
	}
	
	.uni-icon-ax:before {
	  content: "\e608";
	}
	
	.uni-icon-sp:before {
	  content: "\e60f";
	}
	
	.uni-icon-gx:before {
	  content: "\e610";
	}
	
	.uni-icon-wgx:before {
	  content: "\e623";
	}
</style>