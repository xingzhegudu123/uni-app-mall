<template>
	<view class="popup spec" v-show="spaceInfo.showSpace" @tap="hideSpecifications">
		<!-- 遮罩层 -->
		<view class="mask"></view>
		<view class="layer">
			<view class="content">
				<view class="title">选择规格:</view>
				<view class="sp">
					<view @tap.stop="handleSelectSpecification(item)" :class="{'on':item == goodsInfo.spec}" v-for="(item,index) in goodsData.spec" :key="index">
						{{item}}
					</view>
				</view>
				<view class="length">
					<view class="text">数量</view>
					<!-- counter组件 -->
					<counter @sub="sub" @add="add" :goodsInfo="goodsInfo" />
				</view>
			</view>
			<!-- 确定按钮 -->
			<view class="btn">
				<view @tap="hideSpecifications" class="button">确定</view>
			</view>
		</view>
	</view>
</template>
<!-- 模态框 -->
<script>
	import counter from './counter.vue'
	export default{
		components:{
			counter
		},
		props:{
			spaceInfo: Object,
			goodsInfo: Object,
			goodsData: Object
			
		},
		methods:{
			// 弹出模态框
			hideSpecifications() {
				this.$emit("hideSpec");
				
			},
			// 选择规格
			handleSelectSpecification(item) {
				// 子级注册事件 父级执行事件(注意子级不能直接改父级内容 app端有问题)
				this.$emit("setSelectSpec",item);
				// console.log(this.goodsInfo.spec,item);
			},
			//  数量加减
			add(){
				this.$emit("add");
			},
			sub(){
				this.$emit("sub");
			},
		}
	}
</script>

<style lang="scss">
	.popup {
		position: fixed;
		top: 0;
		width: 100%;
		height: 100%;
		z-index: 20;

		.mask {
			position: fixed;
			top: 0;
			width: 100%;
			height: 100%;
			z-index: 21;
			background-color: rgba(0, 0, 0, 0.6);
		}

		.layer {
			position: fixed;
			z-index: 22;
			bottom: 0;
			width: 92%;
			padding: 0 4%;
			height: 70%;
			border-radius: 20upx 20upx 0 0;
			background-color: #fff;
			display: flex;
			flex-wrap: wrap;
			align-content: space-between;

			.content {
				width: 100%;
				padding: 20upx 0;
			}

			.btn {
				width: 100%;
				height: 100upx;

				.button {
					width: 100%;
					height: 80upx;
					border-radius: 40upx;
					color: #fff;
					display: flex;
					align-items: center;
					justify-content: center;
					background-color: #f47952;
					font-size: 28upx;
				}
			}
		}

		&.spec {
			.title {
				font-size: 30upx;
				margin: 30upx 0;
			}

			.sp {
				display: flex;

				view {
					font-size: 28upx;
					padding: 5upx 20upx;
					border-radius: 8upx;
					margin: 0 30upx 20upx 0;
					background-color: #f6f6f6;

					&.on {
						padding: 3upx 18upx;
						border: solid 1upx #f47925;
					}
				}
			}

			.length {
				margin-top: 30upx;
				border-top: solid 1upx #aaa;
				display: flex;
				justify-content: space-between;
				align-items: center;
				padding-top: 20upx;

				.text {
					font-size: 30upx;
				}

			}

		}
	}
</style>
