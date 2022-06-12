<script setup lang="ts">
import { onMounted, reactive, ref, watch } from 'vue'
import { readFileAsync } from "../utils/readFile";
import { loadImage } from "../utils/loadImage";
import { WaterMark, WaterMarkConfig } from "../types/WaterMark";
import { Vector2 } from '../types/Vector';
import ImagePicker from "./ImagePicker.vue";
import SizePicker from "./SizePicker.vue";
import watermark_layout_lb from '../assets/watermark_layout_lb_noborder.svg'
import watermark_layout_lt from '../assets/watermark_layout_lt_noborder.svg'
import watermark_layout_center from '../assets/watermark_layout_center_noborder.svg'
import watermark_layout_rb from '../assets/watermark_layout_rb_noborder.svg'
import watermark_layout_rt from '../assets/watermark_layout_rt_noborder.svg'

const props = defineProps<{
  selectedFiles: Array<File>,
  setSelectedFile: (newFiles: Array<File>) => void,
  prevProcedure: () => void,
}>()

const image_canvas = ref<HTMLCanvasElement>()
const watermark_canvas = ref<HTMLCanvasElement>()
const icon_picker = ref<HTMLInputElement>();
let myImage = new Image()
let currentFile: File
const waterMarkConfig = reactive<WaterMarkConfig>({
  textContent: "@arno_solo",
  textSize: 50,
  iconSize: 50,
  frameWidth: 5,
  opacity: 1,
  position: new Vector2(0, 0)
})

let waterMark = new WaterMark(watermark_canvas.value, waterMarkConfig)

onMounted(() => {
  if (props.selectedFiles.length > 0) {
    redrawAll(props.selectedFiles)
  }
})

watch(props.selectedFiles, async (newVal, oldVal) => {
  redrawAll(newVal)
})

async function redrawAll(newFiles: Array<File>) {
  try {
    let imageUrl = await readFileAsync(newFiles[0])
    myImage = await loadImage(imageUrl)
    currentFile = newFiles[0]
    console.log(currentFile);


    if (image_canvas.value == undefined || watermark_canvas.value == undefined) {
      console.error("Can't find canvas")
    } else {
      let imageLayer = image_canvas.value
      let watermarkLayer = watermark_canvas.value
      let imageCtx = imageLayer.getContext('2d')
      let watermarkLayerCtx = watermarkLayer.getContext('2d')
      imageLayer.width = myImage.width;
      imageLayer.height = myImage.height;
      watermarkLayer.width = myImage.width;
      watermarkLayer.height = myImage.height;
      waterMark.canvas = watermarkLayer

      if (imageCtx && watermarkLayerCtx) {
        waterMarkConfig.textSize = Math.floor(myImage.width * 0.03)
        waterMarkConfig.iconSize = Math.floor(myImage.width * 0.03)
        // waterMark.frameWidth = waterMark.textSize * 0.1
        waterMark.position.x = myImage.height * 0.5
        waterMark.position.y = myImage.width * 0.5

        imageCtx.drawImage(myImage, 0, 0)
        waterMark.refreshCanvas()

        // handleMouseDown
        watermarkLayer.addEventListener('mousedown', evt => {
          handleMouseDown(evt.clientX, evt.clientY)
        })

        watermarkLayer.addEventListener('touchend', evt => {
          handleMouseDown(evt.changedTouches[0].clientX, evt.changedTouches[0].clientY)
        })

        const handleMouseDown = (clientX: number, clientY: number) => {
          const rect = watermarkLayer.getBoundingClientRect()
          let mousePos = new Vector2(
            (clientX - rect.left) * (watermarkLayer.width / rect.width),
            (clientY - rect.top) * (watermarkLayer.height / rect.height)
          )
          waterMark.isSelected = waterMark.isMouseAbove(mousePos)
          Vector2.sub(mousePos, waterMark.position, waterMark.distanceToMouse)
        }

        // handleMouseMove
        watermarkLayer.addEventListener('mousemove', evt => {
          handleMouseMove(evt.clientX, evt.clientY)
        })

        watermarkLayer.addEventListener('touchmove', evt => {
          handleMouseDown(evt.changedTouches[0].clientX, evt.changedTouches[0].clientY)
        })

        const handleMouseMove = (clientX: number, clientY: number) => {
          if (waterMark.isSelected) {
            const rect = watermarkLayer.getBoundingClientRect()
            let mousePos = new Vector2(
              (clientX - rect.left) * (watermarkLayer.width / rect.width),
              (clientY - rect.top) * (watermarkLayer.height / rect.height)
            )
            Vector2.sub(mousePos, waterMark.distanceToMouse, waterMark.position)
            watermarkLayerCtx?.clearRect(0, 0, watermarkLayer.width, watermarkLayer.height)
            waterMark.show()
          }
        }

        // handleMouseUp
        watermarkLayer.addEventListener('mouseup', evt => {
          handleMouseUp()
        })

        watermarkLayer.addEventListener('touchend', evt => {
          handleMouseUp()
        })

        const handleMouseUp = () => {
          waterMark.isSelected = false
          watermarkLayerCtx?.clearRect(0, 0, watermarkLayer.width, watermarkLayer.height)
          watermarkLayerCtx?.drawImage(myImage, 0, 0)
          waterMark.show()
        }

      } else {
        console.warn('Can\'t get context');
      }
    }
  } catch (error) {
    console.error(error)
  }
}

