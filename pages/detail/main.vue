<template>
	<view class="content" v-if="goodInfo._id">
		<view class="wrap-good">
			<view class="wrap-swiper">
				<uni-swiper-dot :info="covers" :current="current" mode="long" :dots-styles="dotsStyles">
					<swiper class="swiper-box" :style="{height:bannerHeight+'px'}" @change="change">
						<swiper-item v-for="(item,index) in covers" :key="index">
							<view :class="item.colorClass" class="swiper-item">
								<image :src="item.url" mode="aspectFill" />
							</view>
						</swiper-item>
					</swiper>
				</uni-swiper-dot>
			</view>
			<p class="name">{{goodInfo.name}}</p>
			<p class="desc">{{goodInfo.desc}}</p>
			<p class="wrap-price">
				<span class="price">{{goodInfo.specsInfo.rprice}}</span>
				<span class="price del" v-if="goodInfo.rebate<10">{{goodInfo.specsInfo.price}}</span>
			</p>
			<view class="wrap-labels">
				<view class="label"><uni-iconfont class="icon" color="#ccc" size="24" type="gwd" />{{goodInfo.specsInfo.name}}</view>
				<view class="label"><uni-iconfont class="icon" color="#ccc" size="24" type="shdd3" />{{goodInfo.origin_place}}</view>
			</view>
		</view>
		<view class="wrap-recommend" v-if="recommends.length>0">
			<view class="title"><span>推荐商品</span></view>
			<view class="uni-product-list">
				<view class="uni-product" v-for="(good,index) in recommends" :key="good" @click="clkRecommend(good)">
					<view class="image-view">
						<image class="uni-product-image" :src="good.avatar"></image>
					</view>
					<view class="uni-product-title">{{good.name}}</view>
					<view class="uni-product-price">
						<text class="price">{{good.specs[0].rprice}}</text>
						<text class="price del" v-if="good.rebate<10">{{good.specs[0].price}}</text>
					</view>
				</view>
			</view>
		</view>
		<view class="wrap-detail" v-if="goodInfo.detail.length>0">
			<view class="title"><span>商品详情</span></view>
			<image :lazy-load="true" mode="widthFix" v-for="(item,index) in goodInfo.detail" :key="index" :src="item"></image>
		</view>
		<view class="wrap-placehold"></view>
		<footer>
			<view class="wrap-btn-car" @click="clkCar">
				<uni-iconfont class="icon" size="24" type="car" />购物车
				<uni-badge class="badge" :text="selectResult.selectCount" type="error" />
			</view>
			<button @click="clkAddCar">加入购物车</button>
		</footer>
		
		<view class="wrap-gg" v-if="ggoption.show" @click="hideGg">
			<div class="wrap-main" :class="{show:ggoption.mainShow}" @click.stop>
				<view class="wrap-header">
					<image :src="goodInfo.avatar"></image>
					<p>
						<span class="price">{{ggrprice}}</span>
						<span class="price del" v-if="goodInfo.rebate<10">{{goodInfo.specs[specsIndex].price}}</span>
					</p>
					<p>库存{{goodInfo.specs[specsIndex].stock}}件</p>
					<p>已选：{{goodInfo.specs[specsIndex].name}}</p>
				</view>
				<view class="wrap-section">
					<span v-for="(sinfo,index) in goodInfo.specs" :key="sinfo.name" :class="{active:specsIndex===index}" @click="clkSpecs(index,sinfo)">{{sinfo.name}}</span>
				</view>
				<button class="wrap-footer" :disabled="goodInfo.specs[specsIndex].stock==0" @click="addCar">确定</button>
			</div>
		</view>
	</view>
</template>

