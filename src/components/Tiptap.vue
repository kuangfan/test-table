<template>
  <div v-if="editor">
    <button @click="setTable">
      insertTable
    </button>
    <button @click="editor.chain().focus().addColumnBefore().run()" :disabled="!editor.can().addColumnBefore()">
      addColumnBefore
    </button>
    <button @click="editor.chain().focus().addColumnAfter().run()" :disabled="!editor.can().addColumnAfter()">
      addColumnAfter
    </button>
    <button @click="editor.chain().focus().deleteColumn().run()" :disabled="!editor.can().deleteColumn()">
      deleteColumn
    </button>
    <button @click="editor.chain().focus().addRowBefore().run()" :disabled="!editor.can().addRowBefore()">
      addRowBefore
    </button>
    <button @click="editor.chain().focus().addRowAfter().run()" :disabled="!editor.can().addRowAfter()">
      addRowAfter
    </button>
    <button @click="editor.chain().focus().deleteRow().run()" :disabled="!editor.can().deleteRow()">
      deleteRow
    </button>
    <button @click="editor.chain().focus().deleteTable().run()" :disabled="!editor.can().deleteTable()">
      deleteTable
    </button>
    <button @click="editor.chain().focus().mergeCells().run()" :disabled="!editor.can().mergeCells()">
      mergeCells
    </button>
    <button @click="editor.chain().focus().splitCell().run()" :disabled="!editor.can().splitCell()">
      splitCell
    </button>
    <button @click="editor.chain().focus().toggleHeaderColumn().run()" :disabled="!editor.can().toggleHeaderColumn()">
      toggleHeaderColumn
    </button>
    <button @click="editor.chain().focus().toggleHeaderRow().run()" :disabled="!editor.can().toggleHeaderRow()">
      toggleHeaderRow
    </button>
    <button @click="editor.chain().focus().toggleHeaderCell().run()" :disabled="!editor.can().toggleHeaderCell()">
      toggleHeaderCell
    </button>
    <button @click="editor.chain().focus().mergeOrSplit().run()" :disabled="!editor.can().mergeOrSplit()">
      mergeOrSplit
    </button>
    <button @click="editor.chain().focus().setCellAttribute('backgroundColor', '#FAF594').run()"
      :disabled="!editor.can().setCellAttribute('backgroundColor', '#FAF594')">
      setCellAttribute
    </button>
    <button @click="editor.chain().focus().setCellAttribute('contentEditable', false).run()"
      :disabled="!editor.can().setCellAttribute('contentEditable', false)">
      setContentEditable
    </button>
    <button @click="editor.chain().focus().fixTables().run()" :disabled="!editor.can().fixTables()">
      fixTables
    </button>
    <button @click="editor.chain().focus().goToNextCell().run()" :disabled="!editor.can().goToNextCell()">
      goToNextCell
    </button>
    <button @click="editor.chain().focus().goToPreviousCell().run()" :disabled="!editor.can().goToPreviousCell()">
      goToPreviousCell
    </button>
  </div>
  <editor-content :editor="editor" />

  <button type="button" @click="console.log(editor.getJSON())">获取数据</button>
</template>

<script>
import Table from '@tiptap/extension-table'
import TableCell from '@tiptap/extension-table-cell'
import TableHeader from '@tiptap/extension-table-header'
import TableRow from '@tiptap/extension-table-row'
import StarterKit from '@tiptap/starter-kit'
import { Editor, EditorContent } from '@tiptap/vue-3'

const CustomTableCell = TableCell.extend({
  addAttributes () {
    return {
      // extend the existing attributes …
      ...this.parent?.(),

      // and add a new one …
      backgroundColor: {
        default: null,
        parseHTML: element => element.getAttribute('data-background-color'),
        renderHTML: attributes => {
          return {
            'data-background-color': attributes.backgroundColor,
            style: `background-color: ${attributes.backgroundColor}`
          }
        }
      },

      contentEditable: {
        default: true,
        parseHTML: element => element.getAttribute('contenteditable'),
        renderHTML: attributes => {
          return {
            contenteditable: attributes.contentEditable,
            style: attributes.contentEditable ? '' : 'background-color: #ccc'
          }
        }
      }
    }
  }
})

