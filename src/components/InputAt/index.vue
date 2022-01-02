<template>
  <div class="imitate-box">
    <div
      id="inputEdit"
      contenteditable
      @input="inputHandler"
      @keydown="keydownHandler"
      @click="clickHandler"
    ></div>
    <div
      v-if="filteredList.length > 0"
      id="inputTip"
      class="input-tip"
      :style="{ left: tipPos.x + 'px', bottom: tipPos.y + 'px' }"
    >
      <div class="tip-box">
        <el-radio-group v-model="picked">
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
      <div class="tip-arrow"></div>
    </div>
  </div>
</template>

<script>
let lastEditRange;

export default {
    name: 'InputAt',
	props: {
        list: {
            type: Array,
            default: []
        },
		autoDelete: {
			type: Boolean,
			default: false
		}
    },
    data() {
		return {
			filteredList: [],
			content: "",
			picked: "",
			tipPos: {
				x: 0,
				y: 0
			},
			cursorPos: ""
		}
	},
  	methods: {

		// input事件：根据条件唤起提示
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

		// click事件：记录光标位置
		clickHandler(e) {
			var selection = getSelection();
			lastEditRange = selection.getRangeAt(0);
		},

		// keydown事件：响应回车事件、上下键切换选中值
		keydownHandler(e) {
			// 回车选中值插入
			if (e.keyCode === 13 && this.filteredList.length > 0) {
				this.insertName(e, this.picked)
				e.preventDefault();
				e.stopPropagation();
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

			// 删除人员后的空格，自动删除@人员，Backspace 8
			if (e.keyCode === 8 && this.autoDelete) {

				try {
					var selection = getSelection();
					lastEditRange = selection.getRangeAt(0);

					var textNode = lastEditRange.startContainer;

					let cursorPos = lastEditRange.startOffset;
					let lastAtPos = textNode.data ? textNode.data.lastIndexOf("@", cursorPos - 1) : -1;
					let count = cursorPos - lastAtPos;

					let slice = textNode.data.slice(lastAtPos + 1, cursorPos - 1);
					if(this.list.map(item=>item.name).indexOf(slice) > -1) {
						// 先移光标

						lastEditRange.setStart(textNode, lastAtPos);
						lastEditRange.collapse(true);
						selection.removeAllRanges();
						selection.addRange(lastEditRange);

						// 再删除
						console.log('textNode:', textNode)
						console.log('lastAtPos:', lastAtPos)
						console.log('count:', count)
						textNode.deleteData(lastAtPos, count)
						e.preventDefault();
						e.stopPropagation();
					}
					
				} catch (error) {
					
				}

			}
		},

		// 在光标处插入文本节点
		insertName(e, val) {
			if (e.target.tagName === "INPUT") return;
			this.filteredList = [];
			let inputEdit = document.getElementById("inputEdit");

			// 还原上次光标对象
			inputEdit.focus();
			var selection = getSelection();
			if (lastEditRange) {
				selection.removeAllRanges();
				selection.addRange(lastEditRange);
			}

			// 获取插入字段
			let cursorPos = lastEditRange.startOffset
			let cut = selection.anchorOffset - inputEdit.innerHTML.lastIndexOf('@', cursorPos - 1 ) - 1;
			val = val.slice(cut) + ' '

			// 根据节点类型插入字段
			if (selection.anchorNode.nodeName != "#text") {
				// 非文本节点
				// 创建并插入节点
				var insertText = document.createTextNode(val);
				if (inputEdit.childNodes.length > 0) {
				for (var i = 0; i < inputEdit.childNodes.length; i++) {
					if (i == selection.anchorOffset) {
					inputEdit.insertBefore(insertText, inputEdit.childNodes[i]);
					}
				}
				} else {
				inputEdit.appendChild(insertText);
				}
				// 创建新的光标对象
				var range = document.createRange();
				range.selectNodeContents(insertText);
				range.setStart(insertText, insertText.length);
				range.collapse(true);
				selection.removeAllRanges();
				selection.addRange(range);
			} else {
				// 如果是文本节点
				// 在光标处插入文本
				var range = selection.getRangeAt(0);
				var textNode = range.startContainer;
				var rangeStartOffset = range.startOffset;
				textNode.insertData(rangeStartOffset, val);

				// 修正光标位置
				range.setStart(textNode, rangeStartOffset + val.length);
				range.collapse(true);
				selection.removeAllRanges();
				selection.addRange(range);
			}

			// 记录最后光标对象
			lastEditRange = selection.getRangeAt(0);

		},
		
		// 获取提示框位置
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
#inputEdit {
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
  white-space: pre-wrap;
  word-wrap: break-word; 
}
#inputEdit:hover {
  border-color: #c0c4cc;
}
#inputEdit:focus {
  outline: none;
  border-color: #409eff;
}
.input-tip {
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

.input-tip:before {
  content: "";
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
.input-tip:after {
  content: "";
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
  content: "";
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
  content: "";
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
