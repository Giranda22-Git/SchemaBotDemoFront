<template>
  <div class="patternsBar_main_wrapper" v-if="$route.query.bot">
    <div class="patterns_wrapper">
      <div
        v-for="pattern in $options.importData.patterns"
        class="pattern"
        :key="pattern.name"
        @click="getNodes($route.query.bot, pattern.getEndPoint, pattern.createAndUpdateEndPoint, pattern.deleteEndPoint)"
      >
        {{ pattern.name }}
      </div>
    </div>
    <div class="nodes_wrapper">
      <div class="node"
        v-for="node in currentNodes"
        :key="node.code._id"
        @click="updateNode(node)"
      >
        {{ node.code.name }}
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'
import importData from '@/settings/importData.json'
import EventBus from '@/components/EventBus.js'
export default {
  name: "patternsBar",
  importData,
  data: () => ({
    currentNodes: []
  }),
  mounted () {
    EventBus.$on('updateNodes', async val => {
      await this.getNodes(val.settings.url, val.settings.getEndPoint, val.settings.createAndUpdateEndPoint, val.settings.deleteEndPoint)
    })
  },
  methods: {
    updateNode (node) {
      EventBus.$emit('newNode', node)
    },
    async getNodes (url, getEndPoint, createAndUpdateEndPoint, deleteEndPoint) {
      try {
        const response = await axios.get(url + getEndPoint)
        // clean array of nodes
        this.currentNodes = []
        for (const tmpNode of response.data) {
          this.currentNodes.push({
            code: tmpNode,
            settings: {
              nodeName: tmpNode.name,
              url,
              getEndPoint,
              createAndUpdateEndPoint,
              deleteEndPoint
            }
          })
        }
        this.currentNodes.push({
          code: {
            _id: 'newNode',
            name: 'Создать новую ноду'
          },
          settings: {
            nodeName: 'Создать новую ноду',
            url,
            getEndPoint,
            createAndUpdateEndPoint,
            deleteEndPoint
          }
        })
      }
      catch (err) {
        alert('error, server is dead\n\n' + err)
        console.log(err)
      }
    }
  }
}
</script>

<style lang="less" scoped>
.patternsBar_main_wrapper {
  width: 450px;
  height: 100vh;
  display: flex;
  border: 1px solid black;
  .patterns_wrapper {
    width: 200px;
    height: 100%;
    overflow: auto;
    border: 1px solid red;
    .pattern {
      width: 100%;
      height: 70px;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      cursor: pointer;
      font-size: 1.2em;
      font-family: sans-serif;
      padding-left: 5%;
    }
    .active {
      background-color: gray;
    }
  }
  .nodes_wrapper {
    width: 250px;
    height: 100%;
    overflow: auto;
    border: 1px solid green;
    .node {
      width: 100%;
      height: 70px;
      display: flex;
      justify-content: flex-start;
      align-items: center;
      cursor: pointer;
      font-size: 1.2em;
      font-family: sans-serif;
      padding-left: 5%;
    }
    .active {
      background-color: gray;
    }
  }
}
</style>