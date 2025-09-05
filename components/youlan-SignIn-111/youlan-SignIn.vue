<template>
	<view class="calendar-box" :class="{ 'dark-theme': darkMode }">
		<view class="top">
			<view>
				<h4>已连续签到 <span> {{checkinDays}} </span> 天</h4>
				<p v-if="isIntegral">今日获得+{{integral}}积分</p>
			</view>
			<view class="right">
				<span v-if="supplementary" @click="shift">补签</span>
			</view>
		</view>
		<view class="miss_box">
			<view class="month">
				<view class="previous" @click="lastMonth">
					<span v-show="showPrevious">上月</span>
				</view>
				<view>{{year}}年{{month}}月</view>
				<view class="previous" @click="nextMonth">
					<span v-show="showNext">下月</span>
				</view>
				<picker v-if="checkDate" class="picker" mode="date" fields="month" @change="changeDate" />
			</view>
			<view class="week">
				<view :style="'color:'+(weeks==currentWeek?bgweek:'')+';'" v-for="weeks in weekArr">{{weeks}}</view>
			</view>
			<view class="day">
				<view :class="[{'checkday':days.date==''},{'choose':days.date==localDate || days.flag }]"
					:style="'background:'+(days.date==localDate || days.flag ? bgday:'')+';'" v-for="(days,index) in dayArr" :key="index"
					@click="signToday(days,index)">
					{{days.day}}
					<view :class="[{'circle':days.flag},{'repair':days.day<day},{'sign':days.day==day}]"></view>
				</view>
			</view>
		</view>
		<view class="but">
			
			<button :disabled='currentMonth == month ? false : true'  :style="'background: ' + (isDaySignedIn ? signin_but_bg : (currentMonth == month ? bgday : signin_but_bg)) + ';'" @click="daySign(dayArr[aheadDay + day - 1])">签到</button>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			lang: {
				type: String,
				default: 'zh'
			},
			type: {
				type: String,
				default: 'calendar'
			},
			checkDate: {
				type: Boolean,
				default: false
			},
			bgweek: {
				type: String,
				default: '#FF8F22'
			},
			bgday: {
				type: String,
				default: '#FF8F22'
			},
			// 不能签到时的按钮颜色
			signin_but_bg: {
				type: String,
				default: '#909399'
			},
			supplementary: {
				type: Boolean,
				default: true
			},
			already: {
				type: Array,
				default: []
			},
			checkinDays: {
				type: [Number, String],
				default: 0
			},
			integral: {
				type: [Number, String],
				default: 0
			},
			isIntegral: {
				type: Boolean,
				default: false
			},
			darkMode: {
				type: Boolean,
				default: false
			}
		},
		watch: {
			already: {
				handler(v) {
				  this.getAlready();
				},
				immediate: true, // 第一次赋值时触发
				deep: true // 深度监听
			}
		},
		data() {
			return {
				currentWeek: '', // 今天周几
				dayArr: [], // 当前月每日
				localDate: '', // 今天日期
				day: new Date().getDate(), // 当前日
				year: new Date().getFullYear(), // 当前年
				month: new Date().getMonth() + 1, // 当前月
				weekArr: ['日', '一', '二', '三', '四', '五', '六'], // 每周
				aheadDay: 0,	// 前方空白天数数量
				currentMonth: new Date().getMonth() + 1,
					showPrevious: true,
				showNext: true,
				// 判断今天是否已经签到
				isDaySignedIn: false,
				// 已经签到的日期
				alreadySignedDates: []
			}
		},
		mounted() {
			let that = this;
			// 初始日期
			that.initDate();
			// 今天日期
			that.localDate = that.year + '-' + that.formatNum(that.month) + '-' + that.formatNum(that.day);
			// 中英切换
			if (that.lang != 'zh') that.weekArr = ['Sun', 'Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat'];
			// 今天周几
			that.currentWeek = that.weekArr[new Date().getDay()];
		},
		methods: {
			// 获取所有已签到日期的时间戳
			getAllSignedTimestamps() {
				return [...this.already, ...this.alreadySignedDates];
			},
			setAlready(days) {
				this.alreadySignedDates = days;
				this.getAlready();
			},
			// 渲染已经签到的日期
			getAlready() {
				const signedTimestamps = this.getAllSignedTimestamps();
				this.isDaySignedIn = false;
				
				this.dayArr.forEach(day => {
					if (day.date) {
						const dayTimestamp = new Date(day.date).getTime();
						day.flag = signedTimestamps.includes(dayTimestamp);
						
						// 检查今日是否已签到
						if (day.date === this.localDate && day.flag) {
							this.isDaySignedIn = true;
						}
					} else {
						day.flag = false;
					}
				});
			},
			// 补签触发的方法
			shift() {
				this.$emit('shift')
			},
			// 选择年月
			changeDate(e) {
				let that = this;
				that.year = parseInt(e.detail.value.split('-')[0]);
				that.month = parseInt(e.detail.value.split('-')[1]);
				that.initDate();
			},
			// 今日签到
			daySign(obj) {
				// 今天日期加上前方空白日期数量
				let index = this.aheadDay + this.day - 1
				if(this.dayArr[index].flag) {
					// 已经签到后不能签到
					return false
				}
				this.dayArr[index].flag = true;
				this.$emit('change', obj.date);
				this.isDaySignedIn = true;
				uni.showToast({
					title: '已签到',
					icon: 'success'
				});
			},
			// 点击补签
			signToday(obj, index) {
				let that = this;
				// 指定签到类型可访问
				if (that.type == 'calendar') return;
				// 限制本月可进行签到
				if ((new Date().getMonth() + 1) != parseInt(obj.date.split('-')[1])) return;
				// 禁用非本月点击签到且限制签到日期小于本日
				if (obj.date != '' && obj.day < that.day) {
					// 开启已签到提醒
					if (that.dayArr[index].flag) {
						uni.showToast({
							title: '已签到',
							icon: 'none'
						});
					} else {
						if(that.day > obj.day && !that.supplementary) {
							return
						}
							uni.showToast({
							title: that.day > obj.day ? '补签成功' : '签到成功',
							icon: 'success'
						});
						that.dayArr[index].flag = true;
						that.$emit('change', obj.date);
					}
				}
			},
			// 初始化日期
			initDate() {
				this.dayArr = this.generateCalendarDays();
				this.getAlready();
			},
			// 生成日历数据
			generateCalendarDays() {
				const days = [];
				const totalDays = new Date(this.year, this.month, 0).getDate();
				const firstDayOfWeek = new Date(this.year, this.month - 1, 1).getDay();
				const lastMonthDays = new Date(this.year, this.month - 1, 0).getDate();
				
				this.aheadDay = firstDayOfWeek;
				
				// 添加上月的空白日期
				for (let i = firstDayOfWeek - 1; i >= 0; i--) {
					days.push({
						date: '',
						day: lastMonthDays - i,
						flag: false
					});
				}
				
				// 添加当月日期
				for (let i = 1; i <= totalDays; i++) {
					days.push({
						date: `${this.year}-${this.formatNum(this.month)}-${this.formatNum(i)}`,
						day: i,
						flag: false
					});
				}
				
				// 添加下月的空白日期
				const lastDayOfWeek = new Date(this.year, this.month - 1, totalDays).getDay();
				if (lastDayOfWeek !== 6) {
					for (let i = 1; i <= 6 - lastDayOfWeek; i++) {
						days.push({
							date: '',
							day: i,
							flag: false
						});
					}
				}
				
				return days;
			},
			// 格式化日期位数
			formatNum(num) {
				return num < 10 ? ('0' + num) : num;
			},
			// 更新上下月按钮显示状态
			updateMonthNavigation() {
				const currentDate = new Date();
				const currentYear = currentDate.getFullYear();
				const currentMonth = currentDate.getMonth() + 1;
				
				this.showPrevious = this.year >= currentYear || this.month > currentMonth + 1;
				this.showNext = this.year <= currentYear || this.month < currentMonth;
			},
			// 上一个月
			lastMonth() {
				if (this.month == 1) {
					this.year -= 1;
					this.month = 12;
				} else {
					this.month -= 1;
				}
				
				this.initDate();
				this.updateMonthNavigation();
			},
			// 下一个月
			nextMonth() {
				if (this.month == 12) {
					this.year += 1;
					this.month = 1;
				} else {
					this.month += 1;
				}
				
				this.initDate();
				this.updateMonthNavigation();
			}
		}
	}
