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
		onLoad() {
			this.historysigncount()
			this.gettask()
		},
		mounted() {
			this.historysign()
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
			async historysign() {
				this.data = []
				const res = {
					code: 200,
					data: {
						"2024-12-01": "0",
						"2024-12-02": "1",
						"2024-12-03": "1",
						"2024-12-04": "0",
						"2024-12-05": "0",
						"2024-12-06": "0",
						"2024-12-07": "0",
						"2024-12-08": "0",
						"2024-12-09": "0",
						"2024-12-10": "0",
						"2024-12-31": "1",
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