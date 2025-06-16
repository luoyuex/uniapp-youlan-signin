<template>
	<view>
		<youlanSignIn ref="youlanSignInRef" @shift="shift" :integral="integral" :isIntegral="isIntegral" :checkinDays="checkinDays" :already="data"
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
					// days [1748822400000, 1748908800000]
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

<style lang="scss">
page {
	background-color: #f8f8f8;
	padding: 8px 12px;
	box-sizing: border-box;
}
</style>