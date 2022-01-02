<template>
  <div id="app">
    <div class="header"><img src="./assets/logo.png" /></div>
    <div class="imitate-box">
      <div
        id="imitateInput"
        v-html="content"
        contenteditable
        @input="inputHandler"
        @keydown="keydownHandler"
        @click="clickHandler"
      ></div>
      <div
        v-if="filteredList.length > 0"
        id="imitateTip"
        class="imitate-tip"
        :style="{ left: tipPos.x + 'px', bottom: tipPos.y +  'px' }"
      >
        <div class="tip-arrow"></div>
        <div class="tip-box">
          <el-radio-group ref="imitateTip" class="is-focus" v-model="picked">
            <el-radio-button
              v-for="item in filteredList"
              :key="item.name"
              :label="item.name"
              @click.native="
                (e) => {
                  insertName(e, item.name);
                }
              "
            >
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
let lastEditRange;

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
    ];
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
      cursorPos: "",
    };
  },
  mounted() {},
  methods: {
    inputHandler(e) {

      var selection = getSelection();
      lastEditRange = selection.getRangeAt(0);


      let content = lastEditRange.startContainer.data;

      // CONTENT不存在
      if (!content) {
        this.filteredList = [];
        return;
      }

      let cursorPos = lastEditRange.startOffset;
      let lastAtPos = content.lastIndexOf("@", cursorPos - 1);

      // 没有@
      if (lastAtPos < 0) {
        this.filteredList = [];
        return;
      }

      // 为邮箱格式
      if(lastAtPos > 1 && /[A-Za-z0-9]/.test(content[lastAtPos - 1])) {
        this.filteredList = [];
        return;
      }

      // 节点为div
      if(selection.anchorNode.nodeName === 'DIV') {
        this.filteredList = [];
        return;
      }

      let filter = content.slice(lastAtPos + 1, cursorPos);

      this.tipPos = this.getTipPos(cursorPos);
      this.filteredList = [...this.list.filter(
        (item) => item.name.indexOf(filter) > -1
      )];
      this.picked = this.filteredList[0] && this.filteredList[0].name;
     
    },
    clickHandler(e) {
      var selection = getSelection();
      lastEditRange = selection.getRangeAt(0);
    },
    keydownHandler(e) {
      // 回车选中值插入
      if (e.keyCode === 13 && this.filteredList.length > 0) {
        this.insertName(e, this.picked)
        e.preventDefault();
        e.stopPropagation();
      }
      // 回车换行
      if(e.keyCode === 13 && this.filteredList.length < 1) {
        // console.log('回车换行')
        // let imitateInput = document.getElementById('imitateInput')
        // let range = document.createRange();
        // let selection = getSelection();
        // let offset = selection.focusOffset
        // let content = imitateInput.innerHTML
        // imitateInput.innerHTML = content.slice(0,offset) + '&nbsp;' +  content.slice(offset)
        // range.setStart(imitateInput.childNodes[0], offset + 1);
        // range.collapse(true);
        // selection.removeAllRanges();
        // selection.addRange(range);
        // e.preventDefault();
        // e.stopPropagation();
        // return false
      }
      // 上下键切换选中值 ArrowUp 38 ArrowDown 40
      if (e.keyCode === 38 && this.filteredList.length > 0) {
        let curIndex = this.filteredList.map(item=>item.name).indexOf(this.picked);
        let nextIndex = (curIndex - 1 + this.filteredList.length) % this.filteredList.length
        this.picked = this.filteredList[nextIndex].name
      }
      if (e.keyCode === 40 && this.filteredList.length > 0) {
        let curIndex = this.filteredList.map(item=>item.name).indexOf(this.picked);
        let nextIndex = (curIndex + 1) % this.filteredList.length
        this.picked = this.filteredList[nextIndex].name
      }
    },
    insertName(e, val) {
      if (e.target.tagName === "INPUT") return;
      this.filteredList = [];
      // 在光标处插入选中名称
      let imitateInput = document.getElementById("imitateInput");
      imitateInput.focus();
      var selection = getSelection();
      // 如果保存的有上次的光标对象
      if (lastEditRange) {
        // 清除所有选区
        selection.removeAllRanges();
        // 添加最后光标还原之前的状态
        selection.addRange(lastEditRange);
      }
      let cursorPos = lastEditRange.startOffset
      let cut = selection.anchorOffset - imitateInput.innerHTML.lastIndexOf('@', cursorPos - 1 ) - 1;
      val = val.slice(cut)
      if (selection.anchorNode.nodeName != "#text") {
        var insertText = document.createTextNode(val);
        if (imitateInput.childNodes.length > 0) {
          for (var i = 0; i < imitateInput.childNodes.length; i++) {
            if (i == selection.anchorOffset) {
              imitateInput.insertBefore(insertText, imitateInput.childNodes[i]);
            }
          }
        } else {
          imitateInput.appendChild(insertText);
        }
        // 创建新的光标对象
        var range = document.createRange();
        // 将光标对象的范围界定为新建的表情节点
        range.selectNodeContents(insertText);
        // 定位光标位置在表情节点的最大长度位置
        range.setStart(insertText, insertText.length);

        // 将选区折叠为一个光标
        range.collapse(true);
        // 清除所有光标对象
        selection.removeAllRanges();
        // 添加新的光标对象
        selection.addRange(range);
        // 如果是文本节点
      } else {
        // 获取光标对象
        var range = selection.getRangeAt(0);
        // 获取光标对象的范围界定对象，一般就是textNode对象
        var textNode = range.startContainer;
        // 获取光标位置
        var rangeStartOffset = range.startOffset;
        // 在光标位置处插入
        textNode.insertData(rangeStartOffset, val);
        // 添加了新内容，将光标移动到新的位置
        range.setStart(textNode, rangeStartOffset + val.length);

        // 将选区折叠为一个光标
        range.collapse(true);
        // 清除所有光标对象
        selection.removeAllRanges();
        // 添加新的光标对象
        selection.addRange(range);
      }
      // 记录最后光标对象
      lastEditRange = selection.getRangeAt(0);

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
      let posX = rect.x - 70;
      let posY = document.documentElement.clientHeight - rect.y + 12;

      // 删除暂时性元素
      fakeNode.parentNode.removeChild(fakeNode);

      return {
        x: posX,
        y: posY,
      };
    },
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
  cursor: text;
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
  /* overflow: hidden; */
  border-radius: 4px;
  border: 1px solid #dcdfe6;
  box-shadow: 0 2px 12px 0 rgb(0 0 0 / 10%);
  position: absolute;
}
.tip-box {
  width: 100%;
  max-height: 168px;
  overflow-y: scroll;
  position: relative;
  border-radius: 4px;
  background: #fff;
}

