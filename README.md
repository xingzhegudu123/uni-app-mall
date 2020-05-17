本项目为uniapp开发的商城项目。主要功能有：uniapp基本结构，自定义导航栏，商品分类，商品列表瀑布流，封装计数组件和模态框，上拉加载下拉刷新，详情页, 购物车,滑动删除,商品评论，订单结算，图片保存等。
## get 技能
- 新建uniapp项目和各端配置--json.page设置tabBar和页面切换和跳转
- 导航栏兼容：详情页不需要导航栏？ 隐藏导航栏：page.json中h5设置titleNview:false，移动端设置app-plus(移动端会出现高度问题:需加上全局状态栏)
             小程序自带返回按钮？：onload里写条件编译MP隐藏返回按钮
- 组件封装：选择规格模态框--数量增减组件
- 分类筛选+瀑布流: 商品列表页
- 本地存储: 加入购物车和立即购买uni.setStorageSync
- 购物车: 
  - 单选: 定义一个选中数组:indexOf判断选中数组中是否包含该元素：有减无加--如选中数组长度=原数组则全选√反之不√  
  - 全选: 遍历原数组每个item.selected取反  
  - 单删(滑动): 根据indexOf(item)找到在在原数组中位置splice删除--根据id找到本地localStorage位置删除  
  - 勾选删除:  while循环-选中数组的商品-只要长度大于0-就调用单删函数传递数组首个  
  - 总价：遍历原数组后:this.sumPrice = this.sumPrice + (item.number * item.price)  
  - 滑动效果: 下一个滑动上一个回去: 每次滑时记录上一个下标, 运用三目运算符：当前下标!=上一个时 上一个回去--当前右滑:  当前下标置空null  
- 保存图片: 调用系统api保存图片:this.$mp.page.$getAppWebview()绘制，new plus.nativeObj.Bitmap()保存
- 遇到问题： 注意子级不要直接改父级内容 app端点击无反应--应该子传父-在父级改
