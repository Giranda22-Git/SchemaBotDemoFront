<template>
  <div class="home">
    <patterns-bar/>
    <nodes-editor/>
  </div>
</template>

<script>
import patternsBar from '@/components/patternsBar'
import NodesEditor from '@/components/nodesEditor.vue'
export default {
  name: 'HomeView',
  components: {
    patternsBar,
    NodesEditor
  }
}
</script>

<style lang="less" scoped>
  .home {
    width: 100vw;
    height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
  }
</style>
