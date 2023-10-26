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
    </div>
    <button type="button" @click="console.log(editor.getJSON())">获取数据</button>
  </div>
</template>

<script setup>
import { Popup as TPopup } from 'tdesign-vue-next'
import { ref } from 'vue'
import Table from '@tiptap/extension-table'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TableRow from '@tiptap/extension-table-row'
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent } from '@tiptap/vue-3'

const editor = ref(null)
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
  content: ''
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
      const target = e.target && e.target.className
      const isBtn = target.includes('btn-create')
      const isCol = target.includes('col')
      if (isBtn || isCol) return
      console.log(isCol)
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
  const tableData = generateTableData(rows, cols)
  console.log(tableData)
  editor.value.commands.setContent(tableData)
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
    border: 1px solid #f00;
  }
}
</style>