.imitate-tip:before {
  content: '';
  display: block;
  width: 22px;
  height: 1px;
  background: #fff;
  position: absolute;
  bottom: -1px;
  left: 0;
  right: 0;
  margin: auto;
}
.imitate-tip:after {
  content: '';
  display: block;
  width: 12px;
  height: 1px;
  position: absolute;
  bottom: -4px;
  left: calc(50% - 8px);
  border-top: 4px solid #fff;
    border-left: 5px solid transparent;
    border-right: 5px solid transparent; 
    height: 0; 
    width: 5px;
}
.tip-arrow {
    position: absolute;
    width: 0;
    height: 0;
    bottom: -11px;
    left: 0;
    right: 0;
    margin: auto;
    border-left: 7px solid transparent;
    border-right: 6px solid transparent;
    border-top: 12px solid #fff;
}

.tip-arrow:before {
  content: '';
    display: block;
    width: 10px;
    height: 10px;
    border-radius: 0 5px 0 0;
    border: 1px solid #dcdfe6;
    position: absolute;
    bottom: 0px;
    left: -14px;
    transform: skew(30deg, 0deg);
    border-left: none;
    border-bottom: none;
}
.tip-arrow:after {
    content: '';
    display: block;
    width: 10px;
    height: 10px;
    border-radius: 5px 0 0 0;
    position: absolute;
    bottom: 0px;
    right: -13px;
    transform: skew(-30deg, 0deg);
    border: 1px solid #dcdfe6;
    border-right: none;
    border-bottom: none;

}
.el-radio-button {
  width: 100%;
}
.el-radio-group {
  border-radius: 4px;
  width: 100%;
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
