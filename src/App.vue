<script setup>
import { ref, watch } from 'vue'

let counter = 1
const colors = [
  '#9c88ff',
  '#487eb0',
  '#00a8ff',
  '#2a9bd5',
  '#5bb8e8',
  '#e75936',
  '#ef7d60',
  '#ffbe76',
  '#badc58',
  '#e056fd',
  '#6368d0',
  '#5d63f6',
  '#e15f41',
  '#c44569',
  '#d26080',
  '#f5cd79',
  '#f19066',
  '#c44569',
  '#f65784',
]
const userWin = ref(false)
const activeTile = ref(null)
const matrix = ref(generateMatrix())
const tiles = ref(generateTiles())

watch(() => tiles.value.length, (hasTiles) => {
  if (!hasTiles) {
    userWin.value = verify()
  }
})

function verify() {
  const numbers = Array.from({ length: 9 }).map((_, idx) => idx + 1) // [1, 2, ..., 9]
  let [previousRow, previousCol] = [null, null]

  for (const numb of numbers) {
    const { key } = getValue(numb)
    const [x, y] = key.split('x') // '1x2' -> ['1', '2']
    const [row, col] = [+x, +y] // [1, 2]

    if (previousCol !== null && previousRow !== null) {
      const c1 = previousRow > row ? (previousRow - row) > 1 : (row - previousRow) > 1
      const c2 = previousCol > col ? (previousCol - col) > 1 : (col - previousCol) > 1

      if (c1 || c2) {
        return false
      }
    }

    [previousRow, previousCol] = [row, col]
  }

  return true
}

function getValue(number) {
  for (const row of matrix.value) {
    for (const col of row) {
      if (col.value === number) {
        return {...col}
      }
    }
  }
}

function shuffle(min, max) {
  return Math.floor(Math.random() * (max - min) + min)
}

function generateTiles() {
  return Array.from({ length: 9 }).map((_, idx) => {
    const row = Math.floor(idx / 3) + 1
    const col = idx % 3 === 0 ? ((counter = 1) && counter) : ++counter
    const key = `${row}x${col}` // 1x1, 1x2, ... 3x3

    return {
      key: `tile-${key}`,
      bg: colors[shuffle(0, 18)],
      value: idx + 1,
    }
  })
}

function generateMatrix() {
  return [
    [
      { key: '1x1', bg: 'white', value: null },
      { key: '1x2', bg: 'white', value: null },
      { key: '1x3', bg: 'white', value: null },
    ],
    [
      { key: '2x1', bg: 'white', value: null },
      { key: '2x2', bg: 'white', value: null },
      { key: '2x3', bg: 'white', value: null },
    ],
    [
      { key: '3x1', bg: 'white', value: null },
      { key: '3x2', bg: 'white', value: null },
      { key: '3x3', bg: 'white', value: null },
    ],
  ]
}

function selectTile(event, tile) {
  const selectedTile = document.querySelector('.tile.active')

  if (selectedTile) {
    selectedTile.classList.remove('active')
  }

  event.target.classList.add('active')
  activeTile.value = tile
}

function selectBox(event, value) {
  const box = event.target
  if (!activeTile.value || box.classList.contains('active')) {
    return
  }

  const [x, y] = value.key.split('x') // '1x1' -> ['1', '1']
  const [row, col] = [+x - 1, +y - 1] // [0, 0]
  const tilesCopy = [...tiles.value]
  const activeTileIndex = tilesCopy.findIndex(({ key }) => activeTile.value.key === key)

  matrix.value[row][col].bg = activeTile.value.bg
  matrix.value[row][col].value = activeTile.value.value
  box.classList.add('active')

  tilesCopy.splice(activeTileIndex, 1)
  tiles.value = tilesCopy
  activeTile.value = null
}

function reset() {
  const activeBoxes = document.querySelectorAll('.box.active')
  activeBoxes.forEach((box) => {
    box.classList.remove('active')
  })
  matrix.value = generateMatrix()
  tiles.value = generateTiles()
  userWin.value = false
}
</script>

<template>
  <div class="container">
    <nav class="navbar navbar-light navbar-expand">
      <div class="navbar-brand fw-regular">123 Game</div>
      <ul class="navbar-nav ms-auto">
        <button type="button" class="btn btn-primary btn-sm fw-medium" @click="reset">
          New Game
        </button>
      </ul>
    </nav>
    <div class="row">
      <div class="col-md-6 col-12">
        <div class="tiles">
          <div
            v-for="tile in tiles"
            :key="tile.key"
            class="tile me-4 mb-2"
            :style="{ backgroundColor: tile.bg }"
            @click="selectTile($event, tile)"
          >
            {{ tile.value }}
          </div>
        </div>
        <div v-if="tiles.length" class="fw-regular">{{ tiles.length }} moves left.</div>
        <div v-else>
          <div class="text-heading" v-if="userWin">
            💐<br />You win
          </div>
          <span v-else class="fw-regular">Game finished, You lose.</span>
        </div>
      </div>
      <div class="col-md-6 col-12">
        <div class="boxes">
          <template v-for="(row, row_index) in matrix" :key="`row_${row_index}`">
            <div
              v-for="col in row"
              :key="col.key"
              :class="['box', `box-${col.key}`]"
              :style="{ backgroundColor: col.bg }"
              @click="selectBox($event, col)"
            >
              {{ col.value }}
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}
.boxes {
  display: grid;
  grid-template-columns: repeat(3, 100px);
  gap: 30px;
  padding-bottom: 30px;
}
.tile {
  display: inline-block;
  width: 60px;
  height: 60px;
  font-size: 30px;
  font-weight: 500;
  line-height: 1;
  text-align: center;
  border-radius: 2px;
  padding: 12px;
  cursor: pointer;
  border: 4px solid transparent;
}
.tile.active {
  border: 4px dashed #5b6565;
}
.box {
  width: 100px;
  height: 100px;
  border-radius: 4px;
  font-size: 40px;
  font-weight: 500;
  line-height: 1;
  text-align: center;
  padding: 30px;
  transition: all 250ms ease-in;
  box-shadow: 0 2px 8px rgba(35, 35, 35, 0.4);
  border: 4px solid transparent;
}
.box:hover:not(.active) {
  border: 4px dashed #abd7d7;
}
.text-heading {
  font-weight: 400;
  font-size: 5rem;
  text-align: center;
  line-height: 1.2;
}
.fw-regular {
  font-weight: 500 !important;
}
.fw-medium {
  font-weight: 600 !important;
}
</style>
