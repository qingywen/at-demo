<template>
	<div id="app">
		<div class="header"><img src="./assets/logo.png" /></div>
		<input-at :list="list" :autoDelete="true"/>
		<!-- <el-input class="input" @input="inputHandler"></el-input> -->
		<div class="description">
			<div>Case说明</div>
			<el-checkbox-group size="small" v-model="checkedList">
				<el-checkbox v-for="item in taskList" :key="item.label" :label="item.label" disabled>{{item.task}}</el-checkbox>
			</el-checkbox-group>
		</div>
  	</div>
</template>

<script>
import InputAt from './components/InputAt';

export default {
	components: {
		InputAt
	},
	data() {
		let list = [{
			name: "明日香",
			picture: "../src/assets/1.png"
		}, {
			name: "艾伦",
			picture: "../src/assets/2.png"
		}, {
			name: "三笠",
			picture: "../src/assets/3.png"
		}, {
			name: "鲁鲁修",
			picture: "../src/assets/1.png"
		}, {
			name: "圣罗兰",
			picture: "../src/assets/2.png"
		}, {
			name: "明日香2",
			picture: "../src/assets/1.png"
		}, {
			name: "艾伦2",
			picture: "../src/assets/2.png"
		}, {
			name: "三笠2",
			picture: "../src/assets/3.png"
		}, {
			name: "鲁鲁修2",
			picture: "../src/assets/1.png"
		}, {
			name: "圣罗兰2",
			picture: "../src/assets/2.png"
		}]
		return {
			list,
			taskList: [{
				label: 'task1',
				task: '唤起提示：输入@唤起提示，邮箱场景不唤起',
			}, {
				label: 'task2',
				task: '提示列表：根据@定位',
			}, {
				label: 'task3',
				task: '提示列表：自动选中第一个且聚焦',
			}, {
				label: 'task4',
				task: '提示列表：按上/下可循环切换选中值',
			}, {
				label: 'task5',
				task: '提示列表：需根据输入进行过滤',
			}, {
				label: 'task6',
				task: '插入文字：点击列表插入文字',
			}, {
				label: 'task7',
				task: '插入文字：回车插入文字',
			}, {
				label: 'task8',
				task: '自动删除：根据autoDelete属性判断是否自动删除',
			}, {
				label: 'task9',
				task: '流线箭头：用CSS完成流线箭头',
			}, {
				label: 'task10',
				task: '多行模式：支持多行模式',
			}, {
				label: 'task11',
				task: '中文匹配：支持根据文字拼音来过滤提示信息',
			}, {
				label: 'task12',
				task: 'slot插件：支持slot扩展形式',
			}], 
			checkedList: ['task1', 'task2', 'task3', 'task4', 'task5', 'task6', 'task7', 'task8', 'task9', 'task10']
		}
  	},
	methods: {
		inputHandler() {
			
		}
	}
 
};
</script>

<style>
#app {
  font-family: Helvetica Neue, Helvetica, PingFang SC, Hiragino Sans GB,
    Microsoft YaHei, SimSun, sans-serif;
}
.header {
  text-align: center;
}
.input {
	width: 300px;
	margin: 10px 565px;
}
.description {
	text-align: center;
	width: 320px;
	position: relative;
	left: 0;
	right: 0;
	margin: 60px auto 0;
}
.description>div {
	margin-bottom: 20px;
}
.description .el-checkbox {
	display: block;
	width: auto;
	margin: 5px 0 !important;
	text-align: left;
}
</style>
