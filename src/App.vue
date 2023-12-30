<script setup lang="ts">
import { onMounted, ref } from 'vue'
const mainCanvas = ref<any>()
const paintTools = ref<any>()

const tools = ref([{
  name: 'Line',
  do: drawLine,
}, {
  name: 'Erase',
  do: erase,
}, {
  name: 'Text',
  do: drawText,
}, {
  name: 'Spray',
  do: spray,
}])

onMounted(() => {
  const context: CanvasRenderingContext2D = mainCanvas.value.getContext('2d')

  let devicePixelRatio = window.devicePixelRatio || 1
  let backingStoreRatio = getBackingStoreRatio(context)
  let ratio = devicePixelRatio / backingStoreRatio

  scale(context, ratio)

  context.canvas.addEventListener('mousedown', (event: any) => {
    if (event.which == 1) {
      const toolName = paintTools.value.value
      const tool = tools.value.find(x => x.name === toolName)
      if (tool) {
        tool.do(event, context)
      }
      event.preventDefault()
    }
  })

})

</script>

<script lang="ts">

function getBackingStoreRatio(ctx: any) {
  return ctx.webkitBackingStorePixelRatio
    || ctx.mozBackingStorePixelRatio
    || ctx.msBackingStorePixelRatio
    || ctx.oBackingStorePixelRatio
    || ctx.backingStorePixelRatio
    || 1
}

function scale(ctx: CanvasRenderingContext2D, ratio: number) {
  const canvas = ctx.canvas

  const w = canvas.width
  const h = canvas.height

  // canvas.width = w * ratio
  // canvas.height = h * ratio
  canvas.style.width = `${w}px`
  canvas.style.height = `${h}px`

  ctx.scale(ratio, ratio)
}

function relativePos(event: any, element: Element) {
  const rect = element.getBoundingClientRect()
  return {
    x: Math.floor(event.clientX - rect.left),
    y: Math.floor(event.clientY - rect.top)
  }
}

function trackDrag(onMove: any, onEnd: any) {
  function end(event: any) {
    removeEventListener("mousemove", onMove);
    removeEventListener("mouseup", end);
    if (onEnd) {
      onEnd(event)
    }
  }
  addEventListener("mousemove", onMove);
  addEventListener("mouseup", end);
}

function drawLine(event: any, ctx: CanvasRenderingContext2D, onEnd?: any) {
  ctx.lineCap = "round"

  var pos = relativePos(event, ctx.canvas)
  trackDrag(function (event: any) {
    ctx.beginPath()
    ctx.moveTo(pos.x, pos.y)
    // console.log('b', pos.x, pos.y)

    pos = relativePos(event, ctx.canvas)
    ctx.lineTo(pos.x, pos.y)
    // console.log('e', pos.x, pos.y)

    ctx.stroke()
  }, onEnd)
}

function erase(event: any, ctx: CanvasRenderingContext2D) {
  ctx.globalCompositeOperation = "destination-out"
  drawLine(event, ctx, function () {
    ctx.globalCompositeOperation = "source-over"
  })
}

function drawText(event: any, ctx: CanvasRenderingContext2D) {
  const text = prompt("Text:", "")

  if (text) {
    var pos = relativePos(event, ctx.canvas)
    ctx.font = Math.max(7, ctx.lineWidth) + "px sans-serif"
    ctx.fillText(text, pos.x, pos.y)
  }
}

function spray(event: any, ctx: CanvasRenderingContext2D) {
  var radius = ctx.lineWidth / 2
  var area = radius * radius * Math.PI
  var dotsPerTick = Math.ceil(area / 30)

  var currentPos = relativePos(event, ctx.canvas)
  var spray = setInterval(function () {
    for (var i = 0; i < dotsPerTick; i++) {
      var offset = randomPointInRadius(radius)
      ctx.fillRect(currentPos.x + offset.x,
        currentPos.y + offset.y, 1, 1)
    }
  }, 25);
  trackDrag(function (event: any) {
    currentPos = relativePos(event, ctx.canvas);
  }, function () {
    clearInterval(spray);
  })
}

function randomPointInRadius(radius: number) {
  for (; ;) {
    var x = Math.random() * 2 - 1;
    var y = Math.random() * 2 - 1;
    if (x * x + y * y <= 1)
      return { x: x * radius, y: y * radius };
  }
}

</script>

<template>
  <div id="container">
    <div id="top-menu">
      menu
    </div>
    <div id="content-div">
      <div id="tool-bar">

        <div>
          <span>Tool:
            <select id="paint-tools" ref="paintTools">
              <option v-for="tool in tools" :key="tool.name">{{ tool.name }}</option>
            </select>
          </span>
        </div>

        <div>
          <span>Color:
            <input type="color" id="color-tool">
          </span>
        </div>

        <div>
          <span>Brush size:
            <select id="brush-size">
            </select>
          </span>
        </div>

      </div>
      <div id="pane-div">
        <canvas id="main-canvas" ref="mainCanvas"></canvas>
      </div>
    </div>
  </div>
</template>

<style scoped>
#container {
  height: 100%;
  width: 100%;
  margin: 0px;
  padding: 0px;
}

#top-menu {
  height: 20px;
}

#content-div {
  width: 100%;
  height: 95%;
}

#tool-bar {
  width: 10%;
  height: 100%;
  float: left;
}

#color-bar {
  width: 10%;
  height: 100%;
  float: left;
}

#pane-div {
  width: 90%;
  height: 100%;
  float: left;
}

#main-canvas {
  background-color: #FFFFFF;
  width: 800px;
  height: 600px;
}
</style>
