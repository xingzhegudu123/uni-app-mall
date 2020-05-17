<template>
	<view>
		<view class="block"></view>
		<view class="QR">
			<image src="../../../static/img/qrcode.jpg"></image>
		</view>
		<view class="title">
			米斯特吴微信, 技术交流, 问题答疑, 感情沟通.
		</view>
		<view class="btn" v-show="showBtn" @tap="handleSavePhotos">
			{{savePhotos}}
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				savePhotos: "保存到相册",
				showBtn: false
			}
		},
		onLoad() {
			// #ifdef APP-PLUS
			this.showBtn = true;
			// #endif
			
			this.showBtn = true;
		},
		methods:{
			handleSavePhotos(){
				// 调用系统方法实现图片保存
				this.savePhotos = "正在保存";
				let ws = this.$mp.page.$getAppWebview();
				let bitmap = new plus.nativeObj.Bitmap();
				
				this.$nextTick(() => {
					setTimeout(() => {
						// 画图
						ws.draw(bitmap,(success) => {
							bitmap.save("_doc/Qr.jpg",{
								overwrite:true,
								quality:100
							},(success) => {
								plus.gallery.save(success.target,(e) => {
									uni.showToast({
										title:"保存成功"
									})
									this.savePhotos = "保存到相册"
									bitmap.clear(); // 销毁对象
								})
							})
							
						},(err) => {
							console.log("保存图片失败")
						}, (option) => {
							console.log("bitmap绘制图片失败")
						})
					},200)
				})
			
			}
		}
	}
</script>

<style lang="scss">
	page {
		background-color: #f06c7a;
	}

	.block {
		width: 100%;
		height: 30vh;
		background-color: #fff;
		display: flex;
		justify-content: center;
	}

	.QR {
		width: 60vw;
		height: 80vw;
		margin: -40vw auto 0 auto;
		background-color: #fff;
		display: flex;
		justify-content: center;
		align-items: center;

		image {
			width: 50vw;
			height: 50vw;
		}
	}

	.title {
		width: 100%;
		margin-top: 50upx;
		display: flex;
		justify-content: center;
		font-size: 36upx;
		color: #fff;
	}

	.btn {
		width: 50%;
		height: 80upx;
		border-radius: 80upx;
		display: flex;
		justify-content: center;
		align-items: center;
		padding: 0 20upx;
		margin: 0 auto;
		margin-top: 50upx;
		background-color: rgba(255, 255, 255, .8);
	}
</style>
