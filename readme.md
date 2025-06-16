## 组件说明
适用项目：uniapp
用途：签到页面功能模块
用于展示uniapp的youlan-SignIn签到插件，此项目是youlan-SignIn插件的演示demo

## 引入组件
### 引入nvue 在首页引入，看你的需求选择引入的文件
`import youlanSignIn from '@/components/youlan-SignIn/youlan-SignIn.nvue'`
### 引入vue
`import youlanSignIn from '@/components/youlan-SignIn/youlan-SignIn.vue'`
### 注册组件
```js
components: {
    youlanSignIn
},
// 调用组件
<youlanSignIn
    :integral="integral"
    :isIntegral="isIntegral" 
    :checkinDays="checkinDays" 
    :already="data" 
    type="sign" 
    lang="zh" 
    @change="signDate" 
/>

// already 参数使用方法
data() {
    return {
        data: [],
        checkinDays: 0,     // 连续签到天数
        integral: 0,        // 本次签到获取的积分
        isIntegral: false,  // 是否显示签到积分模块
    }
}

onLoad() {
    setTimeout(() => {
        let arr = ["2024-01-01", "2024-01-02"];     // 假设这是从后台获取的签到数据
        arr.forEach(el => {
            // 需要换成时间戳
            this.data.push(new Date(el).getTime())
        })
    }, 2000)
},
methods: {
    signDate(v) {
        // 签到成功后需要更新连续签到的数据
        this.checkinDays = this.checkinDays + 1;
        // 获取随机整数
        const randomInteger = Math.floor(Math.random() * (10 - 1 + 1)) + 1;
        // 模拟获取的积分,这个根据你的实际情况来，如果你没有积分系统，可以忽略这些
        this.integral = randomInteger;
        this.isIntegral = true;

        console.log(v);
    }
}
```
## 属性说明

|名称|类型|是否必填|默认值|可选值|说明|
|----|----|----|----|----|----|
|lang|String|否|zh|zh/en|中文/英文|
|type|String|否|calendar|calendar/sign|日历/签到|
|checkDate|Boolean|否|false|true/false|快捷选择年月|
|bgweek|String|否|#FF8F22|rgb/rgba/十六进制|星期选中颜色|
|bgday|String|否|#FF8F22|rgb/rgba/十六进制|当天选中颜色|
|supplementary|Boolean|否|true|true/false|点击日期是否可以补签|
|already|Array|否|[]|[]|显示已经签到的日期|
|checkinDays|Number/String|否|0|连续签到的天数|
|integral|Number/String|否|0	|显示本次签到获取的积分|
|isIntegral|Boolean|否|false|true/false|是否显示积分模块|
|signin_but_bg|String|否|#909399|rgb/rgba/十六进制|不能签到时签到按钮的颜色|

## 方法

|事件名称|说明|回调参数|
|----|----|----|
|change|用户点击签到时触发|返回签到日期|
|shift|点击右上角补签按钮触发|无|
|setAlready|设置签到的日期，和already属性功能一样|无|


## 演示案例
```js
<template>
	<view>
		<youlanSignIn @shift="shift" :integral="integral" :isIntegral="isIntegral" :checkinDays="checkinDays" :already="data"
			:supplementary="true" type="sign" lang="zh" @changeMonth="historysign" @change="signDate" />

	</view>
</template>

<script>
	import youlanSignIn from '@/components/youlan-SignIn/youlan-SignIn.vue'
	export default {
		components: {
			youlanSignIn
		},
		mounted() {
			this.historysign()
			// 模拟设置签到日期
			setTimeout(() => {
				this.setAlready();
			}, 2000)
		},
		data() {
			return {
				data: [],
				checkinDays: 0, // 连续签到天数
				integral: 0, // 本次签到获取的积分
				isIntegral: false, // 是否显示签到积分模块	
			};
		},
		methods: {
			shift() {
				console.log('用户点击了补签按钮');
			},
			setAlready() {
				// 通过方法设置已经签到的日期
				const res = {
					code: 200,
					data: {
						"2025-06-01": "0",
						"2025-06-02": "1",
						"2025-06-03": "1",
						"2025-06-04": "0",
						"2025-06-05": "0",
					}
				}
				const days = []
				if (res.code == 200) {
					for (const i in res.data) {
						if (res.data[i] == "1") {
							console.log(i);
							days.push(new Date(i).getTime())
						}
					}
				
					console.log(days);
				} else this.$refs.uToast.show({
					title: '获取签到信息失败',
					type: 'error',
				})
				// console.log('days' + days)
				this.$refs.youlanSignInRef.setAlready(days);
			},
			async historysign() {
				this.data = []
				const res = {
					code: 200,
					data: {
						"2025-05-01": "0",
						"2025-05-02": "1",
						"2025-05-03": "1",
						"2025-05-04": "0",
						"2025-05-05": "0",
						"2025-05-06": "0",
						"2025-05-07": "0",
						"2025-05-08": "0",
						"2025-05-09": "0",
						"2025-05-10": "0",
						"2025-05-31": "1",
					}
				}

				if (res.code == 200) {
					for (const i in res.data) {
						if (res.data[i] == "1") {
							console.log(i);
							this.data.push(new Date(i).getTime())
						}
					}

					console.log(this.data);
				} else this.$refs.uToast.show({
					title: '获取签到信息失败',
					type: 'error',
				})
			},
			async signDate(v) {
				console.log(v);
				this.checkinDays = this.checkinDays + 1;
			},


		}
	}
</script>
```