<script>
	import {mapState} from 'vuex';
	import uniIconfont from '@/components/uni-iconfont/uni-icon.vue'
	import uniSwiperDot from '@/components/uni-swiper-dot/uni-swiper-dot.vue'
	import uniBadge from '@/components/uni-badge/uni-badge.vue'
	import {turnPage} from '@/common/global.js'
	import {ajaxGetGoodInfo, ajaxGetRecommendGoods} from '@/data/ajax.js'
	
	export default {
		components: {
			uniIconfont,
			uniSwiperDot,
			uniBadge
		},
		computed: {
			selectResult () {
				return this.$store.getters.doneSelectResult;
			},
			ggrprice () {
				return this.goodInfo.specs && this.goodInfo.specs.length > 0 && (this.goodInfo.rebate / 10 * this.goodInfo.specs[this.specsIndex].price).toFixed(2);
			}
		},
		data () {
			return {
				current: 0,
				bannerHeight: uni.getSystemInfoSync().screenWidth,
				goodInfo: {},
				covers: [
					// {colorClass: 'uni-bg-blue', url: 'https://img-cdn-qiniu.dcloud.net.cn/uniapp/images/cbd.jpg',　content: '内容 C'}
				],
				dotsStyles: {
					backgroundColor: '#fff',
					border: '0',
					color: '#fff',
					selectedBackgroundColor: '#fff',
					selectedBorder: '0'
				},
				recommends: [],
				ggoption: {
					show: false,
					mainShow: false
				},
				specsIndex: 0
			};
		},
		onShareAppMessage: function () {
			// 
		},
		onLoad (e) {
			// 获取商品详情
			let _this = this;
			ajaxGetGoodInfo(e, function (data) {
				let info = data.result;
				// banner
				_this.covers = [];
				info.cover.forEach( item => {
					_this.covers.push({url: item});
				});
				info.specs = info.specs.sort(function(a, b){
					if (parseFloat(a.price) > parseFloat(b.price) ) {
						return 1;
					}
					if (parseFloat(a.price) < parseFloat(b.price) ) {
						return -1;
					}
					return 0;
				})
				info.specsInfo = info.specs[0];
				_this.goodInfo = info;
				ajaxGetRecommendGoods('', function (data) {
					// 删除本商品
					// let arr = data.result;
					let arr = [];
					// 随机获取6个
					let sjarr = _this.rnd(0, data.result.length, 6)
					if (sjarr && sjarr.length > 0) {
						for (let j = 0;j < sjarr.length;j++) {
							arr.push(data.result[sjarr[j]])
						}
					}
					for (let i = 0;i < arr.length;i++) {
						if (arr[i]._id === e.id) {
							arr.splice(i, 1);
							break;
						}
					}
					_this.recommends = arr;
				});
			});
		},
		methods: {
			rnd (min, max, count) {
				if (max <= min || !max || max < 0) {
					return;
				}
				if (max <= count) {
					count = max - 1;
				}
				 var arr = [];
				 for (var i = 0;i < count;i++) {
				   var num = parseInt(Math.random() * (max - min) + min);
				   while (arr.indexOf(num) > -1) {
					num = parseInt(Math.random() * (max - min) + min);
				   }
				   arr.push(num);
				 }
				 return arr;	
			},
			change (e) {
				this.current = e.detail.current;
			},
			clkCar () {
				turnPage('car');
			},
			clkAddCar () {
				// 规格选择
				if (this.goodInfo.specs.length > 1) {
					this.showGg();
				} else {
					this.addCar();
				}
			},
			clkRecommend (data) {
				turnPage('detail', data);
			},
			clkSpecs (index, info) {
				console.log('===', index);
				// if (info.stock != 0) {
				this.specsIndex = index;	
				// }
			},
			addCar () {
				this.$set(this.goodInfo, 'select', true);
				this.hideGg();
				
				let _this = this;
				let _info = JSON.parse(JSON.stringify(this.goodInfo));
				
				_info.specsInfo = _info.specs[this.specsIndex];				
				console.log('==specsInfo==', _info.specsInfo);
				this.$store.commit('addGood', [_info, (data) => {
					// 添加商品到购物车后的回调
					_this.EVENTHUB.$emit('updateCount', data);
					uni.showToast({'title': '已添加到购物车'});
				}]);
			},
			showGg () {
				this.ggoption.show = true;
				this.$nextTick(function(){
					this.ggoption.mainShow = true;
				});
			},
			hideGg () {
				let _this = this;
				
				this.ggoption.mainShow = false;
				setTimeout(() => {
					_this.ggoption.show = false;
				}, 200);
			}
		}
	}
</script>

