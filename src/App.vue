<template>
  <div class="wrapper">
        <pre @drop="drop" @dragover="allowDrop" id="box-droppable">
            Liebe Claudia,
    entschuldige, dass ich mich so lange nicht bei dir gemeldet
    Wie du weißt, bin ich vor zwei Wochen <span class="drag-place" id="drag-place-1" @click="setDragItem">....</span>
            Deswegen hatte ich leider keine Zeit <span class="drag-place" id="drag-place-2"
                                                       @click="setDragItem">....</span>  meine Freunde. Aber deinen
    Vorschlag, mal wieder gemeinsam <span class="drag-place" id="drag-place-3"
                                          @click="setDragItem">....</span>  Tag miteinander zu verbringen, finde ich sehr gut. Mir passt es auch am besten am
    Wochenende. Würde es bei dir schon <span class="drag-place" id="drag-place-4"
                                             @click="setDragItem">....</span>  Samstag gehen? Da habe ich noch nichts vor. Es würde
            <span class="drag-place" id="drag-place-5" @click="setDragItem">....</span>  natürlich freuen, wenn
    du dir bei dieser Gelegenheit auch meine neue Wohnung anschaust. Es ist wunderbar, so viel zu haben. Was meinst du, wir
    uns bei mir treffen und dann unsere Einkaufstour machen? Oder ist es dir <span class="drag-place" id="drag-place-6"
                                                                                   @click="setDragItem">....</span>  , wenn wir zuerst einkaufen und am Abend
    zusammen kochen und essen? Übrigens, können wir dein Auto nehmen? <span class="drag-place" id="drag-place-7"
                                                                            @click="setDragItem">....</span>  steht schon wieder in der Werkstatt. Schreib mir
    doch bald, ob du an diesem Tag Zeit hast. <span class="drag-place" id="drag-place-8"
                                                    @click="setDragItem">....</span>  du keine Zeit hast, finden wir sicher einen anderen Tag.
    Marion

        </pre>

    <div class="items">
          <span class="item word" :id="item.id"
                :class="{'is-disabled':!isEnable(item.id), 'active':transferWordId === item.id}"
                @dragstart="onDragging"
                @click="clickToSelect(item.id)"
                :draggable="isEnable(item.id) && transferWordId !== item.id" v-for="item in wordList" :key="item.id">
              {{
              item.label
            }}
          </span>
    </div>
  </div>

</template>

<script>
import {ref, reactive, onMounted, onBeforeUnmount} from "vue";


export default {
  setup() {

    //State
    const transferWordId = ref(null);
    let dragPlaceIds = reactive([]);
    let draggedWordsIds = ref([]);
    const wordList = [
      {id: 'word1', label: 'umgezogen'},
      {id: 'word2', label: 'mich'},
      {id: 'word3', label: 'einen'},
      {id: 'word4', label: 'nächsten'},
      {id: 'word5', label: 'für'},
      {id: 'word6', label: 'lieber'},
      {id: 'word7', label: 'wollen'},
      {id: 'word8', label: 'Meines'},
    ]

    onMounted(() => {
      document.addEventListener('click', closeItem);
    });

    onBeforeUnmount(() => {
      document.removeEventListener('click', closeItem);
    });


    //Methods
    function isEnable(id) {
      return !draggedWordsIds.value.includes(id);
    }

    function onDragging(ev) {
      ev.dataTransfer.setData("text", ev.target.id);
    }

    function allowDrop(ev) {
      ev.preventDefault();
    }

    function drop(ev) {
      ev.preventDefault();

      if (ev.target.id !== 'box-droppable') {
        ev.target.textContent = '';

        let id = ev.dataTransfer.getData("text");
        dragPlaceIds.push({[id]: ev.target.id})
        const selectedItem = document.getElementById(id)

        //Clone item
        const cloneNode = cloneNodeItem(selectedItem, id)

        //Append Cloned node
        draggedWordsIds.value.push(id)
        ev.target.appendChild(cloneNode);
      }
    }

    function clickToSelect(id) {
      if (transferWordId.value === id) {
        return transferWordId.value = null
      }

      transferWordId.value = id
    }

    function setDragItem(e) {
      if (transferWordId.value) {
        const id = e.target.id
        dragPlaceIds.push({[transferWordId.value]: id})
        const replaceContainer = document.getElementById(id)
        const replaceItem = document.getElementById(transferWordId.value)
        replaceItem.classList.remove('active')

        //Clone item
        const cloneNode = cloneNodeItem(replaceItem, transferWordId.value)

        //Append Cloned node
        draggedWordsIds.value.push(transferWordId.value)
        transferWordId.value = null
        document.getElementById(id).textContent = '';
        replaceContainer.appendChild(cloneNode);
      }
    }

    function cloneNodeItem(selectedItem, id) {
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
    }

    function closeItem(e) {
      if (e.target.id.includes('close')) {
        const closeId = e.target.id
        const wordId = closeId.slice(6)

        dragPlaceIds = dragPlaceIds.filter((item) => {
          if (Object.keys(item)[0] !== wordId) return item
          else {
            document.getElementById(Object.values(item)[0]).textContent = '....'
          }
        })

        draggedWordsIds.value = draggedWordsIds.value.filter((item) => item !== wordId)
      }
    }


    return {
      //State
      transferWordId, dragPlaceIds, draggedWordsIds, wordList,

      //Methods
      isEnable, onDragging, allowDrop, drop, clickToSelect, setDragItem
    };


  }
};
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

.wrapper {
  max-width: 1024px;
  margin: 0px auto;
  padding-bottom: 40px;
}

pre {
  line-height: 40px;
  width: 100%;
}

.word {
  width: 100px;
  margin-right: 50%;
  background: #48978b;
  color: white;
  text-align: center;
  border-radius: 8px;
  padding: 8px;
  cursor: pointer;
  position: relative;
}

.close {
  position: absolute;
  background: #c85674;
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 18px;
  height: 18px;
  top: -5px;
  right: -5px;
  font-size: 11px;
}

.is-disabled {
  background: #d8ecec;
  cursor: not-allowed;
}

.active {
  background: #354035;
  box-shadow: 0 0 9px 3px rgba(0, 0, 0, .5);
}

.items {
  margin-top: 70px;
  width: 100%;
  justify-content: space-between;
  display: grid;
  grid-template-columns: 1fr 1fr 1fr 1fr;
}

.item {
  margin-top: 20px;
}

.drag-place {
  color: #48978b;
  height: 40px;
  display: inline-block;
}
</style>
