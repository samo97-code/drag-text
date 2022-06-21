<template>
  <div class="text-block">
    {{dragPlaceId}}
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
      <span class="item word" :id="item.id" :class="{'is-disabled':!isEnable(item.id), 'active':transferId === item.id}"
            @dragstart="onDragging"
            @dragend="onEnd"
            @click="clickToSelect(item.id)"
            :draggable="isEnable(item.id) && transferId !== item.id" v-for="item in wordList" :key="item.id">{{
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
      transferId: null,
      dragPlaceId: [],
      draggedItemsId: [],
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
        return !this.draggedItemsId.includes(id)
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
    drag(ev) {
      ev.dataTransfer.setData("text", ev.target.id);
    },
    onEnd(){
      // const wordId = ev.target.id
      // const copyWord = document.getElementById(`copy-${wordId}`)
      // this.dragPlaceId.push({[wordId]: copyWord.parentNode.id })
    },
    drop(ev) {
      ev.preventDefault();
      let id = ev.dataTransfer.getData("text");

      ev.target.textContent = '';
      this.dragPlaceId.push({[id]: ev.target.id })

      //Clone item
      const selectedItem = document.getElementById(id)
      let cloneNode = selectedItem.cloneNode(true)
      cloneNode.setAttribute('id', `copy-${id}`)
      cloneNode.setAttribute('draggable', false)


      //Cloned item add close word
      const node = document.createElement("span");
      const textnode = document.createTextNode("X");
      node.appendChild(textnode);
      node.setAttribute('class', 'close')
      node.setAttribute('id', `close-${id}`)
      cloneNode.appendChild(node)


      //Append Cloned node
      this.draggedItemsId.push(id)
      ev.target.appendChild(cloneNode);
    },

    clickToSelect(id) {
      if (this.transferId === id){
        return  this.transferId = null
      }

      this.transferId = id
    },

    setDragItem(id) {
      if (this.transferId) {
        this.dragPlaceId.push({[this.transferId]: id })
        const replaceContainer = document.getElementById(id)
        const replaceItem = document.getElementById(this.transferId)
        replaceItem.classList.remove('active')

        //Clone item
        let cloneNode = replaceItem.cloneNode(true)
        cloneNode.setAttribute('id', `copy-${this.transferId}`)
        cloneNode.setAttribute('draggable', false)


        //Cloned item add close word
        const node = document.createElement("span");
        const textnode = document.createTextNode("X");
        node.appendChild(textnode);
        node.setAttribute('class', 'close')
        node.setAttribute('id', `close-${this.transferId}`)
        cloneNode.appendChild(node)


        //Append Cloned node
        this.draggedItemsId.push(this.transferId)
        this.transferId = null
        document.getElementById(id).textContent = '';
        replaceContainer.appendChild(cloneNode);
      }
    },

    closeItem(e) {
      if (e.target.id.includes('close')) {
        const closeId = e.target.id
        const wordId = closeId.slice(6)

        this.dragPlaceId = this.dragPlaceId.filter((item)=> {
          if (Object.keys(item)[0] !== wordId) return item
          else {
            document.getElementById(Object.values(item)[0]).textContent = '....'
          }
        })
        this.draggedItemsId = this.draggedItemsId.filter((item) => item !== wordId)
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
  margin-top: 150px;
}

.drag-place {
  height: 40px;
  display: inline-block;
}
</style>
