<script setup lang="ts">
import { onMounted, ref } from 'vue'

interface Point {
  x: number
  y: number
}

interface Branch {
  startPoint: Point
  angel: number
  length: number
}

const WIDTH = 600
const HEIGHT = 600

const el = ref<HTMLCanvasElement>()
let ctx: CanvasRenderingContext2D
const animateId = ref<number | null>(null)

const pendingTokens: Function[] = []

const isDrawing = ref(true)

function init() {
  ctx = el.value!.getContext('2d')!
}

function line(pointA: Point, pointB: Point) {
  ctx.beginPath()
  ctx.moveTo(pointA.x, pointA.y)
  ctx.lineTo(pointB.x, pointB.y)
  ctx.stroke()
}

function getEndPoint(b: Branch): Point {
  return {
    x: b.startPoint.x + Math.cos(b.angel) * b.length,
    y: b.startPoint.y - Math.sin(b.angel) * b.length,
  }
}

function step(b: Branch, depth: number = 1) {
  const endPoint = getEndPoint(b)
  line(b.startPoint, endPoint)

  if (depth <= 4 || Math.random() < 0.4) {
    pendingTokens.push(() => step({
      startPoint: endPoint,
      angel: b.angel + Math.random() * Math.PI / 4 - Math.PI / 8,
      length: b.length * (0.7 + Math.random() * 0.3),
    }, depth + 0.5))
  }

  if (depth <= 4 || Math.random() < 0.4) {
    pendingTokens.push(() => step({
      startPoint: endPoint,
      angel: b.angel - Math.random() * Math.PI / 4 + Math.PI / 8,
      length: b.length * (0.7 + Math.random() * 0.3),
    }, depth + 0.6))
  }
}

function frame() {
  const tasks = [...pendingTokens]
  pendingTokens.length = 0
  tasks.forEach(task => task())
}

let i = 0

function startFrame() {
  animateId.value = requestAnimationFrame(() => {
    if (i % 20 === 0)
      frame()
    i++

    if (pendingTokens.length === 0 && isDrawing.value)
      isDrawing.value = false

    startFrame()
  })
}

function random(min: number, max: number) {
  return Math.floor(Math.random() * (max - min)) + min
}

function steps() {
  const steps: Function[] = Array.from({ length: 10 }).map(() => {
    return () => step({
      startPoint: { x: random(1, WIDTH), y: HEIGHT },
      angel: Math.PI / 2,
      length: random(20, 60),
    })
  })
  steps.forEach(step => step())
}

function redraw() {
  isDrawing.value = true
  ctx.clearRect(0, 0, WIDTH, HEIGHT)
  pendingTokens.length = 0
  if (!animateId.value)
    startFrame()
  steps()
}

function stop() {
  if (!isDrawing.value)
    return
  isDrawing.value = false
  if (animateId.value) {
    cancelAnimationFrame(animateId.value)
    animateId.value = null
  }
}

function keepOn() {
  if (isDrawing.value)
    return
  isDrawing.value = true
  if (!animateId.value)
    startFrame()
}

onMounted(() => {
  init()
  steps()
  startFrame()
})
</script>

<template>
  <canvas ref="el" :height="HEIGHT" :width="WIDTH" />
  <div class="action">
    <button @click="redraw">
      Redraw
    </button>
    <button v-if="animateId" @click="stop">
      Stop
    </button>
    <button v-else @click="keepOn">
      Continue
    </button>
  </div>
</template>

<style scoped>
  canvas {
    border: 1px solid black;
  }
  .action {
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 24px;
    gap: 12px;
  }

  button {
    display: block;
    width: 12rem;
    padding: 12px 24px;
    cursor: pointer;
  }
</style>
