<template>
	<div class="mpvue-picker">
		<div :class="{'pickerMask':showPicker}" @click="maskClick" catchtouchmove="true"></div>
		<div class="mpvue-picker-content " :class="{'mpvue-picker-view-show':showPicker}">
			<div class="mpvue-picker__hd" catchtouchmove="true">
				<div class="mpvue-picker__action" @click="pickerCancel">取消</div>
				<div class="mpvue-picker__action" :style="{color:themeColor}" @click="pickerConfirm">确定</div>
			</div>
			<picker-view indicator-style="height: 40px;" class="mpvue-picker-view" :value="pickerValue" @change="pickerChange">
				<block>
					<picker-view-column>
						<div class="picker-item" v-for="(item,index) in provinceDataList" :key="index">{{item.name}}</div>
					</picker-view-column>
					<picker-view-column>
						<div class="picker-item" v-for="(item,index) in cityDataList" :key="index">{{item.name}}</div>
					</picker-view-column>
					<picker-view-column>
						<div class="picker-item" v-for="(item,index) in areaDataList" :key="index">{{item.name}}</div>
					</picker-view-column>
				</block>
			</picker-view>
		</div>
	</div>
</template>

<script>
	// import provinceData from './city-data/province.js';
	// import cityData from './city-data/city.js';
	// import areaData from './city-data/area.js';
	export default {
		data() {
			return {
				pickerValue: [0, 0, 0],
				provinceDataList: [],
				cityDataList: [],
				areaDataList: [],
				/* 是否显示控件 */
				showPicker: false,
			};
		},
		created() {
			this.init()
		},
		props: {
			themeColor: String
		},
		watch: {

		},
		created() {
			// 加载数据库的地区数据并保存
			this.$request({
				url: this.$api.default.district
			}).then(res => {
				if (res.code == 0) {
					console.log(res);
					this.provinceDataList = res.data;
					this.cityDataList = res.data[0].list;
					this.areaDataList = res.data[0].list[0].list;
				}
			});
		},
		methods: {
			init() {
				// this.handPickValueDefault(); // 对 pickerValueDefault 做兼容处理
				this.provinceDataList = res.data;
				this.cityDataList = res.data[0].list;
				this.areaDataList = res.data[0].list[0].list;
			},
			show() {
				setTimeout(() => {
					this.showPicker = true;
				}, 0);
			},
			maskClick() {
				this.pickerCancel();
			},
			pickerCancel() {
				this.showPicker = false;
				// this._$emit('onCancel');
			},
			pickerConfirm(e) {
				this.showPicker = false;
				this._$emit('onConfirm');
			},
			showPickerView() {
				this.showPicker = true;
			},
			// handPickValueDefault() {
			// 	if (this.pickerValueDefault !== [0, 0, 0]) {
			// 		if (this.pickerValueDefault[0] > provinceData.length - 1) {
			// 			this.pickerValueDefault[0] = provinceData.length - 1;
			// 		}
			// 		if (this.pickerValueDefault[1] > cityData[this.pickerValueDefault[0]].length - 1) {
			// 			this.pickerValueDefault[1] = cityData[this.pickerValueDefault[0]].length - 1;
			// 		}
			// 		if (this.pickerValueDefault[2] > areaData[this.pickerValueDefault[0]][this.pickerValueDefault[1]].length - 1) {
			// 			this.pickerValueDefault[2] = areaData[this.pickerValueDefault[0]][this.pickerValueDefault[1]].length - 1;
			// 		}
			// 	}
			// },
			pickerChange(e) {
				// console.log(e);
				let changePickerValue = e.mp.detail.value;
				if (this.pickerValue[0] !== changePickerValue[0]) {
					// 第一级发生滚动
					console.log(changePickerValue[0]);
					this.cityDataList = this.provinceDataList[changePickerValue[0]].list;
					this.areaDataList = this.provinceDataList[changePickerValue[0]].list[0].list;
					changePickerValue[1] = 0;
					changePickerValue[2] = 0;
				} else if (this.pickerValue[1] !== changePickerValue[1]) {
					// 第二级滚动
					this.areaDataList = this.provinceDataList[changePickerValue[0]].list[changePickerValue[1]].list;
					changePickerValue[2] = 0;
				}
				this.pickerValue = changePickerValue;
				this._$emit('onChange');
			},
			_$emit(emitName) {
				console.log(emitName);
				let pickObj = {
					// 省市区名字
					provinceName: this._getProvinceName(),
					cityName: this._getCityName(),
					areaName: this._getAreaName(),
					// 省市区ID
					provinceId: this._getProvinceId(),
					cityId: this._getCityId(),
					districtId: this._getDistrictId()
				};
				this.$emit(emitName, pickObj);
			},
			_getProvinceName() {
				return this.provinceDataList[this.pickerValue[0]].name
			},
			_getCityName(){
				return this.cityDataList[this.pickerValue[1]].name
			},
			_getAreaName(){
				return this.areaDataList[this.pickerValue[2]].name
			},
			_getProvinceId() {
				return this.provinceDataList[this.pickerValue[0]].id;
			},
			_getCityId() {
				return this.cityDataList[this.pickerValue[1]].id;
			},
			_getDistrictId() {
				return this.areaDataList[this.pickerValue[2]].id;
			}
			
		}
	};
</script>

<style>
	.pickerMask {
		position: fixed;
		z-index: 1000;
		top: 0;
		right: 0;
		left: 0;
		bottom: 0;
		background: rgba(0, 0, 0, 0.6);
	}

	.mpvue-picker-content {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100%;
		transition: all 0.3s ease;
		transform: translateY(100%);
		z-index: 3000;
	}

	.mpvue-picker-view-show {
		transform: translateY(0);
	}

	.mpvue-picker__hd {
		display: flex;
		padding: 9px 15px;
		background-color: #fff;
		position: relative;
		text-align: center;
		font-size: 17px;
	}

	.mpvue-picker__hd:after {
		content: ' ';
		position: absolute;
		left: 0;
		bottom: 0;
		right: 0;
		height: 1px;
		border-bottom: 1px solid #e5e5e5;
		color: #e5e5e5;
		transform-origin: 0 100%;
		transform: scaleY(0.5);
	}

	.mpvue-picker__action {
		display: block;
		flex: 1;
		color: #1aad19;
	}

	.mpvue-picker__action:first-child {
		text-align: left;
		color: #888;
	}

	.mpvue-picker__action:last-child {
		text-align: right;
	}

	.picker-item {
		text-align: center;
		line-height: 40px;
		text-overflow: ellipsis;
		white-space: nowrap;
		font-size: 12px;
	}

	.mpvue-picker-view {
		position: relative;
		bottom: 0;
		left: 0;
		width: 100%;
		height: 238px;
		background-color: rgba(255, 255, 255, 1);
	}
</style>
