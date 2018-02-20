<template>
  <div>
    Sortable Tree View
    <ul>
      <li v-for="(item, index) in value" v-bind:key="index">
        <TreeViewItem v-on:signalEvent="handleSignalEvent" :item="item" :treeIndex="'' + index"></TreeViewItem>
      </li>
      <li>
        <i class="el-icon-plus" @click="addSibling"></i>
      </li>
    </ul>
  </div>
</template>

<script>
import TreeViewItem from '@/components/TreeViewItem'

export default {
  props: ['value'],
  name: 'sortable-tree-view',
  components: {
    TreeViewItem
  },
  data () {
    return {
      eventQueue: [],
      addition: 0,
      nondroppable: 0
    }
  },
  methods: {
    handleSignalEvent (e) {
      this.eventQueue.push(e)
      switch (this.eventQueue[this.eventQueue.length - 1].type) {
        case 'addSibling':
          this.handleAddSibling()
          break
        case 'dragend':
          this.handleForDragnDrop()
          break
        case 'removeItem':
          this.handleRemoveItem()
          break
        case 'drop-modifier-addition':
          this.addition++
          break
        case 'nondroppable':
          this.nondroppable++
          break
        default:
          break
      }
    },
    handleRemoveItem () {
      if (this.eventQueue.length <= 0) return
      let removeEvent = this.eventQueue.pop()
      let index = this.decodeTreeIndex(removeEvent.treeIndex).reverse()
      if (index.length === 1) {
        this.value.splice(index.pop(), 1)
        return
      }
      let parentNode = this.value[index.pop()]
      if (index.length > 0) {
        while (index.length > 1) {
          parentNode = parentNode.children[index.pop()]
        }
        parentNode.children.splice(index.pop(), 1)
      }
      /* console.log(index)
      let lastIndex = index.pop()
      let node = this.value[index.pop()]
      console.log(node)
      while (index.length > 1) {
        lastIndex = index.pop()
        node = node.children[lastIndex]
      }
      let i = index.pop()
      if (i) {
        node.children = node.children.concat(node.children[i].children)
        this.$delete(node.children, i)
      } else {
        console.log(this.value)
        console.log(lastIndex)
        this.$delete(this.value, lastIndex)
      }
      console.log(this.value) */
    },
    addSibling () {
      this.value.push({
        name: 'New Item',
        children: []
      })
    },
    handleAddSibling () {
      if (this.eventQueue.length <= 0) return
      let siblingEvent = this.eventQueue.pop()
      let index = this.decodeTreeIndex(siblingEvent.treeIndex).reverse()
      console.log(index)
      let node = this.value[index.pop()]
      while (index.length > 0) {
        node = node.children[index.pop()]
      }
      node.children.push({
        name: 'New Item',
        children: []
      })
    },
    handleForDragnDrop () {
      if (this.eventQueue.length <= 0) return
      let dragendEvent = this.eventQueue.pop()
      let dropEvent = { type: null }
      while (dropEvent.type !== 'drop' && this.eventQueue.length) {
        dropEvent = this.eventQueue.pop()
      }
      if (dropEvent.type) {
        if (this.nondroppable) {
          this.nondroppable--
          this.handleException()
          return
        }
        if (!this.addition) {
          this.swapNodes(dragendEvent.treeIndex, dropEvent.treeIndex)
        } else {
          this.addition--
          console.log('handle')
          this.removeAndAddNode(dragendEvent.treeIndex, dropEvent.treeIndex)
        }
      }
    },
    handleException () {
      console.log('Move not allowed')
    },
    // radim na ovome trenutno
    removeAndAddNode (firstNodeIndex, secondNodeIndex) {
      if (firstNodeIndex.length < secondNodeIndex.length && secondNodeIndex.substr(0, firstNodeIndex.length) === firstNodeIndex) {
        this.handleException()
        return
      }
      let firstIndex = this.decodeTreeIndex(firstNodeIndex).reverse()
      let secondIndex = this.decodeTreeIndex(secondNodeIndex).reverse()
      let parentNode = this.value
      if (firstIndex.length > 1) {
        parentNode = this.value[firstIndex.pop()]
        while (firstIndex.length > 1) {
          parentNode = parentNode.children[firstIndex.pop()]
        }
      }
      console.log(parentNode)
      let index = firstIndex.pop()
      let currentNode = (parentNode.children) ? parentNode.children[index] : parentNode[index]
      console.log(currentNode)

      let targetNode = this.value[secondIndex.pop()]
      while (secondIndex.length > 0) {
        targetNode = targetNode.children[secondIndex.pop()]
      }

      if (parentNode.children) {
        parentNode.children.splice(index, 1)
      } else {
        parentNode.splice(index, 1)
      }
      console.log('dodje dovdje')

      console.log(targetNode)
      targetNode.children.push(currentNode)
    },
    swapNodes (firstNodeIndex, secondNodeIndex) {
      let firstIndex = this.decodeTreeIndex(firstNodeIndex).reverse()
      let secondIndex = this.decodeTreeIndex(secondNodeIndex).reverse()
      let firstDepth = firstIndex.length
      let secondDepth = secondIndex.length
      let firstNode = this.value[firstIndex.pop()]
      let secondNode = this.value[secondIndex.pop()]
      while (firstIndex.length) {
        firstNode = firstNode.children[firstIndex.pop()]
      }
      while (secondIndex.length) {
        secondNode = secondNode.children[secondIndex.pop()]
      }
      console.log(firstNode.name)
      console.log(secondNode.name)

      let tmp = firstNode.name
      firstNode.name = secondNode.name
      secondNode.name = tmp

      if (firstDepth > secondDepth) {
        secondNode.children = secondNode.children.concat(firstNode.children)
        firstNode.children = []
      } else if (firstDepth < secondDepth) {
        firstNode.children = firstNode.children.concat(secondNode.children)
        secondNode.children = []
      } else {
        tmp = firstNode.children.slice()
        firstNode.children = secondNode.children
        secondNode.children = tmp
      }
      console.log(this.value)
    },
    decodeTreeIndex (index) {
      let next = index.split(':')
      next.forEach(function (elem) {
        elem = parseInt(elem)
      })
      return next
    }
  }
}
</script>

<style scoped>
.el-icon-plus:hover {
  background-color: #CCCCCC;
}
</style>
