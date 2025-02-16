<template>
  <div class="map-actions">
    <n-button type="error" ghost :disabled="!roomHasEnemies()" @click="cmd('battle')">[<span class="bold-yellow">B</span>]attle</n-button>
    <n-button type="warning" ghost :disabled="!roomHasResources()" @click="cmd('harvest')">[<span class="bold-yellow">H</span>]arvest</n-button>
    <n-button type="success" ghost :disabled="!roomHasChest()" @click="cmd('loot')">[<span class="bold-yellow">L</span>]oot</n-button>
  </div>
</template>

<script setup>
import { onMounted, watch, ref } from 'vue'

import { NButton } from 'naive-ui'

import { useKeyHandler } from '@/composables/key_handler'
import { useWebSocket } from '@/composables/web_socket'
import { state } from '@/composables/state'

const { onKeydown, keyState } = useKeyHandler()
const { cmd, fetchEntity, fetchItem } = useWebSocket()

const roomItems = ref([])
const roomEntities = ref([])

onKeydown((ev) => {
  if (keyState.alt || keyState.ctrl) {
    return false
  }

  if (state.mode == 'input') {
    return false
  }

  if (ev.key == 'B' || ev.key == 'b') {
    cmd('battle')
    return true
  } else if (ev.key == 'H' || ev.key == 'h') {
    cmd('harvest')
    return true
  } else if (ev.key == 'L' || ev.key == 'l') {
    cmd('loot')
    return true
  }

  return false
})

async function getRoomEntities () {
  let entities = []

  for (let eid of state.gameState.room.entities) {
    try {
      let entity = await fetchEntity(eid)
      if (!entity.error) {
        entities.push(entity)
      }
    } catch (err) {
      console.log(`failed to fetch entity eid ${eid}`)
      console.log(err.stack)
    }
  }

  return entities
}

function roomHasEnemies () {
  return roomEntities.value.find(en => en.traits.includes('evil'))
}

async function getRoomItems () {
  let items = []
  for (let iid of state.gameState.room.items) {
    try {
      let item = await fetchItem(iid)
      if (!item.error) {
        items.push(item)
      }
    } catch (err) {
      console.log(`failed to fetch item iid ${iid}`)
      console.log(err.stack)
    }
  }
  return items
}

function roomHasResources () {
  return roomItems.value.find(en => en.type == 'resource' && en.subtype != 'chest')
}

function roomHasChest () {
  return roomItems.value.find(en => en.type == 'resource' && en.subtype == 'chest')
}

watch(() => state.gameState.room.entities, async () => {
  roomEntities.value = await getRoomEntities()
})

watch(() => state.gameState.room.items, async () => {
  roomItems.value = await getRoomItems()
})

onMounted(() => {
  let ids = ['bottom-left', 'bottom-right']
  for (let id of ids) {
    let el = document.getElementById(id)
    if (el) {
      let containers = el.getElementsByClassName('n-layout-sider-scroll-container')
      for (let container of containers) {
        setTimeout(() => {
          container.scrollTo(0, container.scrollHeight)
        })
      }
    }
  }
})

</script>

<style lang="less">
.map-actions {
  display: flex;
  flex-direction: row;
  justify-content: space-around;
  flex-wrap: wrap;
  margin-top: 5px;
  margin-bottom: 5px;
  width: 100%;
  .n-button {
    margin: 1px;
  }
}
</style>