watch(() => waterMarkConfig.textContent, (newVal, oldVal) => {
  if (newVal) {
    waterMark.textContent = newVal
  } else {
    waterMark.textContent = ''
  }
})

watch(() => waterMarkConfig.textSize, (newVal, oldVal) => {
  if (newVal) {
    waterMark.textSize = newVal
  } else {
    console.warn('New textSize is undefined');
  }
})

watch(() => waterMarkConfig.textColor, (newVal, oldVal) => {
  if (newVal) {
    waterMark.textColor = newVal
  } else {
    console.warn('New textColor is undefined');
  }
})

watch(() => waterMarkConfig.iconSize, (newVal, oldVal) => {
  if (newVal) {
    waterMark.iconSize = newVal
  } else {
    console.warn('New iconSize is undefined');
  }
})

function selectIcon() {
  icon_picker.value?.click();
}

async function handleIconChange() {
  const files = icon_picker.value?.files;
  if (files) {
    try {
      let imageUrl = await readFileAsync(files[0])
      waterMark.icon = await loadImage(imageUrl)
    } catch (err) {
      console.error(err)
    }
  } else {
    console.warn("Can't find icon file");
  }
}

function downloadImage() {
  if (watermark_canvas.value) {
    let watermarkLayer = watermark_canvas.value
    const watermarkLayerCtx = watermarkLayer.getContext('2d')
    watermarkLayerCtx?.clearRect(0, 0, watermarkLayer.width, watermarkLayer.height)
    watermarkLayerCtx?.drawImage(myImage, 0, 0)
    waterMark.show()
    const dataUrl = watermarkLayer.toDataURL(currentFile.type)
    var tmpLink = document.createElement('a');
    tmpLink.download = currentFile.name;
    tmpLink.href = dataUrl;

    document.body.appendChild(tmpLink);
    tmpLink.click();
    document.body.removeChild(tmpLink);
  }
}

interface LayoutOption {
  src: string,
  value: string,
  isActived: boolean,
}
let layoutOptions = reactive<Array<LayoutOption>>([
  {
    src: watermark_layout_lb,
    value: 'left_bottom',
    isActived: true,
  },
  {
    src: watermark_layout_lt,
    value: 'left_top',
     isActived: false,
  },
  {
    src: watermark_layout_center,
    value: 'center',
     isActived: false,
  },
  {
    src: watermark_layout_rt,
    value: 'right_top',
     isActived: false,
  },
  {
    src: watermark_layout_rb,
    value: 'right_bottom',
     isActived: false,
  },
])

function changeWatermarkLayout(layout: LayoutOption) {
  if(!layout.isActived) {
    layoutOptions.forEach(opt => opt.isActived = false)
    // const newOpt = layoutOptions.find(opt => opt.value == layout.value)
    // if(newOpt) newOpt.isActived = true
    layout.isActived = true
    switch (layout.value) {
      case 'left_top':
        waterMark.position = new Vector2(
          waterMark.iconSize * 1.4,
          myImage.height * 0.01 + waterMark.bigHeight
          )
        break;
      case 'left_bottom':
        waterMark.position = new Vector2(
          waterMark.iconSize * 1.4,
          myImage.height * 0.98 - waterMark.bigHeight
          )
        break;
      case 'right_top':
        const x = waterMark.textContent.length
        waterMark.position = new Vector2(
          myImage.width * 0.98 - waterMark.textWidth,
          myImage.height * 0.01 + waterMark.bigHeight
          )
        break;
      case 'right_bottom':
        waterMark.position = new Vector2(
          myImage.width * 0.98 - waterMark.textWidth,
          myImage.height * 0.98 - waterMark.bigHeight
          )
        break;
      case 'center':
        waterMark.position = new Vector2(
          (myImage.width - waterMark.textWidth) * 0.5,
          myImage.height * 0.5
          )
        break;

      default:
        break;
    }
  }
}

