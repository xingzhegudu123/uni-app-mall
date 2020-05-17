<template>
	<view>
		<!-- 购物列表 -->
		<view class="goods-list">
			<view class="empty" v-if="goodsList.length ==0">购物车空空如也~</view>
			<view class="row" v-for="(item,index) in goodsList" :key="index">
				<!-- 删除按钮 -->
				<view class="menu" @tap="handleSingleDelete(item)">
					<view class="icon iconfont">&#xe6a6;</view>
				</view>
				<!-- 商品 -->
				<view class="production" :class="[theIndex == index ? 'open' : oldIndex == index ? 'close' : '']"
				@touchstart="handleTouchStart(index,$event)" @touchmove="handleTouchMove(index,$event)"
				@touchend="handleTouchEnd(index,$event)">
					<!-- 左边单选checkbox -->
					<view class="container" @tap="handleCheckbox(item)">
						<view class="checkbox">
							<view :class="{'on':item.selected}"></view>
						</view>
					</view>
					
					<!-- 右边商品详情 -->
					<view class="goods-info" @tap="handleGoodsInfo(item)">
						<view class="img">
							<image :src="item.img"></image>
						</view>
						<view class="info">
							<view class="title">{{item.name}}</view>
							<view class="spec">{{item.spec}}</view>
							<view class="price-number">
								<view class="price">￥{{item.price}}</view>
								<counter :goodsInfo="item" @add="add(item)" @sub="sub(item)" />
							</view>
						</view>
					</view>
				</view>
			</view>
		</view>
		<!-- 底部菜单 -->
		<view class="footer" :style="{bottom: footerbottom}">
			<!-- checkbox -->
			<view class="container" @tap="handleSelectedAll">
				<view class="checkbox">
					<view :class="{'on': isAllSelected}"></view>
				</view>
				<view class="text">全选</view>
			</view>
			<view class="delBtn" @tap="handleMulDelete" v-if="selectedList.length > 0">删除</view>
			<view class="settlement">
				<view class="sum">
					合计:  <view class="money">￥{{sumPrice}}</view>  
				 </view>
				<view class="btn" @tap="handleConfirm">结算{{selectedList.length}}</view>
			</view>
		</view>
	</view>
