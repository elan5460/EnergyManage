<template>
	<div>
		<el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()" size="small">
			<region-select-item  label="所属区域" v-model="dataForm.regionName" @getRegion="getSelectRegion"></region-select-item>
			<el-form-item label="表计类型">
	      		<el-select v-model="dataForm.type" clearable placeholder="请选择" style="width:90px;" @change="getDataList">
			    	<el-option v-for="item in meterTypeList" :key="item.value" :label="item.key" :value="item.value"></el-option>
				</el-select>
	      	</el-form-item>
	    	<hl-date-picker label="查询时间" v-model="dataForm.datatime" ></hl-date-picker>
	    	<el-form-item>
	       		<el-button @click="getDataList()">查询</el-button>
	    	</el-form-item>
		</el-form>
		<hltable v-bind:tburl="tbUrl" v-bind:tbcols="tbCols" ref="dataTable" 
			v-bind:tbstyle="tbStyle" v-bind:tbconfig="tbConfig" @dataDetail="dataDetail"/>
	</div>
</template>

<script>
	import regionSelect from '@/views/modules/pob/region-select'
	import hlDatePicker from '@/components/hl-date-picker'
	import hltable from '@/components/hltable'
	import tool from '@/utils/tool'
	export default{
		data(){
			return {
				loading :false,
        		meterTypeList : this.$sysConfig.getMeterTypes(),
				dataForm:{
					datatime : tool.formatDate(new Date(),'yyyy-MM-dd') ,
					regionid : this.$cookie.get("regionid"),
					regionName : this.$cookie.get("regionName"),
					type : 0
				},
	        	tbUrl :'/data/readMetering/pointbm',
	        	tbConfig : {
	        		isShowSelection : false,
	            	isShowRowIndex : true ,
	            	rowButtonType : 3,
	            	isShowPage :true
	        	}
			}
		},
		components:{
			hlDatePicker,
			hltable,
			'region-select-item':regionSelect
		},
	    computed: {
	      	mainTabs: {
	        	get () { return this.$store.state.common.mainTabs },
	        	set (val) { this.$store.commit('common/updateMainTabs', val) }
	      	},
	      	tbCols(){
	      		if(this.dataForm.type ==1 || this.dataForm.type === 2){
	      			return [
		      			{prop:"pointName", label:"表计名称"}
		      			,{prop:"commaddress", label:"表计通讯地址",width:'90'}
		      			,{prop:"datatime", label:"时间",width:'90'}
		      			,{prop:"flowV", label:"表计读数",width:'90'}
	      			]
	      		}else{
	      			return [
		      			{prop:"pointName", label:"表计名称",width:'60'}
		      			,{prop:"commaddress", label:"表计通讯地址",width:'90'}
		      			,{prop:"numberid", label:"表序号",width:'60'}
		      			,{prop:"rtuAddr", label:"终端地址",width:'70'}
		      			,{prop:"datatime", label:"时间",width:'90'}
		      			,{prop:"zyz", label:"正向有功总",width:'70'}
		      			,{prop:"zyj", label:"正向有功尖",width:'70'}
		      			,{prop:"zyf", label:"正向有功峰",width:'70'}
		      			,{prop:"zyp", label:"正向有功平",width:'70'}
		      			,{prop:"zyg", label:"正向有功谷",width:'70'}
		      			,{prop:"zwz", label:"正向无功总",width:'70'}
		      			,{prop:"fyz", label:"反向有功总",width:'70'}
		      			,{prop:"fyj", label:"反向有功尖",width:'70'}
		      			,{prop:"fyf", label:"反向有功峰",width:'70'}
		      			,{prop:"fyp", label:"反向有功平",width:'70'}
		      			,{prop:"fyg", label:"反向有功谷",width:'70'}
		      			,{prop:"fwz", label:"反向无功总",width:'70'}
	      			]
	      		}
	      	}
	    },
		mounted(){
			this.getDataList()
		},
		methods:{
			getDataList(){
				this.$refs.dataTable.loadData({
					datatime :  this.dataForm.datatime ,
					regionid : this.dataForm.regionid,
					type :this.dataForm.type||0
				});
			},
			getSelectRegion(data){
				this.dataForm.regionid = data.id
		       	this.dataForm.regionName = data.label;
		       	this.getDataList();
			},
		    dataDetail(row){
		    	var name = "pointBm"
		        var newTabs = this.mainTabs.filter(item => item.name !== name)
		        if (newTabs.length >= 1) {
		          this.mainTabs = newTabs
		        } 
		        this.$router.push({ name: name,params: { pointid : row.pointid ,datatime :row.datatime}})
		    }
		}
	}
</script>

<style>
</style>