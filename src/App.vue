<template>
  <div>
    <h1>Bloops - Level generator</h1>
    <div class="mb-2">
      <label style="display: block">World</label>
      <input type="number" v-model="worldNumber" min="1" />
    </div>
    <div class="mb-2">
      <label style="display: block">Block type</label>
      <select v-model="selectedTypeIndex">
        <option
          v-for="(type, index) in types"
          :value="index"
          :key="type.name"
          >{{ type.name }}</option
        >
      </select>
    </div>
    <div class="mb-2">
      <button @click="clear()">Clear</button>
    </div>
    <div class="grid">
      <div
        v-for="item in items"
        :key="item.id"
        class="item"
        :style="{
          backgroundColor:
            item.active && item.selectedTypeIndex > -1
              ? types[item.selectedTypeIndex].color
              : 'white'
        }"
        @click="toggle(item)"
      >
        {{ item.id }}
      </div>
    </div>
    <div class="m-2">
      <button @click="generate()">Generate</button>
    </div>
    <div>
      <textarea v-model="json" cols="120" rows="10"></textarea>
    </div>
  </div>
</template>

<script>
import { ref } from '@vue/composition-api'

export default {
  setup() {
    const items = ref(
      Array.from({ length: 153 }).map((v, i) => ({
        id: i + 1,
        active: false,
        selectedTypeIndex: -1,
        x: -9 + ((i + 1) % 17 === 0 ? 17 : (i + 1) % 17),
        y: -(-5 + Math.ceil((i + 1) / 17))
      }))
    )
    const types = [
      { name: 'wall', color: 'burlywood' },
      { name: 'character', color: 'red' },
      { name: 'portal', color: 'green' }
    ]
    const worldNumber = ref(1)
    const selectedTypeIndex = ref(0)
    const json = ref('')

    function clear() {
      items.value.forEach(i => {
        i.active = false
        i.selectedTypeIndex = -1
      })
    }

    function toggle(item) {
      item.active = !item.active
      item.selectedTypeIndex = item.active ? selectedTypeIndex.value : -1
    }

    function generate() {
      const character = items.value.find(item =>
        item.selectedTypeIndex > -1
          ? types[item.selectedTypeIndex].name === 'character'
          : false
      )
      const portal = items.value.find(item =>
        item.selectedTypeIndex > -1
          ? types[item.selectedTypeIndex].name === 'portal'
          : false
      )
      const blocs = items.value.filter(item =>
        item.selectedTypeIndex > -1
          ? types[item.selectedTypeIndex].name === 'wall'
          : false
      )
      json.value = JSON.stringify({
        name: 'Niveau',
        world: Number(worldNumber.value),
        character: character && {
          type: 'PERSONNAGE',
          pos_x: character.x,
          pos_y: character.y
        },
        portal: portal && {
          type: 'PORTAIL',
          pos_x: portal.x,
          pos_y: portal.y
        },
        blocs: blocs.map(i => ({
          pos_x: i.x,
          pos_y: i.y,
          type: 'wall'
        }))
      })
    }

    return {
      types,
      worldNumber,
      selectedTypeIndex,
      items,
      clear,
      toggle,
      generate,
      json
    }
  }
}
</script>

<style>
.grid {
  display: grid;
  grid-template-columns: repeat(17, 50px);
  grid-template-rows: repeat(9, 50px);
}

.item {
  border: 1px solid black;
  cursor: pointer;
}

.m-2 {
  margin: 20px 0;
}

.mb-2 {
  margin-bottom: 20px;
}
</style>
