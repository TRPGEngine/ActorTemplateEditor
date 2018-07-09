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
      if (JSON.stringify(data.list) === '[]') {
        this.$message({
          message: '没有要保存的数据',
          type: 'warning'
        })
        return
      }
      const save = () => {
        localStorage.setItem('quickSave', JSON.stringify(data))
        this.$message({
          message: '保存成功',
          type: 'success'
        })
      }
      if (localStorage.getItem('quickSave')) {
        this.$confirm('此操作将覆盖之前的存档, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          save()
        }).catch(() => {
          console.log('取消保存操作')
        })
      } else {
        save()
      }
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

<style lang="scss">
html, body, #app {
  height: 100vh;
  overflow: hidden;
}

#app {
  display: flex;
}

.el-container.is-vertical {
  .el-header {
    box-shadow: 0px 1px 5px #dddddd;
    padding: 10px 20px;
  }

  .el-main {
    overflow: hidden;
    display: flex;
    flex-direction: column;
  }
}
</style>
