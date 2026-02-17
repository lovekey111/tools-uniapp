<template>
	<view>
		<view class="input-container">
			<view class="date-container">
				<uni-section :title="'男生生日'" type="line"></uni-section>
				<uni-datetime-picker type="date" :clear-icon="false" v-model="maleBirthday" @show="clearCurve" />
			</view>
			<view class="date-container" style="padding-right: 20rpx;">
				<uni-section :title="'女生生日'" type="line"></uni-section>
				<uni-datetime-picker type="date" :clear-icon="false" v-model="femaleBirthday" @show="clearCurve" />
			</view>
		</view>
		<view class="input-container">
			<view class="date-container">
				<uni-section :title="'曲线起始日'" type="line"></uni-section>
				<uni-datetime-picker type="date" :clear-icon="false" v-model="startDate" @show="clearCurve" />
			</view>
			<view class="date-container" style="padding-right: 20rpx;">
				<uni-section :title="'月经起始日'" type="line"></uni-section>
				<uni-datetime-picker type="date" :clear-icon="false" v-model="menstrualDate" @show="clearCurve" />
			</view>
		</view>
		<view class="input-container">
			<view class="date-container">
				<uni-section :title="'月经周期（天）'" type="line"></uni-section>
				<uni-easyinput v-model="menstrualCycle" type="number" focus placeholder="请输入月经周期"></uni-easyinput>
			</view>
			<view class="date-container" style="padding-right: 20rpx;">
				<uni-section :title="'展示天数'" type="line"></uni-section>
				<uni-easyinput v-model="deltaDays" type="number" focus placeholder="请输入展示天数"></uni-easyinput>
			</view>
		</view>
		
		<view class="button-container">
			<button class="button generate-button" type="primary" @click="drawCurve(),saveUserInfo()">生成曲线</button>
			<button class="button generate-button" type="primary" @click="removeUserInfo()">清除缓存</button>
		</view>

		<!-- <view style="width:750rpx; height:750rpx"><l-echart ref="chartRef" @finished="init()"></l-echart></view> -->
		<view style="width:750rpx; height:750rpx; top: 10rpx"><l-echart ref="chartRef"></l-echart></view>
		<!-- 提示信息弹窗 -->
		<uni-popup ref="message" type="message">
			<uni-popup-message :type="'error'" :message="'请输入生日'" :duration="2000"></uni-popup-message>
		</uni-popup>
	</view>
</template>

<script>
// import * as echarts from '@/uni_modules/lime-echart/static/echarts.min'