<style lang="scss">
	@import '@/common/global.scss';
	
	.content {
		background-color: #f4f5f6;
		
		> view {
			margin-bottom: 10px;
			height: unset;
			background-color: #fff;
			
			> .title {
				height: 40px;
				line-height: 40px;
				text-align: center;
				color: $theme;
				border-bottom: solid 1px $border-color;
				> span {
					display: inline-block;
					height: 38px;
					padding: 0 10px;
					border-bottom: solid 2px;
				}
			}
		}
		> view:nth-last-child(1),
		> view:nth-last-child(2),
		> view:nth-last-child(3) {
			margin: 0;
		}
		
		> .wrap-good {
			padding-bottom: 14px;
			text-align: center;
			
			> .wrap-swiper {
				margin-bottom: 5px;
				
				.swiper-box {
					height: 300px;
				}
				.swiper-item {
					display: flex;
					justify-content: center;
					align-items: center;
					height: 100%;
					background: #eee;
					color: #fff;
				}	
				.uni-swiper__dots-box {
					height: unset;
					text-align: right;
				}
				.swiper-item image {
					width: 100%;
					height: 100%;
				}
			}
			> .name {
				padding: 0 10px;
				font-size: 20px;
			}
			> .desc {
				margin-top: 5px;
				margin-bottom: 5px;
				padding: 0 10px;
				font-size: 14px;
			}
			> .wrap-price {
				> .price:not(.del) {
					font-size: 24px;
				}
			}
			> .wrap-labels {
				display: flex;
				margin-top: 14px;
				height: 60px;
				color: #ccc;
				
				> .label {
					flex: 1;
					font-size: 12px;
					> .icon {
						display: block;
						margin-top: 6px;
					}
				}
			}
		}
		
		> .wrap-recommend {
			/* product */
			.uni-product-list {
			    display: flex;
			    width: 100%;
			    flex-wrap: wrap;
			    flex-direction: row;
			}
			
			.uni-product {
			    padding: 20upx;
			    display: flex;
			    flex-direction: column;
			}
			
			.image-view {
			    height: 330upx;
			    width: 330upx;
				margin:12upx 0;
			}
			
			.uni-product-image {
			    height: 330upx;
			    width: 330upx;
			}
			
			.uni-product-title {
			    width: 300upx;
			    word-break: break-all;
			    display: -webkit-box;
			    overflow: hidden;
				line-height:1.5;
			    text-overflow: ellipsis;
			    -webkit-box-orient: vertical;
			    -webkit-line-clamp: 2;
			}
			
			.uni-product-price {
				margin-top:10upx;
			    font-size: 28upx;
				line-height:1.5;
			    position: relative;
			}
			
			.price {
				font-size: 14px;
			}
			
			.price.del {
			    font-size: 12px;
			}
			
			.uni-product-tip {
			    position: absolute;
			    right: 10upx;
			    padding: 0 10upx;
			    border-radius: 5upx;
			}
		}
		
		> .wrap-detail {
			> image {
				display: block;
				width: 100%;
			}
		}
		
		// 占位区
		> .wrap-placehold {
			height: 40px;
		}
		
		> footer {
			position: fixed;
			margin: 0;
			bottom: 0;
			left: 0;
			width: 100%;
			height: 50px;
			box-shadow: 0 0 5px $border-color;
			background-color: #fff;
			
			> .wrap-btn-car {
				position: relative;
				float: left;
				margin: 0;
				width: 40%;
				height: 98%;
				text-align: center;
				font-size: 12px;
				> .icon {
					display: block;
					margin-top: 5px;
				}
				> .badge {
					position: absolute;
					top: 3px;
					right: 0;
					left: 26px;
					margin: auto;
				}
			}
			> button {
				float: right;
				width: 60%;
				height: 100%;
				line-height: 50px;
				color: #fff;
				background-color: $theme;
			}
		}
		
		// 规格选择
		> .wrap-gg {
			position: fixed;
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;
			background-color: rgba(0, 0, 0, .7);
			z-index: 2;
			
			> .wrap-main {
				position: absolute;
				bottom: -50%;
				left: 0;
				width: 100%;
				height: 50%;
				transition: bottom .4s;
				background-color: #fff;
				z-index: 2;
				
				> .wrap-header {
					height: 80px;
					border-bottom: solid 1px $border-color;
					
					> image {
						position: absolute;
						top: -10px;
						left: 10px;
						width: 80px;
						height: 80px;
						border-radius: 5px;
						border: solid 1px #ccc;
					}
					
					> p {
						padding-left: 100px;
					}
					> p:first-of-type {
						margin-top: 5px;
					}
				}
				> .wrap-section {
					padding: 10px;
					height: calc(100% - 80px - 50px - 20px);
					overflow: auto;
					> p {
						font-size: 18px;
					}
					> span {
						display: inline-block;
						margin-top: 10px;
						margin-right: 10px;
						padding: 4px 6px;
						border-radius: 4px;
						border: solid 1px transparent;
						background-color: #efefef;
					}
					> span.active {
						color: $theme;
						border: solid 1px $theme;
					}
					> span.disabled {
						opacity: 0.5;
					}
				}
				> .wrap-footer:not([disabled]) {
					height: 50px;
					line-height: 50px;
					color: #fff;
					background-color: $theme;
				}
			}
			
			> .wrap-main.show {
				bottom: 0;
			}
		}
	}
</style>
