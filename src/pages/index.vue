/* eslint-disable no-console */
<script setup lang="ts">
import type { Blockstate } from '~/types'
const WIDTH = 10
const HIGHT = 10
const state = ref(
  Array.from({ length: HIGHT }, (_, y) =>
    Array.from(
      { length: WIDTH },
      (_, x): Blockstate => ({
        x,
        y,
        adjacentMines: 0,
        revealed: false,
      }),
    ),
  ),
)

function generaMines(initial: Blockstate) {
  for (const row of state.value) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1)
        continue
      if (Math.abs(initial.y - block.y) <= 1)
        continue
      block.mine = Math.random() < 0.1
    }
  }
  updateNumbers()
}
const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]
const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
  'text-blue-500',
]
function updateNumbers() {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}
function getSiblings(block: Blockstate) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HIGHT)
      return undefined
    return state.value[y2][x2]
  })
    .filter(Boolean) as Blockstate[]
}
let mineGenerated = false
const dev = false
function onRightClick(block: Blockstate) {
  block.flag = !block.flag
}
function onClick(e: MouseEvent, block: Blockstate) {
  console.log(block)
  if (!mineGenerated) {
    generaMines(block)
    mineGenerated = true
  }
  block.revealed = true
  checkGameState()
  if (block.mine)
    console.log('炸了')
  expendZero(block)
}
function expendZero(block: Blockstate) {
  if (block.adjacentMines !== 0)
    return
  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}
function getBlockClass(block: Blockstate) {
  if (block.flag)
    return 'bg-green'
  if (!block.revealed)
    return 'bg-blue'
  return block.mine ? 'text-red' : numberColors[block.adjacentMines]
}
function checkGameState() {
  const blocks = state.value.flat()
  console.log('111', blocks.every((block) => {
    return block.mine ? true : block.revealed
  }))
  if (blocks.every((block) => {
    return block.mine ? true : block.revealed
  }))
    console.log('win')
}
// function init() {
//   mineGenerated = true
// }
updateNumbers()
</script>

<template>
  <div>
    扫雷
    <div p5>
      <div
        v-for="(row, y) in state" :key="y"
        flex="~"
        item-center justify-center
      >
        <button
          v-for="item, x in row"
          :key="x"
          flex="~ gap-1"
          items-center justify-center
          border="1 green-300/50"
          w-10
          h-10
          m="0.5"
          :class="getBlockClass(item)"
          @click="onClick($event,item)"
          @contextmenu.prevent="onRightClick(item)"
        >
          <template v-if="item.flag ">
            <div i-mdi:flag />
          </template>
          <template v-if="item.revealed ||dev">
            <div v-if="item.mine" i-mdi:mine />
            <div v-else>
              {{ item.adjacentMines
              }}
            </div>
          </template>
        </button>
      </div>
    </div>
  </div>
</template>
