<template>
	<view class="content" v-if="orderInfo._id">
		<cust-service v-model="showCts"></cust-service>
		<view class="header">
			<text class="status">{{orderInfo.status===0?'订单已取消':orderInfo.status===1?'等待卖家接单':(orderInfo.status===2||orderInfo.status===3||orderInfo.status===4)?'备货中':(orderInfo.status===5||orderInfo.status===6)?'配送中':'订单已完成'}}</text>
			<text class="sub" v-if="orderorderInfo.status===6">感谢你您的信任，期待再次光临</text>
			<button @click="clkJxgm">继续购买</button>
			<button @click="clkKf">客服</button>
		</view>
		<view class="list">
			<view class="item" style="padding-top:0;padding-bottom:0;">
				商品<view class="right">{{orderInfo.count}}件</view>
			</view>
			<!-- <view class="item" v-for="(item,index) in orderInfo.order_product" :key="item">
				<view class="left">
					<image class="center-hv" :src="item.avatar"></image>
				</view>
				<view class="middle">
					<p class="name text-over">{{item.name}}-{{item.specs_name}}</p>
					<p class="jg">
						<text class="price">{{item.rprice}}</text>
						<text class="price del" v-if="item.rebate<10">{{item.price}}</text>
						<text>x {{item.count}}</text>
					</p>
				</view>
				<view class="right" v-if="item.weight">{{item.weight}}g</view>
			</view> -->
			<view class="item" v-for="(item,index) in orderInfo.order_product" :key="item" style="display:flex;align-items:center;line-height:16px;font-size:12px;">
				<image style="width:40px;height:40px;" :src="item.avatar"></image>
				<view style="width:calc(100% - 40px - 60px - 50px);padding:0 5px;">
					<view class="text-over">{{item.name}}</view>
					<view class="text-over">{{item.specs_name}}</view>
				</view>
				<!-- <text style="flex:1;padding:0 5px;"><br>{{item.specs_name}}</text> -->
				<view style="width:60px;text-align:center;">
					<p style="display:flex;align-items:center;">{{item.price}} <span class="unit" style="width:40px;color:#999;font-size:10px;">元/斤 元/盒</span></p>
					<p style="display:flex;align-items:center;">{{item.weight}} <span class="unit" style="width:40px;color:#999;font-size:10px;">斤/盒</span></p>
				</view>
				<view style="width:50px;text-align:right;">{{item.money}}</view>
			</view>
		</view>
		<view class="list total">
			<view class="item">总价<view class="right price">{{orderInfo.money}}</view></view>
			<!-- <view class="item" v-if="orderInfo.expMoney">配送费<view class="right price">{{orderInfo.expMoney}}</view></view>
			<view class="item" v-if="orderInfo.expMoney">合计<view class="right price">{{orderInfo.money + orderInfo.expMoney}}</view></view> -->
		</view>
		<view class="list type-1">
			<view class="item">
				<view class="left">收货人：</view>
				<view class="middle">{{orderInfo.order_consignees.name}} {{parseMobile(orderInfo.order_consignees?orderInfo.order_consignees.mobile:'')}}</view>
			</view>
			<view class="item">
				<view class="left">收货地址：</view>
				<view class="middle">{{orderInfo.order_consignees.address}} {{orderInfo.order_consignees.door_address}}</view>
			</view>
		</view>
		<view class="list type-1">
			<view class="item">
				<view class="left">订单号：</view>
				<view class="middle">{{orderInfo._id}} <button class="copy" @click="clkCopy">复制</button> </view>
			</view>
			<view class="item">
				<view class="left">下单时间：</view>
				<view class="middle">{{formateDate(orderInfo.time)}}</view>
			</view>
		</view>
	</view>
</template>

