<template>
	<view class="luna-datepicker">
		<view v-if="!show" class="luna-datepicker-input-wrapper" @click="open">
			<view class="luna-datepicker-input" v-if="value">{{value}}</view>
			<view class="luna-datepicker-input-placeholder" v-else>{{placeholder}}</view>
		</view>

		<view v-if="show" class="luna-datepicker__mask" :class="{'luna-datepicker--mask-show':animate}" @click="close">
		</view>
		<view v-if="show" class="luna-datepicker__content" :class="{'luna-datepicker--ani-show':animate}">
			<view class="luna-datepicker__header">
				<view @click="cancel()" class="luna-picker-action luna-picker-action-cancel">取消</view>
				<view class="luna-picker-title">{{title}}</view>
				<view @click="confirm()" class="luna-picker-action luna-picker-action-confirm">完成</view>
			</view>

			<view class="luna-datepicker__box">
				<view class="switch-actions">
					<view class="switch-action" :class="['solar', type === 'solar' ? 'current' : '']"
						@click="changeType('solar')">公历</view>
					<view class="switch-action" :class="['lunar', type === 'lunar' ? 'current' : '']"
						@click="changeType('lunar')">农历</view>
				</view>
				<picker-view :indicator-style="indicatorStyle" :value="selectedValue" @change="bindChange">
					<picker-view-column>
						<view class="luna-picker-item" v-for="(item, index) in years" :key="index">{{item}}年</view>
					</picker-view-column>
					<picker-view-column>
						<view class="luna-picker-item" v-for="(item, index) in months" :key="index">{{item}}</view>
					</picker-view-column>
					<picker-view-column>
						<view class="luna-picker-item" v-for="(item, index) in days" :key="index">{{item}}</view>
					</picker-view-column>
				</picker-view>
			</view>
		</view>
	</view>
</template>
<script>
	import {
		calendar
	} from "./calendar";
	export default {
		name: "chaste-luna-datepicker",
		data() {
			return {
				show: false,
				animate: false,
				type: '', //lunar农历，solar公历
				lastType: '',
				years: [], //年
				months: [], //月
				days: [], //日
				selectedValue: [0, 0, 0], //
				date: {}, // 完整日期数据
				innerValue: '',
				indicatorStyle: `height: 40px;`,
			};
		},
		watch: {
			selectedValue(val, oldVal) {
				if (this.type === this.lastType) {
					if (this.years.length <= 0) {
						this.getYears()
					}
					if (val[0] !== oldVal[0]) {
						this.getMonths()
						this.getDays()
					} else if (val[1] !== oldVal[1]) {
						this.getDays()
					}
				} else {
					this.getYears()
					this.getMonths()
					this.getDays()
				}
				this.lastType = this.type
				if (this.type === "solar") {
					this.setSolarDate(val[0], val[1], val[2]);
				} else {
					this.setLunarDate(val[0], val[1], val[2]);
				}
			},
			value(val, oldVal) {
				if (!val || val === this.innerValue) {
					return
				}
				this.setDate(val)
			}
		},
		props: {
			title: {
				type: String,
				default: '请选择日期',
			},
			value: {
				type: String,
				default: '',
			},
			placeholder: {
				type: String,
				default: '请选择日期',
			},
			defaultValue: {
				type: String,
				default: '2000-01-01',
			},
			defaultType: {
				type: String,
				default: () => "solar",
			},
			minYear: {
				type: Number,
				default: () => 1900,
			},
			maxYear: {
				type: Number,
				default: () => 2100,
			},
		},
		mounted() {
			this.type = this.defaultType
			const value = this.value || this.defaultValue
			if (value) {
				this.innerValue = value
				// 公历的年月日
				this.setDate(value)
			}
		},
		methods: {
			setDate(date) {
				if (typeof date === 'string') {
					date = new Date(date.replace(getRegExp('-', 'g'), '/'))
				} else if (typeof date === 'number') {
					date = new Date(date)
				} else {
					console.log('invalid date')
					return false;
				}
				this.date = calendar.solar2lunar(date.getFullYear(), date.getMonth() + 1, date.getDate());
				this.setSelectedValue()
			},
			open() {
				this.show = true
				this.$nextTick(() => {
					setTimeout(() => {
						this.animate = true
					}, 50)
				})
			},
			close() {
				this.animate = false
				this.$nextTick(() => {
					setTimeout(() => {
						this.show = false
						this.$emit('close')
					}, 300)
				})
			},
			getYears() {
				this.years = [];
				if (this.type === "solar") {
					for (let i = this.minYear; i <= this.maxYear; i++) {
						// 年
						this.years.push(i);
					}
					return;
				}

				if (this.type === "lunar") {
					for (let i = this.minYear; i <= this.maxYear; i++) {
						// 年
						this.years.push(calendar.toChinaYear(i));
					}
					return;
				}
			},
			getMonths() {
				this.months = [];
				if (this.type === "solar") {
					for (let i = 1; i <= 12; i++) {
						//月
						this.months.push(i + "月");
					}
					return;
				}

				if (this.type === "lunar") {
					const year = this.selectedValue[0] + this.minYear;
					const leap_month = calendar.leapMonth(year);
					//返回农历 闰月没有就返回0
					for (let i = 1; i <= 12; i++) {
						this.months.push(calendar.toChinaMonth(i));
						if (i == leap_month) {
							this.months.push("闰" + calendar.toChinaMonth(i));
						}
					}
					return;
				}
			},
			getDays() {
				this.days = [];
				let year = this.selectedValue[0] + this.minYear;
				let month = this.selectedValue[1] + 1;
				if (this.type === "solar") {
					for (let i = 1; i <= calendar.solarDays(year, month); i++) {
						//日
						this.days.push(i + "日");
					}
					return;
				}

				if (this.type === "lunar") {
					// 农历返回天数
					// **leapMonth 返回闰月 calendar.leapMonth(year)
					// **monthDays 返回非闰月的天数
					// **leapDays 返回闰月的天数
					if (
						calendar.leapMonth(year) != 0 &&
						(calendar.leapMonth(year) == month ||
							month - 1 == calendar.leapMonth(year))
					) {
						for (
							let i = 1; i <= calendar.leapDays(year); i++
						) {
							this.days.push(calendar.toChinaDay(i));
						}
					} else {
						let lunarMonth = "";
						if (calendar.leapMonth(year)) {
							lunarMonth = month < calendar.leapMonth(year) ? month : month - 1;
						} else {
							lunarMonth = month;
						}
						for (let i = 1; i <= calendar.monthDays(year, lunarMonth); i++) {
							this.days.push(calendar.toChinaDay(i));
						}
					}
					return;
				}
			},
			bindChange(e) {
				this.selectedValue = e.detail.value;
			},
			setSelectedValue() {
				if (this.type === "solar") {
					const pickerYear = this.date.cYear - this.minYear;
					const pickerMonth = this.date.cMonth - 1;
					const pickerDay = this.date.cDay - 1;
					this.selectedValue = [pickerYear, pickerMonth, pickerDay];
				} else {
					const lunarDateArr = this.date.lunarDate.split("-");
					const pickerYear = Number(lunarDateArr[0]) - this.minYear;
					let pickerMonth = Number(lunarDateArr[1]) - 1;
					const pickerDay = Number(lunarDateArr[2]) - 1;

					const leap_month = calendar.leapMonth(pickerYear + this.minYear)
					if (leap_month && pickerMonth > leap_month) { // 如果有闰月
						pickerMonth = pickerMonth + 1
					}
					this.selectedValue = [pickerYear, pickerMonth, pickerDay];
				}
			},
			setSolarDate(y_idx, m_idx, d_idx) {
				const date = calendar.solar2lunar(y_idx + this.minYear, m_idx + 1, d_idx + 1);
				//设置公历数据
				date.type = 'solar'
				this.date = date;
			},
			setLunarDate(y_idx, m_idx, d_idx) {
				let y = y_idx + this.minYear,
					m = m_idx + 1,
					d = d_idx + 1
				let isLeapMonth = false
				const leapMonth = calendar.leapMonth(y)
				if (leapMonth && m > leapMonth) {
					m = m - 1
				}
				isLeapMonth = (leapMonth) == m
				const date = calendar.lunar2solar(y, m, d, isLeapMonth);
				date.type = 'lunar'
				this.date = date;
			},
			cancel() {
				this.$emit("cancel");
				this.close()
			},
			confirm() {
				const y = this.date.cYear
				const m = this.date.cMonth
				const d = this.date.cDay
				const value = String(y) + '-' + String(m).padStart(2, 0) + '-' + String(d).padStart(2, 0)
				this.innerValue = value
				this.$emit('input', value)
				this.close()
			},
			fixZero(s) {
				if (s <= 9) return '0' + s
				return s
			},
			changeType(type) {
				this.type = type;
				this.setSelectedValue()
			},
		},
	};