export default {
  components: {
    EditorContent
  },

  data () {
    return {
      editor: null
    }
  },

  mounted () {
    this.editor = new Editor({
      extensions: [
        StarterKit,
        Table.configure({
          resizable: true
        }),
        TableRow,
        TableHeader,
        // Default TableCell
        // TableCell,
        // Custom TableCell with backgroundColor attribute
        CustomTableCell
      ],
      content: `
        <h3>
          Have you seen our tables? They are amazing!
        </h3>
        <ul>
          <li>tables with rows, cells and headers (optional)</li>
          <li>support for <code>colgroup</code> and <code>rowspan</code></li>
          <li>and even resizable columns (optional)</li>
        </ul>
        <p>
          Here is an example:
        </p>
        <table>
          <tbody>
            <tr>
              <th>Name</th>
              <th colspan="3">Description</th>
            </tr>
            <tr>
              <td>Cyndi Lauper</td>
              <td>singer</td>
              <td>songwriter</td>
              <td>actress</td>
            </tr>
            <tr>
              <td>Marie Curie</td>
              <td>scientist</td>
              <td>chemist</td>
              <td>physicist</td>
            </tr>
            <tr>
              <td>Indira Gandhi</td>
              <td>prime minister</td>
              <td colspan="2">politician</td>
            </tr>
          </tbody>
        </table>
      `
    })
  },

  methods: {
    safeJSONParse (str, defaultValue = {}) {
      if (typeof str === 'object') return str

      try {
        return JSON.parse(str)
      } catch (e) {
        return defaultValue
      }
    },
    setTable () {
      const c = {
        default: {
          type: 'doc',
          content: [{
            type: 'title',
            attrs: {
              cover: ''
            }
          }, {
            type: 'table',
            content: [{
              type: 'tableRow',
              content: [{
                type: 'tableHeader',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableHeader',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableHeader',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableHeader',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }]
            }, {
              type: 'tableRow',
              content: [{
                type: 'tableHeader',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableCell',
                attrs: {
                  colspan: 1,
                  rowspan: 2,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableCell',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableCell',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }]
            }, {
              type: 'tableRow',
              content: [{
                type: 'tableHeader',
                attrs: {
                  colspan: 1,
                  rowspan: 1,
                  colwidth: [100]
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  }
                }]
              }, {
                type: 'tableCell',
                attrs: {
                  colspan: 2,
                  rowspan: 1,
                  colwidth: [50, 50],
                  contentEditable: false
                },
                content: [{
                  type: 'paragraph',
                  attrs: {
                    indent: 0,
                    textAlign: 'left'
                  },
                  content: [
                    { type: 'text', text: '123' }
                  ]
                }]
              }]
            }]
          }, {
            type: 'paragraph',
            editable: true,
            attrs: {
              indent: 0,
              textAlign: 'left'
            }
          }]
        }
      }
      const json = c.default || c
      this.editor.commands.setContent(json.content[1])
    }
  },

  beforeUnmount () {
    this.editor.destroy()
  }
}
</script>

<style lang="scss">
/* Basic editor styles */
.tiptap {
  margin: 1rem 0;

  >*+* {
    margin-top: 0.75em;
  }

  ul,
  ol {
    padding: 0 1rem;
  }

  h1,
  h2,
  h3,
  h4,
  h5,
  h6 {
    line-height: 1.1;
  }

  code {
    background-color: rgba(#616161, 0.1);
    color: #616161;
  }

  pre {
    background: #0D0D0D;
    color: #FFF;
    font-family: 'JetBrainsMono', monospace;
    padding: 0.75rem 1rem;
    border-radius: 0.5rem;

    code {
      color: inherit;
      padding: 0;
      background: none;
      font-size: 0.8rem;
    }
  }

  img {
    max-width: 100%;
    height: auto;
  }

  blockquote {
    padding-left: 1rem;
    border-left: 2px solid rgba(#0D0D0D, 0.1);
  }

  hr {
    border: none;
    border-top: 2px solid rgba(#0D0D0D, 0.1);
    margin: 2rem 0;
  }
}

/* Table-specific styling */
.tiptap {
  table {
    border-collapse: collapse;
    table-layout: fixed;
    width: 100%;
    margin: 0;
    overflow: hidden;

    td,
    th {
      min-width: 1em;
      border: 2px solid #ced4da;
      padding: 3px 5px;
      vertical-align: top;
      box-sizing: border-box;
      position: relative;

      >* {
        margin-bottom: 0;
      }
    }

    th {
      font-weight: bold;
      text-align: left;
      background-color: #f1f3f5;
    }

    .selectedCell:after {
      z-index: 2;
      position: absolute;
      content: "";
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      background: rgba(200, 200, 255, 0.4);
      pointer-events: none;
    }

    .column-resize-handle {
      position: absolute;
      right: -2px;
      top: 0;
      bottom: -2px;
      width: 4px;
      background-color: #adf;
      pointer-events: none;
    }

    p {
      margin: 0;
    }
  }
}

.tableWrapper {
  overflow-x: auto;
}

.resize-cursor {
  cursor: ew-resize;
  cursor: col-resize;
}</style>