<script>
	import custService from '@/components/cust-service/cust-service.vue';
	import {turnPage} from '@/common/global.js';
	import {dataFormat, encodeMobile} from '@/common/tool.js';
	import {ajaxGetOrderInfo} from '@/data/ajax.js';
	
	export default {
		components: {
			custService
		},
		data() {
			return {
				// 显示、隐藏客服
				showCts: false,
				orderInfo: {}
			};
		},
		computed: {},
		onLoad(e) {
			let _this = this;
			this.id = e.id;
			ajaxGetOrderInfo(e, (data) => {
				_this.orderInfo = data.result;
			});
		},
		methods: {
			clkCopy () {
				uni.setClipboardData({
					data: this.orderInfo._id,
					success: function() {
						uni.showToast({'title': '订单号复制成功'});
					}
				})
			},
			clkJxgm () {
				turnPage('home', '', true);
			},
			clkKf () {
				this.showCts = true;
			},
			parseMobile (data) {
				return data && encodeMobile(data);
			},
			formateDate (date) {
				return dataFormat(new Date(date), 'yyyy-MM-dd hh:mm');
			}
		}
	};
</script>

<style lang="scss">
	@import '@/common/global.scss';
	
	.content {
		height: 100%;
		overflow: auto;
		background-color: #f4f5f6;
		
		.price {
			font-size: inherit;
			color: inherit;
		}
		
		> view {
			margin-bottom: 10px;
			background-color: #fff;
		}
		
		> .header {
			padding: 10px 0;
			line-height: 32px;
			text-align: center;
			
			> .status {
				display: block;
				font-size: 18px;
			}
			> .sub {
				display: block;
			}
			> button {
				display: inline-block;
				margin-top: 10px;
				margin-left: 10px;
				width: 100px;
				height: 30px;
				line-height: 28px;
				font-size: inherit;
				color: #999;
				border: solid 1px $border-color;
				border-radius: 2px;
				background-color: transparent;
			}
		}
		
		> .list {
			> .item {
				padding: 10px;
				height: 50px;
				line-height: 50px;
				overflow: hidden;
				border-bottom: solid 1px $border-color;
				
				> .left {
					position: relative;
					float: left;
					width: 50px;
					height: 100%;
					
					> image {
						right: inherit;
						width: 80%;
						height: 80%;
					}
					
				}
				> .middle {
					float: left;
					width: calc(100% - 50px - 50px);
					line-height: 25px;
					
					.del {
						color: #999;
					}
				}
				> .right {
					float: right;
					width: 50px;
					height: 100%;
					text-align: right;
				}
			}
		}
		
		> .list.total > .item {
			padding-top: 5px;
			padding-bottom: 5px;
			
			> .right {
				width: 120px;
			}
		}
		
		> .list.type-1 {
			> .item {
				height: unset;
				line-height: unset;
				border: 0;
				
				> .left {
					width: 80px;
					height: unset;
					color: #999;
				}
				
				> .middle {
					width: calc(100% - 80px);
					line-height: unset;
				}
				
				.copy {
					display: inline-block;
					margin-left: 10px;
					padding: 0;
					width: 40px;
					height: 24px;
					line-height: 20px;
					color: $theme;
					background-color: transparent;
					border: solid 1px $theme;
					font-size: 12px;
					vertical-align: middle;
				}
			}
		}
		
		// 客服
		> .wrap-kf {
			position: fixed;
			top: 0;
			left: 0;
			width: 100%;
			height: 100%;
			line-height: 22px;
			background-color: rgba(255, 255, 255, 1);
			z-index: 2;
			
			> .main {
				width: 240px;
				height: 374px;
				
				> image {
					margin-bottom: 10px;
					width: 100%;
				}
				> text {
					display: block;
				}
			}
		}
		.unit {
			position: relative;
			padding: 0 5px;
		}
		.unit:before {
			content: '(';
			width: 5px;
			height: 16px;
			color: inherit;
			position: absolute;
			left: 2px;
			top: 0;
			bottom: 0;
			margin: auto;
			font-size: 12px;
		}
		.unit:after {
			content: ')';
			width: 5px;
			height: 16px;
			color: inherit;
			position: absolute;
			right: 2px;
			top: 0;
			bottom: 0;
			margin: auto;
			font-size: 12px;
		}
	}
</style>
