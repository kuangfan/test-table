<template>
  <div class="about">
    <div class="create-table">
      <t-popup
        :visible="visible"
        trigger="click"
        triggerElement="col"
        placement="bottom"
        :showArrow="true"
      >
        <div class="btn-create" @click="onToggle">表格</div>
        <template #content>
          <div class="grid-select" @mouseleave="gridHover({ rows: 0, cols: 0 })">
            <div v-for="(row, i) in createTableArray" :key="i" class="row">
              <div
                v-for="(item, j) in row"
                :key="j"
                :class="['col', {'active': i < currentHoverGrid.rows && j < currentHoverGrid.cols}]"
                @mouseenter="gridHover(item)"
                @click="createTable(item)"
              ></div>
            </div>
          </div>
        </template>
      </t-popup>
    </div>
    <div class="container">
      <editor-content :editor="editor" />
      <div v-if="focused">
        <div class="fixed-menu" @click="hanleMenuClick">
          <div class="menu-item" @click="tableCellMerge">
            <t-tooltip v-if="!isMerged" content="合并单元格">
              <i class="iconfont icon-hebingdanyuange"></i>
            </t-tooltip>
            <t-tooltip v-else content="取消合并单元格">
              <i class="iconfont icon-quxiaohebingdanyuange"></i>
            </t-tooltip>
          </div>
          <div class="menu-item">
            <t-tooltip content="插入列">
              <i class="iconfont icon-charulie"></i>
            </t-tooltip>
          </div>
          <div class="menu-item">
            <t-tooltip content="插入行">
              <i class="iconfont icon-charuhang"></i>
            </t-tooltip>
          </div>
          <div class="menu-item">
            <t-tooltip content="删除列">
              <i class="iconfont icon-shanchulie"></i>
            </t-tooltip>
          </div>
          <div class="menu-item">
            <t-tooltip content="删除行">
              <i class="iconfont icon-shanchuhang"></i>
            </t-tooltip>
          </div>
        </div>
      </div>
    </div>
    <button type="button" @click="console.log(editor.getJSON())">获取数据</button>
  </div>
</template>

<script setup>
import { Popup as TPopup, Tooltip as TTooltip } from 'tdesign-vue-next'
import { ref } from 'vue'
import Table from '@tiptap/extension-table'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TableRow from '@tiptap/extension-table-row'
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent } from '@tiptap/vue-3'

const editor = ref(null)
const focused = ref(false)
const blurTimer = ref(null)
const tableData = ref(null)
editor.value = new Editor({
  extensions: [
    StarterKit,
    Table.configure({
      resizable: true
    }),
    TableRow,
    TableHeader,
    TableCell
  ],
  content: '',
  onFocus ({ editor, event }) {
    // The editor is focused.
    console.log('onFocus', editor, event)
    focused.value = true
  },
  onBlur ({ editor, event }) {
    // The editor isn’t focused anymore.
    console.log('onBlur', editor, event)
    blurTimer.value = setTimeout(() => {
      focused.value = false
      isMerged.value = false
      editor.commands.setContent(tableData.value)
    }, 300)
  }
})

const maxRow = 10
const maxColumn = 10
const generateCreateTableArray = (maxRow, maxColumn) => {
  const array = []
  for (let i = 0; i < maxRow; i++) {
    array.push([])
    for (let j = 0; j < maxColumn; j++) {
      array[i].push({ rows: i + 1, cols: j + 1 })
    }
  }
  return array
}

const createTableArray = generateCreateTableArray(maxRow, maxColumn)

console.log(createTableArray)

const visible = ref(false)
const onToggle = () => {
  visible.value = !visible.value
  if (visible.value) {
    const handleDomClick = (e) => {
      console.log(e.target)
      const target = e.target && e.target.className
      const isBtn = target.includes('btn-create')
      const isCol = target.includes('col')
      if (isBtn || isCol) return
      visible.value = false
      document.removeEventListener('click', handleDomClick)
    }
    document.addEventListener('click', handleDomClick)
  }
}
const currentHoverGrid = ref({
  rows: 0,
  cols: 0
})
const gridHover = (grid) => {
  currentHoverGrid.value = grid
}
const createTable = (grid) => {
  const { rows, cols } = grid
  console.log(grid)
  visible.value = false
  tableData.value = generateTableData(rows, cols)
  editor.value.commands.setContent(tableData.value)
}

const generateTableData = (rows, cols) => {
  const temp = {
    type: 'table',
    content: []
  }
  for (let i = 0; i < rows; i++) {
    temp.content.push({
      type: 'tableRow',
      content: []
    })
    for (let j = 0; j < cols; j++) {
      temp.content[i].content.push(
        {
          type: 'tableCell',
          attrs: {
            colspan: 1,
            rowspan: 1
          },
          content: [{
            type: 'paragraph',
            attrs: {
              indent: 0,
              textAlign: 'left'
            }
          }]
        }
      )
    }
  }
  return temp
}

const isMerged = ref(false)
const hanleMenuClick = () => {
  console.log('hanleMenuClick')
  if (blurTimer.value) {
    clearTimeout(blurTimer.value)
    blurTimer.value = null
  }
  editor.value.chain().focus().run()
}
const tableCellMerge = () => {
  console.log('isMerged', isMerged.value)
  if (isMerged.value) {
    editor.value.chain().splitCell().run()
  } else {
    editor.value.chain().mergeCells().run()
  }
  isMerged.value = !isMerged.value
}
</script>

<style lang="scss" scoped>
.create-table{
  width: 60px;
  margin: 0 auto;
  .btn-create{
    height: 30px;
    line-height: 30px;
    text-align: center;
    border: 1px solid #333;
    border-radius: 5px;
    cursor: pointer;
  }
}
.grid-select{
  width: 215px;
  height: 215px;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  .row{
    display: flex;
    justify-content: space-around;
    .col{
      width: 16px;
      height: 16px;
      box-sizing: border-box;
      border: 1px solid #ccc;
      cursor: pointer;
      &.active{
        background: #ccc;
      }
    }
  }
}
.container{
  width: 500px;
  margin: 100px auto 0;
  :deep(.tiptap table td),
  :deep(.tiptap table th){
    border: 1px solid #bbbfc4;
    // background-color: #eff0f1;
  }
  :deep(.ProseMirror){
    &:focus-visible{
      outline: none;
    }
  }
  position: relative;
}
.fixed-menu{
  display: flex;
  padding: 4px;
  background: #ffffff;
  border: 1px solid #ccc;
  border-radius: 6px;
  box-shadow: 0 0 5px #f1f1f1;
  position: absolute;
  top: -40px;
  left: 0;
  .menu-item{
    width: 26px;
    height: 26px;
    text-align: center;
    line-height: 26px;
    cursor: pointer;
    i{
      font-size: 20px;
    }
  }
}
.bubble-menu {
  display: flex;
  background-color: #0D0D0D;
  padding: 0.2rem;
  border-radius: 0.5rem;

  button {
    border: none;
    background: none;
    color: #FFF;
    font-size: 0.85rem;
    font-weight: 500;
    padding: 0 0.2rem;
    opacity: 0.6;

    &:hover,
    &.is-active {
      opacity: 1;
    }
  }
}

.floating-menu {
  display: flex;
  background-color: #0D0D0D10;
  padding: 0.2rem;
  border-radius: 0.5rem;

  button {
    border: none;
    background: none;
    font-size: 0.85rem;
    font-weight: 500;
    padding: 0 0.2rem;
    opacity: 0.6;

    &:hover,
    &.is-active {
      opacity: 1;
    }
  }
}
</style>
