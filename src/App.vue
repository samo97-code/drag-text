<template>
  <div class="wrapper">
    <pre @drop="drop" @dragover="allowDrop" id="box-droppable">
      Lorem Ipsum is simply dummy text of
      the printing and <span class="drag-place" id="drag-place-1" @click="setDragItem('drag-place-1')">....</span>  industry. Lorem Ipsum has been the industry's
      standard dummy text ever since the 1500s,
      when an unknown printer took a galley of type and scrambled it to <span class="drag-place" id="drag-place-2"
                                                                              @click="setDragItem('drag-place-2')">....</span> specimen book.
      It <span class="drag-place" id="drag-place-3" @click="setDragItem('drag-place-3')">....</span> survived not only five centuries, but also the leap into electronic typesetting,
      remaining essentially unchanged.
    </pre>

    <div class="items">
      <span class="item word" :id="item.id" :class="{'is-disabled':!isEnable(item.id), 'active':transferWordId === item.id}"
            @dragstart="onDragging"
            @click="clickToSelect(item.id)"
            :draggable="isEnable(item.id) && transferWordId !== item.id" v-for="item in wordList" :key="item.id">{{
          item.label
        }}
      </span>
    </div>
  </div>

</template>

<script>

export default {
  name: 'App',
  data() {
    return {
      transferWordId: null,
      dragPlaceIds: [],
      draggedWordsIds: [],
      wordList: [
        {id: 'word1', label: 'First Item'},
        {id: 'word2', label: 'Second Item'},
        {id: 'word3', label: 'Third Item'},
      ]
    }
  },

  computed: {
    isEnable() {
      return (id) => {
        return !this.draggedWordsIds.includes(id)
      }
    }
  },

  mounted() {
    document.addEventListener('click', this.closeItem);
  },
  beforeUnmount() {
    document.removeEventListener('click', this.closeItem);
  },

  methods: {
    onDragging(ev) {
      ev.dataTransfer.setData("text", ev.target.id);
    },
    allowDrop(ev) {
      ev.preventDefault();
    },
    drop(ev) {
      ev.preventDefault();

      if (ev.target.id !== 'box-droppable') {
        ev.target.textContent = '';

        let id = ev.dataTransfer.getData("text");
        this.dragPlaceIds.push({[id]: ev.target.id })
        const selectedItem = document.getElementById(id)

        //Clone item
        const cloneNode = this.cloneNodeItem(selectedItem,id)

        //Append Cloned node
        this.draggedWordsIds.push(id)
        ev.target.appendChild(cloneNode);
      }
    },

    clickToSelect(id) {
      if (this.transferWordId === id){
        return  this.transferWordId = null
      }

      this.transferWordId = id
    },

    setDragItem(id) {
      if (this.transferWordId) {
        this.dragPlaceIds.push({[this.transferWordId]: id })
        const replaceContainer = document.getElementById(id)
        const replaceItem = document.getElementById(this.transferWordId)
        replaceItem.classList.remove('active')

        //Clone item
        const cloneNode = this.cloneNodeItem(replaceItem,this.transferWordId)

        //Append Cloned node
        this.draggedWordsIds.push(this.transferWordId)
        this.transferWordId = null
        document.getElementById(id).textContent = '';
        replaceContainer.appendChild(cloneNode);
      }
    },

    cloneNodeItem(selectedItem,id){
      //Clone node item
      let cloneNode = selectedItem.cloneNode(true)
      cloneNode.setAttribute('id', `copy-${id}`)
      cloneNode.setAttribute('draggable', false)


      //In Cloned item add close btn
      const node = document.createElement("span");
      const textnode = document.createTextNode("X");
      node.appendChild(textnode);
      node.setAttribute('class', 'close')
      node.setAttribute('id', `close-${id}`)
      cloneNode.appendChild(node)

      return cloneNode
    },

    closeItem(e) {
      if (e.target.id.includes('close')) {
        const closeId = e.target.id
        const wordId = closeId.slice(6)

        this.dragPlaceIds = this.dragPlaceIds.filter((item)=> {
          if (Object.keys(item)[0] !== wordId) return item
          else {
            document.getElementById(Object.values(item)[0]).textContent = '....'
          }
        })

        this.draggedWordsIds = this.draggedWordsIds.filter((item) => item !== wordId)
      }
    }
  }

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  /*text-align: center;*/
  color: #2c3e50;
  margin-top: 60px;
}

pre {
  line-height: 40px;
}

.word {
  background: green;
  color: white;
  border-radius: 8px;
  padding: 8px;
  cursor: pointer;
  position: relative;
}

.close {
  position: absolute;
  background: red;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 15px;
  height: 15px;
  top: -5px;
  right: -5px;
}

.is-disabled {
  background: #87b787;
  cursor: not-allowed;
}

.active {
  background: #354035;
  box-shadow: 0 0 9px 3px rgba(0, 0, 0, .5);
}

.items {
  margin-top: 70px;
  width: 300px;
  display: flex;
  justify-content: space-between;
}

.drag-place {
  height: 40px;
  display: inline-block;
}
</style>
