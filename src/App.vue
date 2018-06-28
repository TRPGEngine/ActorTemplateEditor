<template>
  <div id="app">
    <el-container>
      <el-header>
        <Header/>
      </el-header>
      <el-main>
        <Editor/>
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
  }
}
</script>

<style>
.el-header {
  box-shadow: 0px 1px 5px #dddddd;
  padding: 10px 20px;
}
</style>
