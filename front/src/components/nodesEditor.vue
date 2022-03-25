<template>
  <div class="nodesEditor_wrapper">
    <VJsoneditor class="editor" :options="{ mode: 'code' }" v-model="json" :plus="false" @error="onEditorError" />
    <div class="rightContentWrapper">
      <div class="saveButton element"
        @click="saveCurrentNode"
      >
        Сохранить текущую ноду
      </div>

      <div class="saveButton element"
        @click="restartBot"
      >
        Перезапустить бота
      </div>

      <div class="requstsLogs element">
        Requests Logs:
        <div class="log"
          v-for="log in reverseRequestsLogs"
          :key="log.id"
        >
          {{ log.value }}
        </div>
      </div>

      <div class="editorError element">
        <div>
          Last Editor error:
        </div>
        <span>
          {{ editorError }}
        </span>
      </div>

      <div class="saveButton element"
        @click="deleteCurrentNode"
      >
        Удалить текущую ноду
      </div>
    </div>
  </div>
</template>

<script>
import VJsoneditor from 'v-jsoneditor'
import importData from '@/settings/importData.json'
import EventBus from '@/components/EventBus.js'
import axios from 'axios'
export default {
  name: "nodesEditor",
  importData,
  data: () => ({
    json: {},
    currentNodeSettings: {},
    fieldsForClean: [
      '__v',
      '_id'
    ],
    requestsLogs: [],
    editorError: ''
  }),
  mounted () {
    EventBus.$on('newNode', val => {
      this.json = this.cleanJson(val.code)
      this.currentNodeSettings = val.settings
    })
  },
  components: {
    VJsoneditor
  },
  computed: {
    reverseRequestsLogs: function () {
      return this.requestsLogs.reverse()
    }
  },
  methods: {
    onEditorError (err) {
      this.editorError = err
    },
    getRandomArbitrary (min, max) {
      return Math.random() * (max - min) + min
    },
    async restartBot () {
      try {
        const response = await axios.post(this.$route.query.bot + this.$options.importData.mainSettings.restartEndPoint)
        if (response.data === 'OK') {
          this.requestsLogs.push({ value: 'Бот перезапущен', id: this.getRandomArbitrary(1, 100000) })
        }
      }
      catch (err) {
        this.requestsLogs.push({ value: err, id: this.getRandomArbitrary(1, 100000) })
      }
    },
    updateNodes () {
      EventBus.$emit('updateNodes', { settings: this.currentNodeSettings })
    },
    async deleteCurrentNode () {
      try {
        const response = await axios.delete(this.currentNodeSettings.url + this.currentNodeSettings.deleteEndPoint, { data: {name: this.currentNodeSettings.nodeName} })
        this.requestsLogs.push({ value: 'удалено: ' + response.data, id: this.getRandomArbitrary(1, 100000) })
        this.updateNodes()
      } catch (err) {
        this.requestsLogs.push({ value: err, id: this.getRandomArbitrary(1, 100000) })
      }
    },
    async saveCurrentNode () {
      try {
        const response = await axios.post(this.currentNodeSettings.url + this.currentNodeSettings.createAndUpdateEndPoint, this.json)
        if (response.data === 'OK') {
          this.requestsLogs.push({value: 'сохранено', id: this.getRandomArbitrary(1, 100000)})
        }
        this.updateNodes()
      } catch (err) {
        this.requestsLogs.push({ value: err, id: this.getRandomArbitrary(1, 100000) })
      }
    },
    isObject (val) {
      if (
        typeof val === 'object' &&
        !Array.isArray(val) &&
        val !== null
      ) {
        return true
      }
      return false
    },
    cleanJson (jsonObj) {
      for (const key in jsonObj) {
        if (this.fieldsForClean.includes(key) || this.fieldIsEmpty(jsonObj[key])) {
          delete jsonObj[key]
        }
        if (this.isObject(jsonObj[key])) {
          for (const keyon in jsonObj[key]) {
            if (Array.isArray(jsonObj[key][keyon])) {
              for (const [index, iterator] of jsonObj[key][keyon].entries()) {
                jsonObj[key][keyon][index] = this.cleanJson(jsonObj[key][keyon][index])
              }
            }
          }
        }
        if (Array.isArray(jsonObj[key])) {
          for (const [index, iterator] of jsonObj[key].entries()) {
            if (this.isObject(iterator)) {
              for (const keyin in iterator) {
                if (this.fieldsForClean.includes(keyin) || this.fieldIsEmpty(jsonObj[key][index][keyin])) {
                  delete jsonObj[key][index][keyin]
                }
              }
            }
          }
        }
      }
      return jsonObj
    },
    fieldIsEmpty (field) {
      // if field is String or Array
      if (typeof field === 'string' || Array.isArray(field)) {
        return field.length === 0
      }
      // if field is object
      if (this.isObject(field)) {
        return field && Object.keys(field).length === 0 && Object.getPrototypeOf(field) === Object.prototype
      }
    }
  }
}
</script>

<style lang="less" scoped>
.nodesEditor_wrapper {
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: space-evenly;
  align-items: center;
  .editor {
    width: 75% !important;
    height: 98% !important;
    border: 1px solid red;
  }
  .rightContentWrapper {
    width: 20%;
    height: 98%;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
    align-items: center;
    & .element {
      margin-bottom: 15%;
    }
    .saveButton {
      width: 90%;
      height: 40px;
      display: flex;
      justify-content: center;
      align-items: center;
      color: black;
      font-family: sans-serif;
      font-size: 1.2em;
      border: 2px solid black;
      border-radius: 7px;
      cursor: pointer;
      padding: 1%;
      &:hover {
        background-color: rgba(0,0,0,0.1);
      }
    }
    .requstsLogs {
      width: 90%;
      height: 100px;
      overflow: auto;
      border: 2px solid black;
      border-radius: 7px;
      font-size: 1.3em;
      padding-top: .1em;
      padding-left: .5em;
      font-family: sans-serif;
      .log {
        padding-left: .6em;
        font-size: .9em;
      }
    }
    .editorError {
      width: 90%;
      height: 150px;
      overflow: auto;
      border: 2px solid red;
      border-radius: 7px;
      font-size: 1.3em;
      font-family: sans-serif;
      padding-top: .1em;
      padding-left: .5em;
      span {
        padding-left: .6em;
        font-size: .9em;
      }
    }
  }
}
</style>