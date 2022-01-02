<template>
	<div id="app">
		<div class="header"><img src="./assets/logo.png" /></div>
		<input-at :list="list" :autoDelete="true"/>
		<!-- <el-input class="input" @input="inputHandler"></el-input> -->
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
			list
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
</style>