</script>

<style scoped>
	/* CSS变量 - 浅色主题 */
	.calendar-box {
		--bg-primary: #FFFFFF;
		--bg-secondary: #FFFFFF;
		--text-primary: #333333;
		--text-secondary: #666666;
		--text-disabled: #999999;
		--date-text: #333333;
		--theme-color: #FF871E;
		--theme-border: #FF871E;
		--button-text: #FFFFFF;
		--supplement-bg: transparent;
		--supplement-border: #FF871E;
	}
	
	/* CSS变量 - 暗色主题 */
	.calendar-box.dark-theme {
		--bg-primary: #1a1a1a;
		--bg-secondary: #2d2d2d;
		--text-primary: #ffffff;
		--text-secondary: #e0e0e0;
		--text-disabled: #999999;
		--date-text: #ffffff;
		--theme-color: #ffc470;
		--theme-border: #ffc470;
		--button-text: #ffffff;
		--supplement-bg: rgba(255, 196, 112, 0.1);
		--supplement-border: rgba(255, 196, 112, 0.3);
	}
	
	.calendar-box {
		width: 100%;
		flex-direction: column;
		justify-content: center;
		
		
	}
	
	.miss_box {
		width: 100%;
		padding: 20rpx 40rpx;
		box-sizing: border-box;
		background-color: var(--bg-primary);
		border-radius: 25rpx;
	}
	
	.calendar-box,
	.month,
	.week,
	.day {
		display: flex;
		align-items: center;
		justify-content: space-between;
	}
	
	.week, .day {
		display: grid;
		grid-template-columns: repeat(7, 1fr); /* 一行7列，每列等宽 */
	}
	
	.month,
	.week,
	.day {
		width: 100%;
	}
	
	.month {
		margin: 30rpx 0;
		position: relative;
		color: var(--text-primary);
	}
	
	.picker {
		width: 160rpx;
		height: 40rpx;
		position: absolute;
		top: 20rpx;
		left: 50%;
		transform: translate(-50%, -50%);
	}
	
	.day {
		flex-wrap: wrap;
	}
	
	.week>view,
	.day>view {
		width: 70rpx;
		height: 70rpx;
		margin: 10rpx;
		text-align: center;
		position: relative;
		line-height: 70rpx;
		color: var(--date-text);
	}
	
	.checkday {
		color: var(--text-disabled);
	}
	
	.choose {
		color: var(--button-text);
		border-radius: 50%;
	}
	
	.circle {
		width: 10rpx;
		height: 10rpx;
		border-radius: 50%;
		position: absolute;
		bottom: 10%;
		left: 50%;
		transform: translate(-50%, -50%);
	}
	
	.sign {
		background-color: var(--bg-primary);
	}
	
	.repair {
		background-color: var(--bg-primary);
	}
	
	.but {
		display: flex;
		flex: 1;
		margin-top: 80rpx;
		justify-content: center;
	}
	.but button {
		width: 650rpx;
		height: 80rpx;
		line-height: 80rpx;
		border-radius: 30rpx;
		border: 1px solid transparent;
		outline: none;
		color: var(--button-text);
		font-size: 32rpx;
	}
	uni-button:after {
		border: none;
	}
	
	.top {
		display: flex;
		width: 100%;
		justify-content: space-between;
		padding: 30rpx;
		box-sizing: border-box;
		background-color: var(--bg-secondary);
		border-radius: 20rpx;
		margin-bottom: 16rpx;
		align-items: center;
	}
	.top h4 {
		font-weight: 600;
		font-size: 36rpx;
		line-height: 50rpx;
		color: var(--text-primary);
	}
	.top h4 span {
		color: var(--theme-color);;
		margin: 0 10rpx;
		font-size: 32rpx
	}
	.top p {
		font-style: normal;
		font-weight: normal;
		font-size: 28rpx;
		line-height: 40rpx;
		color: var(--theme-color);
	}
	
	.right span {
		font-style: normal;
		font-weight: normal;
		font-size: 24rpx;
		line-height: 48rpx;
		color: var(--theme-color);
		border: 1px solid var(--supplement-border);
		background: var(--supplement-bg);
		padding: 10rpx 20rpx;
		border-radius: 32rpx;
		transition: background-color 0.2s ease;
	}
	.month .previous {
		min-width: 35px;
	}
</style>
