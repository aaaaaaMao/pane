<script setup lang="ts">
import { onMounted, ref } from 'vue'
const mainCanvas = ref<any>(null)

onMounted(() => {
  const context: CanvasRenderingContext2D = mainCanvas.value.getContext('2d')

  let devicePixelRatio = window.devicePixelRatio || 1
  let backingStoreRatio = getBackingStoreRatio(context)
  let ratio = devicePixelRatio / backingStoreRatio

  scale(context, ratio)

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
            <select id="paint-tools">
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
