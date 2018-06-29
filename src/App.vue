<template>
  <div id="app">
    <el-container>
      <el-header>
        <Header @onSave="onSave" @onLoad="onLoad"/>
      </el-header>
      <el-main>
        <Editor ref="editor"/>
      </el-main>
    </el-container>
  </div>
</template>

<script>
import Header from './layout/Header'
import Editor from './layout/Editor'

export default {
  name: 'App',
  components: {
    Header,
    Editor
  },
  methods: {
    onSave () {
      let editor = this.$refs.editor
      if (!editor) {
        this.$notify({
          title: '保存失败',
          message: '无法正确获取编辑器'
        })
      }

      let data = {
        list: editor.list,
        autoIncrement: editor.autoIncrement
      }
      localStorage.setItem('quickSave', JSON.stringify(data))
      this.$message({
        message: '保存成功',
        type: 'success'
      })
    },
    onLoad () {
      let editor = this.$refs.editor
      if (!editor) {
        this.$notify({
          title: '保存失败',
          message: '无法正确获取编辑器'
        })
      }

      let data = localStorage.getItem('quickSave')
      data = JSON.parse(data) || {}
      editor.autoIncrement = data.autoIncrement || 1
      editor.list = data.list || []
      editor.editingNodeData = null
      this.$message({
        message: '读取成功',
        type: 'success'
      })
    }
  }
}
</script>

<style>
.el-header {
  box-shadow: 0px 1px 5px #dddddd;
  padding: 10px 20px;
}
</style>