export default {
	data() {
		return {
			maleBirthday: '',
			femaleBirthday: '',
			startDate: this.getCurrentDate(),
			menstrualDate: '',
			menstrualCycle: '',
			deltaDays: 10,
			totalDeltaDays: 200,
			errorMessage: '',
			echarts: null,
			
			option: {
				text: '人体三曲线',
				tooltip: {
					trigger: 'axis',
					axisPointer: {
						type: 'shadow' 
					},
					confine: true
				},
				legend: {
					data: ['男生-体力节律', '男生-情绪节律', '男生-智力节律', '女生-体力节律', '女生-情绪节律', '女生-智力节律', '排卵日期'],
					textStyle: {
						fontSize: 10  // 图例文本的字体大小
					},
				},
				grid: {
					show: true
				},
				xAxis: {
					axisTick: {
						show: true, // 显示刻度线
						alignWithLabel: true, // 刻度线与标签对齐
						interval: 0 ,// 显示所有刻度线
					},
					splitLine: {
						show: true, // 显示分割线
						lineStyle: {
							color: '#ccc', // 分割线颜色
							type: 'solid', // 分割线类型
							width: 1 // 分割线宽度
						}
					},
					axisLabel: {
						rotate: 90,
						interval: 1
					},
					boundaryGap: false,
					data: []
				},
				yAxis: {
					axisTick: {
						show: true, // 显示刻度线
						alignWithLabel: true, // 刻度线与标签对齐
						interval: 0 ,// 显示所有刻度线
					},
					min: -1, // 保持 y 轴最小值不变
					max: 1  // 保持 y 轴最大值不变
				},
				splitLine: {
					show: true
				},
				series: [
					{
						name: '男生-体力节律',
						type: 'line',
						data: [],
						smooth: true,
						sampling: 'average',
						showSymbol: 'false',
						symbol: 'none'
					},
					{
						name: '男生-情绪节律',
						type: 'line',
						data: [],
						smooth: true,
						sampling: 'average',
						showSymbol: 'false',
						symbol: 'none'
					},
					{
						name: '男生-智力节律',
						type: 'line',
						data: [],
						smooth: true,
						sampling: 'average',
						showSymbol: 'false',
						symbol: 'none'
					},
					{
						name: '女生-体力节律',
						type: 'line',
						data: [],
						smooth: true,
						sampling: 'average',
						showSymbol: 'false',
						symbol: 'none'
					},
					{
						name: '女生-情绪节律',
						type: 'line',
						data: [],
						smooth: true,
						sampling: 'average',
						showSymbol: 'false',
						symbol: 'none'
					},
					{
						name: '女生-智力节律',
						type: 'line',
						data: [],
						smooth: true,
						sampling: 'average',
						showSymbol: 'false',
						symbol: 'none'
					},
					{
						name: '排卵日期',
						type: 'scatter',
						data: [],
						smooth: true,
						sampling: 'average',
					}
				],
				dataZoom: [{
					type: 'inside',          // 内置拖动
					start: 0,
					end: 80,
					moveOnMouseMove: true,   // 允许拖动
					zoomOnMouseWheel: false,// 禁用滚轮缩放
					debounce: 100 ,// 设置防抖延迟，单位为毫秒
				}]
			},
		};
	},
	created() {
		this.chartInstance = null; // 普通属性，非响应式
	},
	// 组件能被调用必须是组件的节点已经被渲染到页面上
	async mounted() {
		const module = await import('@/uni_modules/lime-echart/static/echarts.min.js');
		this.echarts = module;
		this.$refs.chartRef.init(this.echarts, chart =>{
			this.chartInstance = chart;
		});
		const userInfo = uni.getStorageSync('user_info');
		if(userInfo){
			this.maleBirthday = userInfo.maleBirthday;
			this.femaleBirthday = userInfo.femaleBirthday;
			this.menstrualDate = userInfo.menstrualDate;
			this.menstrualCycle = userInfo.menstrualCycle;
			this.deltaDays = userInfo.deltaDays;
		}
	},
	computed:{
		deltaDaysRadio() {
			return this.deltaDays / this.totalDeltaDays * 100;
		}
	},
	methods: {
		getCurrentDate(){
			const currentDate = new Date();
			return currentDate.toISOString().split("T")[0];
		},
		
		clearCurve(){
			this.chartInstance.clear();
		},
		
		//根据填写内容判断展示曲线
		judgeCondition(){
			let passFlag = true
			if(this.maleBirthday == '' && this.femaleBirthday == ''){
				this.$refs.message.open();
				passFlag = false;
			}
			if(this.deltaDays == '') this.deltaDays = 10;
			if(this.startDate == '') this.startDate = this.getCurrentDate();
			return passFlag;
		},
		
		generateMaleCurveData(days){
			this.option.series[0].data = this.generateCurveData(this.maleBirthday, 23, days, this.startDate);
			this.option.series[1].data = this.generateCurveData(this.maleBirthday, 28, days, this.startDate);
			this.option.series[2].data = this.generateCurveData(this.maleBirthday, 33, days, this.startDate);
		},
		
		generateFemaleCurveData(days){
			this.option.series[3].data = this.generateCurveData(this.femaleBirthday, 23, days, this.startDate);
			this.option.series[4].data = this.generateCurveData(this.femaleBirthday, 28, days, this.startDate);
			this.option.series[5].data = this.generateCurveData(this.femaleBirthday, 33, days, this.startDate);
		},
		
		drawCurve(){
			if(!this.judgeCondition()) return;
			
			this.option.xAxis.data = this.generateDateArray(this.totalDeltaDays, this.startDate).map(str => str.replace(/-/g, ''));
			if(this.maleBirthday != '' && this.femaleBirthday == ''){
				this.generateMaleCurveData(this.totalDeltaDays);	
			}else if(this.maleBirthday == '' && this.femaleBirthday != ''){
				this.generateFemaleCurveData(this.totalDeltaDays);
			}else{
				this.generateMaleCurveData(this.totalDeltaDays);	
				this.generateFemaleCurveData(this.totalDeltaDays);
			}
			this.option.series[6].data = this.generateMenstrualCurveData(this.menstrualDate, this.menstrualCycle);
			this.option.dataZoom[0].end = this.deltaDaysRadio;
			this.chartInstance.setOption(this.option);
		},
		
		saveUserInfo(){
			// 存储数据
			uni.setStorageSync('user_info', {
			  maleBirthday: this.maleBirthday,
			  femaleBirthday: this.femaleBirthday,
			  menstrualDate: this.menstrualDate,
			  menstrualCycle: this.menstrualCycle,
			  deltaDays: this.deltaDays,
			});
		},
		
		removeUserInfo(){
			// 移除数据
			uni.removeStorageSync('user_info');
			this.maleBirthday = '';
			this.femaleBirthday = '';
			this.startDate = '';
			this.menstrualDate = '';
			this.menstrualCycle = '';
			this.deltaDays = '';
		},
		
		//生成连续日期列表
		generateDateArray(days, startDate) {
			const start = startDate == '' ? new Date(this.getCurrentDate()): new Date(startDate);
			const dateArray = [];
			// 使用循环生成日期数组
			var i = 0;
			while (i < days) {
				// 将当前日期转换为字符串格式（如 YYYY-MM-DD），并添加到数组中
				const dateString = start.toISOString().split("T")[0];
				dateArray.push(dateString);
				// 将日期加一天
				start.setDate(start.getDate() + 1);
				i += 1;
			}
			return dateArray;
		},
		
		//生成曲线数据
		generateCurveData(birthday, modNum, days, startDate){
			const dateArray = this.generateDateArray(days, startDate);
			const curveArray = [];
			for(const eachDate of dateArray){
				curveArray.push(this.normalDaysMod(this.computeTotalDays(birthday, eachDate), modNum));
			}
			return curveArray;
		},
		
		//生成排卵时间数据
		generateMenstrualCurveData(menstrualDate, menstrualCycle){
			const start = new Date(this.startDate);
			const end = new Date(menstrualDate);
			while(start - end < 0){
				end.setDate(end.getDate() - menstrualCycle);
			}
			const afterDays = Math.floor((start - end) / (1000 * 60 * 60 * 24)) + 1;
			//月经周期起始天数
			const periodOvulationStart = Math.floor(menstrualCycle / 2) - 4;
			//月经周期终止天数
			const periodOvulationEnd = Math.floor(menstrualCycle / 2) + 5;
			const curveArray = [];
			
			for(let i=0; i<this.totalDeltaDays; i++){
				//月经周期的第几天
				let dayInPeriod = (afterDays + i) % menstrualCycle;
				//当前日期在月经周期的第几天
				if(dayInPeriod >= periodOvulationStart && dayInPeriod <= periodOvulationEnd){
					curveArray.push(1);
				}else{
					curveArray.push(null);
				}
			}
			return curveArray;
		},
		
		normalDaysMod(totalDays, modNum){
			return Math.sin(2 * Math.PI * (totalDays % modNum) / modNum);
		},
		
		computeTotalDays(birthday, aimDate){
			const start = new Date(birthday);
			const end = new Date(aimDate);
			const timeDifference = end.getTime() - start.getTime();
			return Math.floor(timeDifference / (1000 * 60 * 60 * 24));
		},
	}
}
</script>

<style>
	.curve {
		width: 300px; 
		height: 300px;
	}
	.chart-container {
	  width: 100%;
	  height: 500rpx;
	}
	.date-container {
		width: 45%;
		padding-left: 20rpx;
	}
	.input-container {
		display: flex;
		justify-content: space-between;
	}
	.button-container{
		display: flex;
		justify-content: space-around;
		padding: 0 100rpx;
	}
	.generate-button{
		width: 40%;
		height: 80%;
		margin-top: 40rpx;
		font-size: 30rpx;
	}
</style>