</template>
<!-- 功能 左滑删除: 下面滑上面回去 -->
<script>
	import counter  from '../../../components/counter.vue'
	export default{
		components: {
			counter
		},
		onLoad() {
			// 兼容h5下的底部菜单
			// #ifdef H5
			this.footerbottom = document.getElementsByTagName("uni-tabbar")[0].offsetHeight + "px";
			// #endif
		},
		// 页面显示时
		onShow(){
			uni.getStorage({
				key:"goodsList",
				success:(res=>{
					console.log(res.data);
					// 将所有商品选中状态都设置为false;
					for(let i = 0; i<res.data.length; i++){
						res.data[i].selected = false; // 每个对象加了一个属性selected
					}
					this.goodsList = res.data;
					
					// 属性数据的初始化
					this.selectedList = [];
					this.isAllSelected = false;
					this.sumPrice = '0.00';
					
				})
			})
		},
		data(){
			return{
				goodsList:[],
				theIndex: null, // 控制滑动效果 当前滑动下标
				oldIndex: null, // 上一个左滑下标
				footerbottom: 0, // 距离底部==tabBar高度
				selectedList:[], // 选中数组
				isAllSelected: false, //是否全选
				sumPrice: '0.00' // 购物车总价
			}
		},
		methods:{
			add(item){
				item.number++;
				// 更新storage
				uni.getStorage({
					key:"goodsList",
					success: (res) => {
						uni.setStorageSync("goodsList",this.goodsList);
					}
				})
				this.sum();
		   },
		   sub(item){
				if(item.number <= 1){
					return;
				}
				item.number--;
				// 更新storage
				uni.getStorage({
					key:"goodsList",
					success: (res) => {
						uni.setStorageSync("goodsList",this.goodsList);
					}
				})
				this.sum();
			},
			// 跳转商品详情页
			handleGoodsInfo(item) {
				uni.navigateTo({
					url: "../../goods/goods?goodsInfo=" + JSON.stringify(item)
				})
			},
			
			handleTouchStart(index, event) {
				console.log(event);
				// 多点触控 不触发
				if (event.touches.length > 1) {
					return;
				}
				
				this.oldIndex = this.theIndex;  // 滑下一个时上一个滑回去  (点下一个记录上一个下标)
				this.theIndex = null; // 可右滑 (走三目运算符后面的)

				// 初始化坐标
				this.initXY = [event.touches[0].pageX, event.touches[0].pageY];
			},
			handleTouchMove(index, event) {
				// 多点触控 不触发
				if (event.touches.length > 1) {
					return;
				}

				// 移动位置
				let moveX = event.touches[0].pageX - this.initXY[0];
				let moveY = event.touches[0].pageY - this.initXY[1];
				
				// 滑动位置小 不触发
				if (Math.abs(moveX) < 5) {
					return;
				}

				// 竖向滑动 不触发
				if (Math.abs(moveY) > Math.abs(moveX)) {
					return;
				}

				// 左滑
				if (moveX < 0) {
					this.theIndex = index;
				}
			},
			handleTouchEnd(index, event) {
				
			},
			// 单选
			handleCheckbox(item){
				item.selected = !item.selected;
				
				// 条件: 数组中是否包含该元素：有减无加(有说明已添加，再选为取消则删除--没有就添加)
				let isExist = this.selectedList.indexOf(item); // 返回下标
				if(isExist > -1){
					this.selectedList.splice(isExist,1);
				}else {
					// push 数组里
					this.selectedList.push(item);
				}
				// 全选状态
				if(this.selectedList.length == this.goodsList.length){
					this.isAllSelected = true;
				}else {
					this.isAllSelected = false;
				}
				
				this.sum();
				
			},
			// 全选
			handleSelectedAll(){
				this.isAllSelected = !this.isAllSelected;
				// 数据处理
				let arr = [];
				this.goodsList.forEach((item,i) => {
					item.selected = this.isAllSelected;
					arr.push(item);
				})
				
			    this.selectedList =	this.isAllSelected ? arr : [];
				
				// 调用合计
				this.sum();
			},
			// 单删(滑动)
			handleSingleDelete(item){
				// 更新storage
				uni.getStorage({
					key:"goodsList",
					success: (res => {
						let goodsList = res.data
						// 本地删除选中商品
						goodsList.forEach((goods, index)=>{
							if(goods.goods_id == item.goods_id){ // 本地中
									goodsList.splice(index, 1)
							}
						})
						uni.setStorageSync("goodsList",goodsList);
					}) 
				})
				// 更新数组
				this.goodsList.splice(this.goodsList.indexOf(item),1);
				this.selectedList.splice(this.selectedList.indexOf(item),1);
				
				
				// 全部删除取消全选状态
				this.isAllSelected = this.goodsList.length>0? this.isAllSelected : false;
				this.sum();
				this.oldIndex = null;
				this.theIndex = null;
			},
			// 删除 (选中数组>0出现)
			handleMulDelete(){
				// 循环删除所有选中的商品
				while(this.selectedList.length > 0){
					this.handleSingleDelete(this.selectedList[0]); // 只要长度>0 首个肯定有值  每次循环删item在两数组中下标也会变
				}
				
				// 初始化数据
				this.selectedList = [];
				this.isAllSelected = false;
				this.sum();
			},
			// 合计总价
			sum(){
				this.sumPrice = 0;
				this.goodsList.forEach((item,i) => {
					if(item.selected){
						this.sumPrice = this.sumPrice + (item.number * item.price)
					}
				})
				this.sumPrice  = this.sumPrice.toFixed(2);
			},
			// 结算
			handleConfirm(){ // 结算
				if(this.selectedList.length < 1){
					uni.showToast({
						title:"请选择结算的商品",
						icon:"none"
					})
					return;
				}
				
				// 本地存储
				uni.setStorage({
					key:"confirmList",
					data: this.selectedList,
					success: () => {
						uni.navigateTo({
							url:"../../order/confirm"
						})
					}
				})
			}
			
		},
		
	}
</script>

