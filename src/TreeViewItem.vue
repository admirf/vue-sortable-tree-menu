<template>
  <div class="dragItem" :id="item.name">
    <div v-if="flag" draggable='true' @dragleave="dragLeave" @dragstart="dragStart" @dragover="dragOver" @drop="drop" @dragend="dragEnd" @dblclick="toggle">
      <span class="dragLabel">{{ item.name }}</span>
      <i @click="addChild" class="el-icon-plus" @drop="dropInto"></i>
      <i @click="removeChild" class="el-icon-minus" @drop="nondroppable"></i>
    </div>
    <div v-else>
      <input ref="inputField" @keyup="changeField" type="text" :placeholder="item.name" v-model="item.name" />
    </div>
    <ul>
      <li v-for="(model, index) in item.children" v-bind:key="index">
        <tree-view-item v-on:signalEvent="handleSignalEvent" :item="model" :treeIndex="concatenate(treeIndex, index)"></tree-view-item>
      </li>
      <div v-if="this.item.children.length > 0">
        <li>
          <i @click="addSibling" class="el-icon-plus" draggable="false"></i>
        </li>
      </div>
    </ul>
  </div>
</template>

<script>
export default {
  props: ['item', 'treeIndex'],
  name: 'tree-view-item',
  data () {
    return {
      flag: true
    }
  },
  methods: {
    nondroppable () {
      this.emitSignal('nondroppable')
    },
    changeField (e) {
      if (e.code === 'Enter') this.flag = !this.flag
    },
    dragStart (e) {
      if (!this.flag) e.preventDefault()
    },
    addSibling () {
      this.emitSignal('addSibling')
    },
    addChild () {
      this.item.children.push({
        name: 'New Item',
        children: []
      })
    },
    removeChild () {
      this.emitSignal('removeItem')
    },
    emitSignal (str) {
      this.$emit('signalEvent', {
        type: str,
        item: this.item,
        treeIndex: this.treeIndex
      })
    },
    concatenate (a, b) {
      return `${a}:${b}`
    },
    toggle (e) {
      e.preventDefault()
      e.stopPropagation()
      if (this.flag) this.flag = !this.flag
    },
    dragOver (e) {
      e.preventDefault()
      e.target.style = 'background-color: lightgray;'
    },
    dragLeave (e) {
      e.preventDefault()
      e.target.style = 'background-color: none'
    },
    drop (e) {
      e.stopPropagation()
      e.preventDefault()
      e.target.style = 'background-color: none'
      this.emitSignal('drop')
    },
    dropInto (e) {
      e.preventDefault()
      this.emitSignal('drop-modifier-addition')
    },
    dragEnd (e) {
      e.stopPropagation()
      e.preventDefault()
      this.emitSignal('dragend')
    },
    handleSignalEvent (e) {
      this.$emit('signalEvent', e)
    }
  }
}
</script>

<style scoped>
  [draggable] {
    user-select: none;
  }
  .el-icon-plus:hover {
    background-color: #CCCCCC;
  }
  .el-icon-minus:hover {
    background-color: #CCCCCC;
  }
  .dragLabel:hover {
    cursor: move;
  }
</style>
