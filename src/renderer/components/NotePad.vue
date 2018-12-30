<template>
  <div class="grid-container">
    <div class="sidebar">
      <div class="memos-title">
        <font-awesome-icon icon="sticky-note"/>
      </div>
      <div class="memos">
        <div
          v-for="(memo, memoIndex) in memos"
          :key="memoIndex"
          @click="changeIndex(memoIndex)"
          :class="['memo', index === memoIndex ? 'active' : '']"
        >{{ memo.filePath && memo.filePath.split('/').reverse()[0] || memo.text.split('\n')[0].substr(0,5) || "Untitled" }}</div>
      </div>
    </div>
    <div class="method">
      <div @click="addNew" class="method-item">
        <font-awesome-icon icon="plus"/>
      </div>
      <div @click="delMemo" class="method-item">
        <font-awesome-icon icon="minus"/>
      </div>
    </div>
    <div class="editor">
      <input ref v-model="tags[index]" placeholder="tag1 tag2..." class="tagarea">
      <hr>
      <textarea
        ref="editor"
        v-model="memos[index].text"
        placeholder="TextArea"
        class="textarea"
        @keydown.meta.83="save"
        @keyup.ctrl.83="save"
        autofocus
      ></textarea>
    </div>
  </div>
</template>

<script>
const { dialog } = require("electron").remote
const fs = require("fs")

const newMemo = { text: "", filePath: "" }
const newTag = ""

export default {
  data: () => {
    return {
      memos: [{ text: "", filePath: "" }],
      tags: [""],
      index: 0
    }
  },
  methods: {
    initStorage (){
      this.memos = JSON.parse(localStorage.getItem("memos")) || []
      this.tags = JSON.parse(localStorage.getItem("tags")) || []
    },
    setStorage (){
      localStorage.setItem("memos", JSON.stringify(this.memos))
      localStorage.setItem("tags", JSON.stringify(this.tags))
    },
    addNew (){
      this.memos.push(Object.assign({}, newMemo))
      this.tags.push(newTag)
      this.changeIndex(this.memos.length - 1)
      this.setStorage()
    },
    changeIndex (index){
      this.index = index
      this.$refs.editor.focus()
    },
    delMemo (){
      if (this.index !== 0) {
        this.memos.splice(this.index, 1)
        this.tags.splice(this.index, 1)
        this.index = this.index - 1
      } else {
        this.memos = [{ text: "", filePath: "" }]
        this.tags = [""]
      }
      this.setStorage()
    },
    save (){
      const memo = this.memos[this.index]
      const tag = this.tags[this.index]
      if (!memo.filePath) {
        const filePath = dialog.showSaveDialog({ title: "Save as" })
        memo.filePath = filePath
      }
      fs.writeFileSync(memo.filePath, memo.text)
      fs.writeFileSync(memo.filePath, tag)
      this.setStorage()
    }
  }
}
</script>

<style scoped>
.grid-container {
  display: grid;
  grid-template-columns: 150px 1fr;
  grid-template-rows: 1fr 60px;
  grid-template-areas:
    "sidebar editor"
    "method editor";
}

.sidebar {
  grid-area: sidebar;
  background-color: #4863ad;
  color: #ffffff;
  max-height: calc(100vh - 60px);
  overflow: auto;
}

.method {
  grid-area: method;
  background-color: #4863ad;
  color: #ffffff;
}

.editor {
  grid-area: editor;
  height: 100vh;
}

.textarea {
  padding: 8px 8px;
  width: 100%;
  height: calc(100vh - 33px);
  resize: none;
  border: none;
  font-size: 18px;
  line-height: 1.2em;
}

.tagarea {
  padding: 0px 8px;
  width: 100%;
  height: 25px;
  resize: none;
  border: none;
  font-size: 15px;
}

.memos {
  overflow-y: auto;
}

.memos-title {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 40px;
  font-size: 18px;
  color: #ffffff;
}

.memo {
  display: flex;
  justify-content: center;
  align-items: center;
  color: #ffffff;
  cursor: pointer;
}
.memo:hover {
  background-color: #49821a;
}
.memo.active {
  background-color: #c9d6bc;
}

.method-title {
  padding: 8px 16px;
}

.method-item {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 30px;
  font-size: 22px;
  color: #ffffff;
  cursor: pointer;
}
.method-item:hover {
  background-color: #1a3682;
}
</style>
