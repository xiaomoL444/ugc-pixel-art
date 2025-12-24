<template>
  <Toaster position="top-center" />
  <div style="display: flex; flex-direction: column;height:100vh;">
    <div style="/* text-align: center; */
  display: flex;
  /* justify-content: center; */
  /* 水平居中 */
  align-items: center;
    flex: 1.2;
  padding-left: 2rem;   background: linear-gradient(90deg, #6a5acd, #00bfff);">
      <img :src="imgSrc" style="height: 2.5rem; width: 2.5rem; margin-right: 1rem;">
      <h2 class="title">像素画生成器</h2>
    </div>
    <div class="container">
      <div class="block" style="display: flex;flex-direction: column; flex:0;  width: 20rem;      /* 固定宽度 */
  flex-shrink: 0;    /* 不允许被压缩 */">
        <Title :title="'选择图片'"></Title>

        <div style=" margin: 0.25rem; display: flex;
  flex-direction: column;
  gap: 12px; /* 子元素上下间距 */">
          <input ref="fileInput" type="file" style="display: none" accept="image/*" @change="onFileChange" />

          <!-- 原图显示 -->
          <!-- 原图：<img v-if="imgUrl" :src="imgUrl" class="preview" /> -->

          <!-- 像素画 canvas -->
          预览：
          <div style="overflow-y: auto;height: 16rem;">
            <canvas
              :style="{  height: 15.5 * pixelHeight / pixelWidth + 'rem', width: '15.5rem' }"
              ref="canvas" :width="pixelWidth" :height="pixelHeight"></canvas>
          </div>

          <button class="selectBtn" @click="selectFile">选择文件</button>

          <div class="ElementBox">
            <div class="ElementBoxTitle">高度</div>
            <div class="ElementContent"><input class="input" type="number" v-model="pixelHeight"></div>
          </div>



          <div class="ElementBox">
            <div class="ElementBoxTitle">宽度</div>
            <div class="ElementContent"><input class="input" type="number" v-model="pixelWidth"></div>
          </div>

          <div class="ElementBox">
            <div class="ElementBoxTitle">每行最大像素宽：（建议不超过30）</div>
            <div class="ElementContent"> <input class="input" type="number" v-model="maxPixelWidth"></div>
          </div>
        </div>
      </div>
      <div class="block" style="  overflow-y: scroll;flex: 1;">
        <Title :title="'分段输出'"></Title>
        <div class="outputContainer" v-for="(line, index) in pixels" :key="index">
          <CopyBox title='我是标题' :content="line"></CopyBox>
        </div>
      </div>
      <!-- <pre class="output">
{{ pixels }}
    </pre> -->
    </div>
  </div>
</template>

<script setup>

/* eslint-disable */

import logo from '@/assets/7.png'  // @ = src/
const imgSrc = logo

import { ref } from 'vue'
import CopyBox from './components/CopyBox.vue'
import { Toaster } from 'vue-sonner'
import 'vue-sonner/style.css'
import Title from './components/TitleBox.vue'

const canvas = ref(null)
const imgUrl = ref(null)
const pixelHeight = ref(20)//像素画高
const pixelWidth = ref(20) //像素画宽
const maxPixelWidth = ref(20) //每行最大像素画宽
const pixels = ref([])

const fileInput = ref(null)
function selectFile() {
  fileInput.value.click()
}

function onFileChange(e) {
  const file = e.target.files[0]
  if (!file) return

  imgUrl.value = URL.createObjectURL(file)

  const img = new Image()
  img.onload = () => drawToPixelCanvas(img)
  img.src = imgUrl.value
  e.target.value = ''
}

function drawToPixelCanvas(img) {
  const ctx = canvas.value.getContext('2d')

  // 禁用平滑，确保“硬像素”
  ctx.imageSmoothingEnabled = false

  // 清空画布
  ctx.clearRect(0, 0, pixelWidth.value, pixelHeight.value)

  // 把原图压缩绘制到小 canvas
  ctx.drawImage(img, 0, 0, pixelWidth.value, pixelHeight.value)

  // 读取像素数据
  const imageData = ctx.getImageData(0, 0, pixelWidth.value, pixelHeight.value)
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
.selectBtn {
  color: white;
  background-color: #4FACFE;
  border: #000;
  border-radius: 10px;
  font-size: 1.2rem;
  padding: 1rem;
  cursor: pointer;
  transition:
    background-color 0.3s ease,
    transform 0.2s ease-out,
    color 0.3s ease;
}

.selectBtn:hover {
  background-color: #337ecc;
  /* 更深一点的颜色 */

}

.title {
  font-weight: 700;
  /* 粗体 */
  font-size: 2rem;
  /* 放大一点好看 */
  /* background: linear-gradient(0deg, #6a5acd, #00bfff); */
  /* 紫 → 蓝 */
  background-color: white;
  -webkit-background-clip: text;
  /* 背景裁剪到文字 */
  color: transparent;
  /* 隐藏文字颜色，让渐变显现 */
  -webkit-text-fill-color: transparent;
  /* Safari 兼容 */
}

.container {
  display: flex;
  flex-direction: row;
  /* gap: 12px; */
  flex: 18;
  height: 90vh;
}

.container .block {
  border: 1px solid #ccc;
  flex: 1;

}

.container .block .titleBlock {
  border: 1px solid #ccc;
}

.container .block .containerBlock {
  border: 1px solid #ccc;
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

.outputContainer {
  margin: 0.5rem .5rem;
}
</style>
