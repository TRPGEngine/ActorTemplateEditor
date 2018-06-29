<template>
  <el-row class="editor-header">
    <el-col :sm="12" :xs="24">
      <h1>TRPG进阶模板在线编辑器</h1>
    </el-col>
    <el-col :sm="6" :offset="6" hidden-sm-and-down>
      <el-button type="text" @click="load">读取</el-button>
      <el-button type="text" @click="save">保存</el-button>
      <el-button type="text" @click="backToApp">返回应用</el-button>
    </el-col>
  </el-row>
</template>

<script>
export default {
  props: ['onSave'],
  methods: {
    backToApp () {
      window.open('https://trpg.moonrailgun.com')
    },
    save () {
      this.$emit('onSave')
    },
    load () {
      this.$emit('onLoad')
    }
  }
}
</script>

<style lang="scss">
.editor-header {
  text-align: right;

  h1 {
    margin: 0;
    line-height: 40px;
    text-align: left;
  }
}
</style>