</script>

<style lang="scss">
	.luna-datepicker {
		display: flex;
		flex-direction: column;
	}

	picker-view {
		width: 100%;
		height: 300px;
		margin-top: 10px;
	}

	.switch-action {
		font-size: 12px;
		border: 1px solid #ccc;
		margin-right: 10px;
		padding: 5px 10px;
		border: 1px #dcdfe6 solid;
		border-radius: 5px;
		background-color: #f5f5f5;
	}

	.switch-action.current {
		color: #FFF;
		background-color: #2979ff;
		border-color: #2979ff;
	}

	.luna-datepicker__mask {
		position: fixed;
		bottom: 0;
		top: 0;
		left: 0;
		right: 0;
		background-color: rgba(0, 0, 0, 0.4);
		transition-property: opacity;
		transition-duration: .3s;
		opacity: 0;
		/* #ifndef APP-NVUE */
		z-index: 99;
		/* #endif */
	}

	.luna-datepicker--mask-show {
		opacity: 1;
	}

	.luna-datepicker__content {
		background-color: #fff;
		border-top-left-radius: 10px;
		border-top-right-radius: 10px;
		box-shadow: 0px 0px 5px 3px rgba(0, 0, 0, 0.1);

		transition-property: transform;
		transition-duration: 0.3s;
		transform: translateY(460px);

		position: fixed;
		bottom: calc(var(--window-bottom));
		left: 0;
		right: 0;
		/* #ifndef APP-NVUE */
		z-index: 99;
		/* #endif */
	}

	.luna-datepicker--ani-show {
		transform: translateY(0);
	}

	.luna-datepicker__header {
		position: relative;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		height: 50px;
	}

	.luna-picker-action {
		padding: 0 15px;
	}

	.luna-picker-action-confirm {
		color: #2979ff;
	}

	.switch-actions {
		display: flex;
		flex-direction: row;
		justify-content: center;
		align-items: center;
	}

	.lunar-picker-title {
		flex: 1;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		align-items: center;
	}

	.luna-picker-item {
		display: flex;
		align-items: center;
		justify-content: center;
	}

	.luna-datepicker--fixed-width {
		width: 50px;
	}

	.luna-datepicker__header-text {
		text-align: center;
		width: 100px;
		font-size: 15px;
		color: #666;
	}

	.luna-datepicker__box {
		position: relative;
		// padding: 0 10px;
		padding-bottom: 7px;
	}
</style>
