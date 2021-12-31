<template>
  <div id="app">
    <div class="header"><img src="./assets/logo.png" /></div>
    <div class="imitate-box">
      <div
        id="imitateInput"
        v-html="content"
        contenteditable
        @input="changeHandler"
        @keydown="keydownHandler"
      ></div>
      <div
        v-if="filteredList.length > 0"
        id="imitateTip"
        class="imitate-tip"
        :style="{ left: tipPos.x + 'px', top: tipPos.y + 'px' }"
      >
        <div class="tip-box">
          <el-radio-group ref="imitateTip" class="is-focus" v-model="picked" @keyup.native.enter="insertName">
            <el-radio-button v-for="item in filteredList" :key="item.name" :label="item.name">
              <img class="title" :src="item.picture" />
              <span>{{ item.name }}</span>
            </el-radio-button>
          </el-radio-group>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    let list = [
        {
          name: "明日香",
          picture: "../src/assets/1.png",
        },
        {
          name: "艾伦",
          picture: "../src/assets/2.png",
        },
        {
          name: "三笠",
          picture: "../src/assets/3.png",
        },
        {
          name: "鲁鲁修",
          picture: "../src/assets/1.png",
        },
        {
          name: "圣罗兰",
          picture: "../src/assets/2.png",
        },
      ]
    return {
      list,
      filteredList: [],
      content: "",
      picked: "",
      tipPos: {
        x: 0,
        y: 0,
      },
      insertFlag: null,
    };
  },
  mounted() {


  },
  methods: {
    changeHandler(e) {
      let content = e.target.innerHTML
      let cursorPos = this.getCursorPos()
      let lastAtPos = content.lastIndexOf('@', cursorPos-1)
      if(lastAtPos < 0) return;
      let filter = content.slice(lastAtPos + 1, cursorPos)
      this.tipPos = this.getTipPos();
      this.filteredList = this.list.filter(item=>item.name.indexOf(filter) > -1)
      this.picked = this.filteredList[0] && this.filteredList[0].name
    },
    keydownHandler(e) {
      if(e.keyCode === 13 && this.filteredList.length > 0) {
        // 插入字符串
        console.log(this.picked)
      }
    },
    getCursorPos() {
      let selection = getSelection();
      let range = selection.getRangeAt(0);
      let cursorPos = range.startOffset;
      return cursorPos;
    },
    getTipPos() {
      // 在光标处插入暂时性元素
      let selection = window.getSelection();
      let range = selection.getRangeAt(0);
      let fakeNode = document.createElement("span");
      fakeNode.innerHTML = "|";
      fakeNode.id = "fakeCursor";
      range.insertNode(fakeNode);

      // 计算暂时性元素屏幕位置
      let rect = fakeNode.getBoundingClientRect();
      let posX = rect.x - 50;
      let posY = rect.y - 155;

      // 删除暂时性元素
      fakeNode.parentNode.removeChild(fakeNode);

      return {
        x: posX,
        y: posY,
      };
    },
    insertName(val) {
      console.log('val1:', this.picked)
    },
    insertPicked() {
      // 将filter置之不理;插入选中符号；不显示提示框
    }
  },
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
#imitateInput {
  width: 300px;
  line-height: 20px;
  max-height: 100px;
  padding: 5px 15px;
  border: 1px solid #dcdfe6;
  border-radius: 4px;
  color: #606266;
  transition: border-color 0.2s cubic-bezier(0.645, 0.045, 0.355, 1);
  font-size: 12px;
  margin: auto;
  overflow-y: auto;
}
#imitateInput:hover {
  border-color: #c0c4cc;
}
#imitateInput:focus {
  outline: none;
  border-color: #409eff;
}
.imitate-tip {
  width: 150px;
  overflow: hidden;
  border-radius: 4px;
  border: 1px solid #dcdfe6;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  position: absolute;
  z-index: 100;
}
.tip-box {
  width: 100%;
  max-height: 168px;
  overflow-y: scroll;
  position: relative;
}
.el-radio-button {
  width: 100%;
}
.el-radio-group {
  border-radius: 4px;
}
.el-radio-button__inner {
  width: 100%;
  border: none !important;
  border-radius: 0 !important;
  height: 28px;
  line-height: 32px;
  padding: 0 10px;
  display: flex;
  align-items: center;
  background: #fff !important;
}
.el-radio-button.is-active .el-radio-button__inner {
  background: #ecedf0 !important;
}
.el-radio-button {
  box-shadow: none !important;
}
.el-radio-button__inner img {
  width: 22px;
  height: 22px;
}
.el-radio-button__inner span {
  margin-left: 6px;
  font-weight: normal;
  font-size: 12px;
  color: #606266 !important;
}
</style>