function setIconSize(newVal: string) {
  waterMarkConfig.iconSize = Number(newVal)
}

function changeWatermarkColor(colorOpt: ColorOption) {
  if(!colorOpt.isActived) {
    colorOptions.forEach(opt => opt.isActived = false)
    colorOpt.isActived = true
    waterMarkConfig.textColor = colorOpt.color
  }
}

interface ColorOption {
  color: string
  borderColor: string
  hasOutborder: boolean
  isActived: boolean
}
let colorOptions = reactive<Array<ColorOption>>([
  {
    color: '#ffaa00',
    borderColor: '#fff',
    hasOutborder: false,
    isActived: false,
  },
  {
    color: '#ffffff',
    borderColor: '#e5e7eb',
    hasOutborder: true,
    isActived: true,
  },
  {
    color: '#555555',
    borderColor: '#fff',
    hasOutborder: false,
    isActived: false,
  },
  {
    color: '#1a71fc',
    borderColor: '#fff',
    hasOutborder: false,
    isActived: false,
  },
  {
    color: '#32c759',
    borderColor: '#fff',
    hasOutborder: false,
    isActived: false,
  },
  {
    color: '#fe3a30',
    borderColor: '#fff',
    hasOutborder: false,
    isActived: false,
  },
])
  
</script>

<template>
  <div class="flex flex-wrap justify-center">
    <div class="static">
      <canvas class="w-screen sm:w-96 md:w-[30rem] border " ref="image_canvas"></canvas>
      <canvas class="w-screen sm:w-96 md:w-[30rem] border absolute top-0" ref="watermark_canvas"></canvas>
    </div>
      <!-- Control Panel -->
      <div class="flex flex-col gap-2 p-2">
        <ImagePicker :setSelectedFile="setSelectedFile" :selectedFiles="selectedFiles"></ImagePicker>
        <div class="flex flex-wrap gap-1">
          <img class="w-6" src="../assets/add_image_black.svg" @click="selectIcon" alt="icon image">
          <input class="invisible w-0" type="file" ref="icon_picker" accept="image/*" @change="handleIconChange">
          <input v-model="waterMarkConfig.iconSize" type="number" step="5" class="w-8">
        </div>
        <!-- <SizePicker :sizeNum="waterMarkConfig.iconSize" @num-change="setIconSize"></SizePicker> -->
        <div class="flex flex-wrap gap-1">
          <img class="w-6 p-0.5" src="../assets/input.svg" alt="text image">
          <input v-model="waterMarkConfig.textContent" type="text" class="w-28 text-lg" placeholder="watermark">
          <input v-model="waterMarkConfig.textSize" type="number" step="5" class="w-8">
        </div>
        <div class="flex align-middle gap-1">
          <img class="w-6" src="../assets/opacity_block.svg" alt="opacity image">
          <input v-model="waterMark.opacity" type="number" step="0.1" max="1" min="0" class="w-8">
        </div>

        <div class="flex items-center gap-1.5">
          <img class="w-6" src="../assets/color.svg" alt="color image">
          <div v-for="colorOpt of colorOptions"
            class="w-5 h-5 p-0.5 rounded-full flex"
            @click="changeWatermarkColor(colorOpt)"
            :class="{'border': colorOpt.hasOutborder}"
            :style="{'background': colorOpt.color}"
          >
            <div class="flex-1 rounded-full"
              :class="{'border-2': colorOpt.isActived}"
              :style="{'border-color': colorOpt.borderColor}"
            ></div>
          </div>
         
          <input v-model="waterMarkConfig.textColor" type="string" class="w-20">
        </div>

        <div class="flex flex-wrap gap-2">
          <img v-for="layout of layoutOptions"
            @click="changeWatermarkLayout(layout)"
            :key="layout.value"
            :src="layout.src" :alt="layout.value" 
            class="pt-1 rounded-lg border"
            :class="{
              'border-2': layout.isActived,
              'border-blue-500': layout.isActived,
              'border-gray-500': !layout.isActived,
            }"
          >
        </div>

        <button @click="downloadImage" class="flex align-middle gap-1">
          <img class="w-6 p-0.5" src="../assets/download_bold.svg" alt="download image">
          <p class="font-semibold text-m">Download</p>
        </button>

        <div class="h-8"></div>
      </div>
    </div>
</template>

<style scoped>
input[type=number] {
  -moz-appearance: textfield;
}

input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}

.primary-button:active {
  transform: scale(0.98);
  box-shadow: 0em 0.1em 0.3em rgba(0, 0, 0, 0.24);
}
/* div{
  border-color:#ffaa00;
} */
</style>
