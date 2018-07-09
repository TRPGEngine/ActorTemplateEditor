<template>
  <el-row :gutter="20" class="editor-main">
    <!-- 左面板 -->
    <el-col :span="24">
      <div class="editor-tree-action">
        <el-button type="primary" @click="addNode">增加属性</el-button>
        <el-button type="primary" @click="addGroup">增加组</el-button>
        <el-dropdown @command="handleTestCommand">
          <el-button type="primary">
            模板测试<i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item :command="execDice">投骰生成</el-dropdown-item>
            <el-dropdown-item :command="execTest">运算测试</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
        <el-button type="primary" @click="showExportDialog">导出模板</el-button>
        <el-button type="primary" @click="copyCell">复制元素</el-button>
      </div>
    </el-col>
    <el-col :span="12">
      <el-tree
        :data="list"
        node-key="id"
        default-expand-all
        :expand-on-click-node="false"
        @node-click="handleNodeClick"
        draggable
        :allow-drop="allowDrop">
        <span class="template-node" slot-scope="{ node, data }">
          <span>
            <i class="iconfont" v-if="data.isGroup">&#xe62f;</i>
            <i class="iconfont" v-else>&#xe6e9;</i>
            <span
              :style="{
                fontWeight: editingNodeData && data.id === editingNodeData.id ? 'bold' : 'normal',
                color: data.info.value === '数据错误' ? 'red' : 'black'
              }"
              :title="data.info.desc">
              <template v-if="data.isGroup">
                {{data.info.name}}
              </template>
              <template v-else>
                {{data.info.name}} - {{data.info.func}} - {{data.info.default}}
              </template>
            </span>
          </span>
          <el-popover
            placement="right"
            width="200"
            class="template-node-btn"
            v-model="templateNodePopover[data.id]">
            <p v-if="data.isGroup">确定移除组[{{data.info.name}}]么?该组下面所有属性都会被删除</p>
            <p v-else>确定移除属性[{{data.info.name}}]么?</p>
            <div style="text-align: right; margin: 0">
              <el-button type="primary" size="mini" @click="(e) => removeNode(node, data)">确定</el-button>
            </div>
            <el-button slot="reference" class="template-node-action" size="mini" type="text" @click="(e) => handleShowRemoveNodeConfirm(e, data.id)">删除</el-button>
          </el-popover>
        </span>
      </el-tree>
    </el-col>

    <!-- 右面板 -->
    <el-col :span="12">
      <el-form :model="editingNodeData" label-width="80px" v-if="editingNodeData" class="edit-node-panel">
        <template v-if="editingNodeData.isGroup">
          <!-- 组 -->
          <el-form-item label="组名称:">
            <el-input v-model="editingNodeData.info.name"></el-input>
          </el-form-item>
          <el-form-item label="组描述:">
            <el-input
              type="textarea"
              :autosize="{ minRows: 4, maxRows: 6}"
              placeholder="请输入组描述信息"
              v-model="editingNodeData.info.desc">
            </el-input>
          </el-form-item>
        </template>
        <template v-else>
          <!-- 属性 -->
          <el-form-item label="可见性:">
            <el-checkbox v-model="editingNodeData.info.visibility">所有人可见</el-checkbox>
          </el-form-item>
          <el-form-item label="属性名称:">
            <el-input v-model="editingNodeData.info.name"></el-input>
          </el-form-item>
          <el-form-item label="属性描述:">
            <el-input
              type="textarea"
              :autosize="{ minRows: 4, maxRows: 6}"
              placeholder="请输入属性描述信息"
              v-model="editingNodeData.info.desc">
            </el-input>
          </el-form-item>
          <el-form-item label="方法:">
            <el-select v-model="editingNodeData.info.func" placeholder="请选择方法">
              <el-option
                v-for="item in funcOptions"
                :key="item.value"
                :label="`${item.label}(${item.value})`"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="类型:">
            <el-select v-model="editingNodeData.info.type" placeholder="请选择类型">
              <el-option
                v-for="item in typeOptions"
                :key="item.value"
                :label="`${item.label}(${item.value})`"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item :label="editingNodeData.info.func === 'value' ? '默认值:' : '表达式:'">
            <el-input :type="editingNodeData.info.type" v-model="editingNodeData.info.default"></el-input>
          </el-form-item>
          <el-form-item label="测试值:">
            <template v-if="editingNodeData.info.func !== 'enum'">
              <el-input
                :type="editingNodeData.info.type"
                v-model="editingNodeData.info.value"
                :disabled="['expression', 'dice'].indexOf(editingNodeData.info.func) >= 0"
              ></el-input>
            </template>
            <template v-else>
              <el-select v-model="editingNodeData.info.value" placeholder="请选择">
                <el-option
                  v-for="item in (editingNodeData.info.default.split(',') || [])"
                  :key="item"
                  :label="item"
                  :value="item">
                </el-option>
              </el-select>
            </template>
          </el-form-item>
          <el-form-item label="额外参数:">
            <el-checkbox v-model="editingNodeData.info.showExt">显示额外参数</el-checkbox>
          </el-form-item>
          <el-form-item v-if="editingNodeData.info.showExt">
            <!-- 额外参数 -->
            <p>额外变量 <el-button type="text" @click="addEditingNodeDataParameter">新增</el-button></p>
            <div v-for="(item, index) in editingNodeData.info.ext.parameter" :key="index + editingNodeData.info.name" class="ext-parameter">
              <el-input v-model="editingNodeData.info.ext.parameter[index]['name']" placeholder="变量名"></el-input>
              :
              <el-input v-model="editingNodeData.info.ext.parameter[index]['value']" placeholder="测试值"></el-input>
            </div>
          </el-form-item>
        </template>
      </el-form>
      <div class="tip" v-else>
        请选择您要修改的节点
      </div>
    </el-col>

    <!-- 对话框 -->
    <el-dialog title="导出模板" :visible.sync="dialogExportShow">
      <el-form :model="templateInfo" label-width="100px" ref="dialogForm" :rules="dialogExportRules">
        <el-form-item label="模板名称:" prop="name">
          <el-input v-model="templateInfo.name"></el-input>
        </el-form-item>
        <el-form-item label="作者:" prop="creator">
          <el-input v-model="templateInfo.creator"></el-input>
        </el-form-item>
        <el-form-item label="模板描述:">
          <el-input
            type="textarea"
            :autosize="{ minRows: 4, maxRows: 6}"
            placeholder="请输入模板描述信息"
            v-model="templateInfo.desc">
          </el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="generateTemplateStr">生成模板信息</el-button>
        </el-form-item>
      </el-form>
      <el-input
        type="textarea"
        rows="10"
        placeholder="导出信息"
        v-model="exportString">
      </el-input>
    </el-dialog>
  </el-row>
