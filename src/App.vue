<template>
  <Toaster position="top-center" />
  <div class="container">
    <input type="file" accept="image/*" @change="onFileChange" />

    高度：<input v-model="pixelHeight">
    宽度：<input v-model="pixelWidth">
    每行最大像素宽：（建议不超过30）<input v-model="maxPixelWidth">
    <!-- 原图显示 -->
    原图：<img v-if="imgUrl" :src="imgUrl" class="preview" />

    <!-- 像素画 canvas -->
    预览：<canvas ref="canvas" :width="pixelWidth" :height="pixelHeight"></canvas>

    <!-- <pre class="output">
{{ pixels }}
    </pre> -->

    <div v-for="(line, index) in pixels" :key="index">
      <CopyBox :content="line"></CopyBox>
    </div>

  </div>
</template>

<script setup>
import { ref } from 'vue'
import CopyBox from './components/CopyBox.vue'
import { Toaster } from 'vue-sonner'
import 'vue-sonner/style.css'

const canvas = ref(null)
const imgUrl = ref(null)
const pixelHeight = ref(20)//像素画高
const pixelWidth = ref(20) //像素画宽
const maxPixelWidth = ref(20) //每行最大像素画宽
const pixels = ref([])

function onFileChange(e) {
  const file = e.target.files[0]
  if (!file) return

  imgUrl.value = URL.createObjectURL(file)

  const img = new Image()
  img.onload = () => drawToPixelCanvas(img)
  img.src = imgUrl.value
}

function drawToPixelCanvas(img) {
  const ctx = canvas.value.getContext('2d')

  // 禁用平滑，确保“硬像素”
  ctx.imageSmoothingEnabled = false

  // 清空画布
  ctx.clearRect(0, 0, pixelHeight.value, pixelWidth.value)

  // 把原图压缩绘制到小 canvas
  ctx.drawImage(img, 0, 0, pixelHeight.value, pixelWidth.value)

  // 读取像素数据
  const imageData = ctx.getImageData(0, 0, pixelHeight.value, pixelWidth.value)
  const data = imageData.data

  const result = []

  let pixelIndex = 0;
  let pixelLineIndex = 0;

  let line = '';
  let isTrans = false;
  for (let i = 0; i < data.length; i += 4) {

    pixelIndex++;
    pixelLineIndex++;

    let hexWithAlpha = rgbaToHex(
      data[i],
      data[i + 1],
      data[i + 2],
      data[i + 3]
    )

    if (data[i + 3] == 0) {
      if (isTrans) {
        line += "█";
      }
      else {
        line += `<color=${hexWithAlpha}>█`;
        isTrans = true;
      }
    }
    else {

      if (isTrans) {
        line += `</color><color=${hexWithAlpha}>█</color>`;
        isTrans = false;
      }
      else {
        line += `<color=${hexWithAlpha}>█</color>`;
      }
      isTrans = false;

    }

    if (pixelIndex >= maxPixelWidth.value || pixelLineIndex >= pixelWidth.value) {
      pixelIndex = 0;
      if (pixelLineIndex >= pixelWidth.value) {
        pixelLineIndex = 0;
      }

      if (isTrans) {
        line += "</color>";
      }

      result.push(line);
      line = '';//清空line
      isTrans = false;
    }

  }

  pixels.value = result
}

function rgbaToHex(r, g, b, a = 255) {
  const toHex = v => v.toString(16).padStart(2, '0').toUpperCase()
  return `#${toHex(r)}${toHex(g)}${toHex(b)}${toHex(a)}`
}

</script>

<style scoped>
.container {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.preview {
  max-width: 200px;
  border: 1px solid #ccc;
}

canvas {
  width: 200px;
  height: 200px;
  border: 1px solid #000;
  image-rendering: pixelated;
}

.output {
  max-height: 200px;
  overflow: auto;
  background: #111;
  color: #0f0;
  padding: 8px;
  font-size: 12px;
}
</style>
