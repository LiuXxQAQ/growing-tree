<script setup lang="ts">
import type { Ref } from 'vue'
import { computed, onMounted, onUnmounted, ref } from 'vue'

enum Direction {
  Left,
  Right,
  Up,
  Down,
}

interface Position {
  x: number
  y: number
}

function gridStyle(position: Position) {
  return {
    'grid-column': position.x,
    'grid-row': position.y,
  }
}

const gameStart = ref(false)
const gridSize = 40
const gameSpeedDelay = 100
const currentScore = ref(0)
const score = computed(() => addZero(currentScore))
const currentHighScore = ref(0)
const highScore = computed(() => addZero(currentHighScore))
const snake: Ref<Position[]> = ref([{ x: 20, y: 20 }])

let food: Position = generateFood()
let direction: Direction = Direction.Right
let gameInterval: number

function startGame() {
  gameStart.value = true
  gameInterval = setInterval(() => {
    move()
    updateScore()
    checkCollision()
  }, gameSpeedDelay)
}

function move() {
  const head = { ...snake.value[0] }
  switch (direction) {
    case Direction.Up:
      head.y--
      break
    case Direction.Down:
      head.y++
      break
    case Direction.Left:
      head.x--
      break
    case Direction.Right:
      head.x++
      break
  }
  snake.value.unshift(head)

  if (food.x === head.x && food.y === head.y)
    food = generateFood()
  else
    snake.value.pop()
}

function addZero(n: Ref<number>, maxLength: number = 3) {
  return n.value.toString().padStart(maxLength, '0')
}

function generateFood(): Position {
  return {
    x: Math.floor(Math.random() * 40 + 1),
    y: Math.floor(Math.random() * 40 + 1),
  }
}

function handleKeydown(event: KeyboardEvent) {
  if (
    !gameStart.value && event.key === ' '
        || !gameStart.value && event.code === 'Space'
  ) {
    startGame()
    return
  }
  switch (event.key) {
    case 'w':
    case 'W':
    case 'ArrowUp':
      if (direction !== Direction.Down)
        direction = Direction.Up
      break

    case 's':
    case 'S':
    case 'ArrowDown':
      if (direction !== Direction.Up)
        direction = Direction.Down
      break

    case 'a':
    case 'A':
    case 'ArrowLeft':
      if (direction !== Direction.Right)
        direction = Direction.Left
      break

    case 'd':
    case 'D':
    case 'ArrowRight':
      if (direction !== Direction.Left)
        direction = Direction.Right
      break

    default:
      break
  }
}

function checkCollision() {
  const head = snake.value[0]
  if (head.x < 1 || head.y < 1 || head.y > gridSize || head.x > gridSize)
    resetGame()

  for (let i = 1; i < snake.value.length; i++) {
    const item = snake.value[i]
    if (head.x === item.x && head.y === item.y)
      resetGame()
  }
}

function resetGame() {
  gameStart.value = false
  clearInterval(gameInterval)
  direction = Direction.Left
  snake.value = [{ x: 20, y: 20 }]
  food = generateFood()
  updateHightScore()
  updateScore()
}

function updateScore() {
  currentScore.value = snake.value.length - 1
}

function updateHightScore() {
  if (currentHighScore.value < currentScore.value)
    currentHighScore.value = currentScore.value
}

function addKeydownListener() {
  document.addEventListener('keydown', handleKeydown)
}

function removeEventListener() {
  document.removeEventListener('keydown', handleKeydown)
}

onMounted(() => {
  addKeydownListener()
})

onUnmounted(() => {
  clearInterval(gameInterval)
  removeEventListener()
})
</script>

<template>
  <div>
    <div class="score">
      <div>Score: {{ score }}</div>
      <div>High Score: {{ highScore }}</div>
    </div>
    <div class="border-1">
      <div class="border-2">
        <div class="main">
          <div class="grid">
            <div v-if="gameStart" class="food" :style="gridStyle(food)" />
            <template v-if="gameStart">
              <div
                v-for="(item, key) in snake"

                :key="key"
                class="snake" :style="gridStyle(item)"
              />
            </template>
          </div>
          <div v-if="!gameStart" class="hint">
            Press Spacebar To Start The Game
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.score {
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: large;
    letter-spacing: 1px;
    font-family: 'Robot';
}

.border-1 {
    border: 12px solid seagreen;
    border-radius: 12px;
    box-shadow: inset 0 0 0px 5px seagreen;
}

.border-2 {
    border: 25px solid darkgreen;
    border-radius: 10px;
}

.main {
    position: relative;
}

.grid {
    display: grid;
    grid-template-columns: repeat(40, 10px);
    grid-template-rows: repeat(40, 10px);
    background-color: beige;
}

.snake {
    background-color: black;
}

.food {
    background-color: slategrey;
}

.hint {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 1.7rem;
    letter-spacing: .2rem;
    font-family: 'Robot';
}
</style>