<style lang="scss">
	.container {
		display: flex;
		align-items: center;

		.checkbox {
			width: 35upx;
			height: 35upx;
			border-radius: 100%;
			border: solid 2upx #f06c7a;
			display: flex;
			justify-content: center;
			align-items: center;

			.on {
				width: 25upx;
				height: 25upx;
				border-radius: 100%;
				background-color: #f06c7a;
			}
		}

		.text {
			font-size: 28upx;
			margin-left: 10upx;
		}
	}


	.goods-list {
		width: 100%;
		padding: 20upx 0 120upx 0;

		.empty {
			width: 100%;
			height: 60upx;
			display: flex;
			justify-content: center;
			align-items: center;
			font-size: 32upx;
			color: #a7a7a7;
		}

		.row {
			width: calc(92%);
			height: calc(22vw + 40upx);
			margin: 20upx auto;

			border-radius: 15upx;
			box-shadow: 0upx 5upx 20upx rgba(0, 0, 0, 0.1);
			display: flex;
			align-items: center;
			position: relative;
			overflow: hidden;
			z-index: 4;
			border: 0;
            // 滑动删除按钮
			.menu {
				.icon {
					color: #fff;
					font-size: 60upx;
				}

				position: absolute;
				width: 30%;
				height: 100%;
				right: 0;
				display: flex;
				justify-content: center;
				align-items: center;
				background-color: red;
				color: #fff;
				z-index: 2;
			}
              // 商品item滑动删除动画
			.production {
				@keyframes showMenu {
					0% {
						transform: translateX(0);
					}

					100% {
						transform: translateX(-30%);
					}
				}

				@keyframes closeMenu {
					0% {
						transform: translateX(-30%);
					}

					100% {
						transform: translateX(0);
					}
				}
                    // 左滑
				&.open {
					animation: showMenu 0.25s linear both;
				}
                   // 右滑
				&.close {
					animation: closeMenu 0.15s linear both;
				}

				background-color: #fff;

				.container {
					padding-left: 20upx;
					flex-shrink: 0;
					height: 22vw;
					margin-right: 20upx;
				}

				position: absolute;
				width: 100%;
				padding: 0 0;
				height: 100%;
				z-index: 3;
				display: flex;
				align-items: center;

				.goods-info {
					width: 100%;
					display: flex;
					padding-right: 20upx;

					.img {
						width: 22vw;
						height: 22vw;
						border-radius: 10upx;
						overflow: hidden;
						flex-shrink: 0;
						margin-right: 10upx;

						image {
							width: 22vw;
							height: 22vw;
						}
					}

					.info {
						width: 100%;
						height: 22vw;
						overflow: hidden;
						display: flex;
						flex-wrap: wrap;
						position: relative;

						.title {
							width: 100%;
							font-size: 28upx;
							display: -webkit-box;
							-webkit-box-orient: vertical;
							-webkit-line-clamp: 2;
							// text-align: justify;
							overflow: hidden;
						}

						.spec {
							font-size: 20upx;
							background-color: #f3f3f3;
							color: #a7a7a7;
							height: 30upx;
							display: flex;
							align-items: center;
							padding: 0 10upx;
							border-radius: 15upx;
							margin-bottom: 20vw;
						}

						.price-number {
							position: absolute;
							width: 100%;
							bottom: 0upx;
							display: flex;
							justify-content: space-between;
							align-items: flex-end;
							font-size: 28upx;
							height: 60upx;

							.price {}

						}
					}
				}
			}
		}
	}

	.footer {
		width: 92%;
		padding: 0 4%;
		background-color: #fbfbfb;
		height: 100upx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		font-size: 28upx;
		position: fixed;
		bottom: 0upx;
		z-index: 5;

		.delBtn {
			border: solid 1upx #f06c7a;
			color: #f06c7a;
			padding: 0 30upx;
			height: 50upx;
			border-radius: 30upx;
			display: flex;
			justify-content: center;
			align-items: center;
		}

		.settlement {
			width: 60%;
			display: flex;
			justify-content: flex-end;
			align-items: center;

			.sum {
				width: 50%;
				font-size: 28upx;
				margin-right: 10upx;
				display: flex;
				justify-content: flex-end;

				.money {
					font-weight: 600;
				}
			}

			.btn {
				padding: 0 30upx;
				height: 50upx;
				background-color: #f06c7a;
				color: #fff;
				display: flex;
				justify-content: center;
				align-items: center;

				border-radius: 30upx;
			}
		}
	}
</style>

