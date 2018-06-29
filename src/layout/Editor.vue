<template>
  <el-row :gutter="20" class="editor-main">
    <el-col :span="12">
      <div class="editor-tree-action">
        <el-button type="primary" @click="addNode">增加属性</el-button>
        <el-button type="primary" @click="addGroup">增加组</el-button>
      </div>
      <el-tree
        :data="list"
        node-key="id"
        default-expand-all
        :expand-on-click-node="false"
        @node-click="handleClick"
        draggable
        :allow-drop="allowDrop">
        <span class="template-node" slot-scope="{ node, data }">
          <i class="iconfont" v-if="data.isGroup">&#xe62f;</i>
          <i class="iconfont" v-else>&#xe6e9;</i>
          <span>{{data.info.name}}</span>
        </span>
      </el-tree>
    </el-col>
    <el-col :span="12">
      <el-form ref="form" :model="editingNodeData" label-width="80px" v-if="editingNodeData">
        <template v-if="editingNodeData.isGroup">
          <!-- 组 -->
          <el-form-item label="组名称">
            <el-input v-model="editingNodeData.info.name"></el-input>
          </el-form-item>
        </template>
        <template v-else>
          <!-- 属性 -->
          <el-form-item label="属性名称">
            <el-input v-model="editingNodeData.info.name"></el-input>
          </el-form-item>
        </template>
      </el-form>
      <div class="tip" v-else>
        请选择您要修改的节点
      </div>
    </el-col>
  </el-row>
</template>

<script>
import at from 'trpg-actor-template'

export default {
  data () {
    return {
      list: [],
      autoIncrement: 1,
      editingNodeData: null
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
    addGroup () {
      this.list.push({
        id: this.autoIncrement,
        isGroup: true,
        info: at.getInitGroup('新组' + this.autoIncrement),
        children: []
      })
      this.autoIncrement++
    },
    handleClick (data, node, ev) {
      console.log('点击了节点')
      this.editingNodeData = data
    },
    allowDrop (draggingNode, dropNode, type) {
      return dropNode.data.isGroup || type !== 'inner' // 仅能拖拽到组里
    }
  }
}
</script>

<style lang="scss">
.editor-main {
  .editor-tree-action {
    margin-bottom: 20px;
  }

  .tip {
    text-align: center;
    margin-top: 120px;
    color: #999999;
  }

  .template-node {
    padding: 0 4px;
  }

  .el-tree-node.is-drop-inner>.el-tree-node__content .template-node {
    background-color: #409eff;
    color: white;
  }
}

</style>