</template>

<script>
import at from 'trpg-actor-template'

export default {
  data () {
    return {
      dialogExportShow: false,
      dialogExportRules: {
        name: [
          { required: true, message: '请输入模板名', trigger: 'blur' }
        ],
        creator: [
          { required: true, message: '请输入模板作者', trigger: 'blur' }
        ]
      },
      funcOptions: [
        {
          label: '值类型',
          value: 'value'
        },
        {
          label: '表达式',
          value: 'expression'
        },
        {
          label: '多选项',
          value: 'enum'
        },
        {
          label: '投骰生成',
          value: 'dice'
        }
      ],
      typeOptions: [
        {
          label: '文本',
          value: 'text'
        },
        {
          label: '多行文本',
          value: 'textarea'
        },
        {
          label: '数字',
          value: 'number'
        }
      ],
      templateNodePopover: [],
      exportString: '',
      list: [],
      autoIncrement: 1,
      editingNodeData: null,
      templateInfo: {
        name: '',
        creator: '',
        desc: ''
      }
    }
  },
  methods: {
    addNode () {
      this.list.push({
        id: this.autoIncrement,
        isGroup: false,
        info: at.getInitCell('新属性' + this.autoIncrement)
      })
      this.autoIncrement++
    },
    removeNode (node, data) {
      const parent = node.parent
      const children = parent.data.children || parent.data
      const index = children.findIndex(d => d.id === data.id)
      children.splice(index, 1)

      // 检测正在编辑节点的信息, 并销毁
      if (this.editingNodeData && data.id === this.editingNodeData.id) {
        this.editingNodeData = null
      }
    },
    addGroup () {
      this.list.push({
        id: this.autoIncrement,
        isGroup: true,
        info: at.getInitGroup('新组' + this.autoIncrement),
        children: []
      })
      this.autoIncrement++
    },
    handleShowRemoveNodeConfirm (e, id) {
      e.stopPropagation()
      this.templateNodePopover.fill(false)
      this.templateNodePopover[id] = true
    },
    handleNodeClick (data, node, ev) {
      console.log('点击了节点')
      this.editingNodeData = data
    },
    allowDrop (draggingNode, dropNode, type) {
      return dropNode.data.isGroup || type !== 'inner' // 仅能拖拽到组里
    },
    addEditingNodeDataParameter () {
      if (!this.editingNodeData.info.ext.parameter) {
        this.$set(this.editingNodeData.info.ext, 'parameter', [])
      }

      this.editingNodeData.info.ext.parameter.push({
        name: '',
        value: ''
      })
    },
    showExportDialog () {
      this.dialogExportShow = true
    },
    generateTemplateStr () {
      this.$refs.dialogForm.validate(valid => {
        if (!valid) {
          return
        }

        const template = this.getTemplate(this.templateInfo.name, this.templateInfo.creator, this.templateInfo.desc)
        console.log('导出的模板:', template)

        this.exportString = at.stringify(template)
      })
    },
    getTemplate (name, creator, desc) {
      const template = at.getInitTemplate()
      template.name = name
      template.creator = creator
      template.desc = desc

      for (const item of this.list) {
        if (item.isGroup) {
          let group = this.convertGroup(item.info, item.children)
          template.insertGroup(group)
        } else {
          let cell = this.convertCell(item.info)
          template.insertCell(cell)
        }
      }

      return template
    },
    convertCell (cellItem) {
      // 将编辑器的属性转化为模板的属性
      const cell = at.getInitCell()
      cell.visibility = cellItem.visibility
      cell.name = cellItem.name
      cell.desc = cellItem.desc
      cell.default = cellItem.default
      cell.func = cellItem.func
      cell.type = cellItem.type
      cell.value = cellItem.value

      cell.ext = {}
      // 解析parameter
      if (cellItem.ext.parameter) {
        let parameter = {}
        cellItem.ext.parameter.map(p => {
          parameter[p.name] = p.value || 0
        })
        cell.ext.parameter = parameter
      }

      return cell
    },
    convertGroup (groupItem, childrenItem) {
      // 将编辑器的组转化为模板的组
      const group = at.getInitGroup()
      group.name = groupItem.name
      group.desc = groupItem.desc

      for (const c of childrenItem) {
        if (c.isGroup) {
          group.insertGroup(this.convertGroup(c.info, c.children))
        } else {
          group.insertCell(this.convertCell(c.info))
        }
      }

      return group
    },
    handleTestCommand (func) {
      if (typeof func === 'function') {
        func()
      }
    },
    execTest () {
      const template = this.getTemplate()
      template.eval()
      const data = template.getData()
      for (const item of this.list) {
        if (!item.isGroup) {
          item.info.value = data[item.info.name]
        } else {
          this.convertTemplateGroupValueToList(data, item)
        }
      }

      this.$forceUpdate()
      console.log('运算测试完毕')
      this.$message({
        message: '运算测试完毕',
        type: 'success'
      })
    },
    execDice () {
      const template = this.getTemplate()
      template.generateRollResult()
      const data = template.getData()
      for (const item of this.list) {
        if (!item.isGroup) {
          item.info.value = data[item.info.name]
        } else {
          this.convertTemplateGroupValueToList(data, item)
        }
      }

      this.$forceUpdate()
      console.log('投骰生成完毕')
      this.$message({
        message: '投骰生成完毕',
        type: 'success'
      })
    },
    convertTemplateGroupValueToList (data, item) {
      for (const sub of item.children) {
        if (!sub.isGroup) {
          sub.info.value = data[sub.info.name]
        } else {
          this.convertTemplateGroupValueToList(data, sub)
        }
      }
    },
    copyCell () {
      if (!this.editingNodeData) {
        this.$message({
          message: '没有选中任何元素',
          type: 'warning'
        })
        return
      }

      if (this.editingNodeData.isGroup) {
        this.$message({
          message: '不能复制组',
          type: 'warning'
        })
        return
      }

      this.list.push({
        id: this.autoIncrement,
        isGroup: false,
        info: JSON.parse(JSON.stringify(this.editingNodeData.info))
      })
      this.autoIncrement++
    }
  }
}
</script>

<style lang="scss">
.editor-main {
  .editor-tree-action {
    margin-bottom: 20px;

    .el-button+.el-button {
      margin-left: 0;
    }
  }

  .tip {
    text-align: center;
    margin-top: 80px;
    color: #999999;
  }

  .template-node {
    padding: 0 4px;
    flex: 1;
    display: flex;
    justify-content: space-between;
    align-items: center;

    .template-node-btn {
      position: absolute;
      right: 0;
    }

    .template-node-action {
      display: none;
    }

    &:hover .template-node-action {
      display: inline;
    }
  }

  .el-tree-node.is-drop-inner>.el-tree-node__content .template-node {
    background-color: #409eff;
    color: white;
  }

  .edit-node-panel {
    p {
      margin: 0;
    }

    .ext-parameter {
      margin-bottom: 10px;

      .el-input {
        width: 180px;
      }
    }
  }
}

</style>
