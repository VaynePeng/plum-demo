<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'

const canvasRef = ref<HTMLCanvasElement | null>(null)

const ctx = computed(() => {
  return canvasRef.value?.getContext('2d') as CanvasRenderingContext2D
})

const WIDTH = 500
const HEIGHT = 500

interface Point {
  x: number
  y: number
}

interface Line {
  start: Point
  length: number
  angle: number
}

let pendingTasks: Array<Function> = []

const lineTo = (startPoint: Point, endPoint: Point) => {
  ctx.value.beginPath()
  ctx.value.moveTo(startPoint.x, startPoint.y)
  ctx.value.lineTo(endPoint.x, endPoint.y)
  ctx.value.stroke()
}

const getEndPoint = (line: Line): Point => {
  const { start, length, angle } = line
  return {
    x: start.x + length * Math.cos(angle),
    y: start.y + length * Math.sin(angle)
  }
}

const drawLine = (line: Line) => {
  const end = getEndPoint(line)
  lineTo(line.start, end)
}

const step = (line: Line, deep = 0) => {
  const endPoint = getEndPoint(line)
  drawLine(line)

  if (Math.random() > 0.5 || deep < 2) {
    // 绘制左侧分支
    pendingTasks.push(() => {
      step(
        {
          start: endPoint,
          length: line.length + (Math.random() * 10 - 10),
          angle: line.angle - 0.3 * Math.random()
        },
        deep + 1
      )
    })
  }

  if (Math.random() > 0.5 || deep < 2) {
    // 绘制右侧分支
    pendingTasks.push(() => {
      step(
        {
          start: endPoint,
          length: line.length + (Math.random() * 10 - 10),
          angle: line.angle + 0.3 * Math.random()
        },
        deep + 1
      )
    })
  }
}

const frame = () => {
  const tasks: Function[] = []
  pendingTasks = pendingTasks.filter((i) => {
    if (Math.random() > 0.4) {
      tasks.push(i)
      return false
    }
    return true
  })
  tasks.forEach((fn) => fn())
}

let framesCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    framesCount += 1
    if (framesCount % 3 === 0) {
      frame()
    }
    startFrame()
  })
}

const init = () => {
  ctx.value.strokeStyle = '#000'
  const line = {
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 50,
    angle: -Math.PI / 2
  }
  step(line)
  console.log('pendingTasks', pendingTasks)
  startFrame()
}

onMounted(() => {
  const canvas = canvasRef.value as HTMLCanvasElement
  const dpr = window.devicePixelRatio || 1
  canvas.width = canvas?.clientWidth * dpr
  canvas.height = canvas?.clientHeight * dpr
  ctx.value.scale(dpr, dpr)
  init()
})
</script>

<template>
  <div>
    <canvas ref="canvasRef" border border-solid w-500px h-500px />
  </div>
</template>
