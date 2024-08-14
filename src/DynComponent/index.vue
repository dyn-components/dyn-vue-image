<script lang="tsx" setup>
import './styles/index.scss'
import 'swiper/css';
import 'swiper/css/navigation';
import 'swiper/css/pagination';
import 'swiper/css/zoom';
import Swiper from 'swiper';
import { Navigation, Pagination, Zoom } from 'swiper/modules';
import { ref } from 'vue';

const dialogRef = ref<any>()
const previewSwiperRef = ref<any>()
const dialogVisble = ref(false)
const activeIndex = ref(0)
const activeIndexZoom = ref<number>(1);

const props = withDefaults(defineProps<{
  src: string,
  previewSrcList?: string[],
  initialIndex?: number
}>(), {
  previewSrcList: () => [],
  initialIndex: 0
}); 

// HEX 转换为 RGB
function hexToRgb(hex: string) {
  hex = hex.replace('#', '');
  const bigint = parseInt(hex, 16);
  const r = (bigint >> 16) & 255;
  const g = (bigint >> 8) & 255;
  const b = bigint & 255;
  return `${r}, ${g}, ${b}`;
}

let swiper: Swiper | null = null
const onClick = (_event: MouseEvent) => {
  if (!dialogRef.value?.dialog || swiper) {
    return
  }
  dialogVisble.value = true
  const hexColor = getComputedStyle(dialogRef.value).getPropertyValue('--dyn-background-color').trim();
  const rgbColor = hexToRgb(hexColor);
  dialogRef.value.dialog.style.backgroundColor = `rgba(${rgbColor}, 0.6)`;
  swiper = new Swiper(previewSwiperRef.value, {
    modules: [Navigation, Pagination, Zoom],
    zoom: false, // 双击缩放
    loop: true,  // 启用无限循环轮播
    initialSlide: props.initialIndex,
    pagination: {
      el: ".swiper-pagination",
      type: "progressbar",
      clickable: true,
    },
    navigation: {
      nextEl: ".swiper-button-next",
      prevEl: ".swiper-button-prev",
    },
    // autoplay: {
    //   delay: 3000,  // 设置自动播放的时间间隔（毫秒）
    //   disableOnInteraction: false,  // 用户操作后是否禁用自动播放
    // },
  });

  swiper.on('slideChange', () => {
    const images = previewSwiperRef.value!.querySelectorAll('img.preview-img') as HTMLImageElement[];
    for (const image of images) {
      image.style.transform = 'scale(1)';
    }
    activeIndex.value = swiper!.realIndex;
    activeIndexZoom.value = 1;
  });
}

const onImageWheel = (event: WheelEvent) => {
  event.preventDefault();
  const image = event.target as HTMLImageElement;
  let scale = Number(image.style.transform.match(/scale\((\d+(?:\.\d+)?)\)/)?.[1] || 1);
  const scaleAmount = 0.1;  // 控制缩放的速度
    
  if (event.deltaY < 0) {
    scale += scaleAmount;
  } else {
    scale = Math.max(scale - scaleAmount, 0.1);  // 防止缩放到负值
  }
  
  image.style.transform = `scale(${scale})`;
  activeIndexZoom.value = scale;
}

const onPre = () => {
  previewSwiperRef.value?.swiper.slidePrev();
}

const onNext = () => {
  previewSwiperRef.value?.swiper.slideNext();
}

const onZoomIn = () => {
  activeIndexZoom.value = Math.max(activeIndexZoom.value - 0.1, 0.1);
  // 设置当前激活的swiper item内的图片样式
  const images = previewSwiperRef.value!.querySelectorAll('img.preview-img') as HTMLImageElement[];
  images[activeIndex.value].style.transform = `scale(${activeIndexZoom.value})`;
}

const onZoomOut = () => {
  activeIndexZoom.value = activeIndexZoom.value + 0.1;
  const images = previewSwiperRef.value!.querySelectorAll('img.preview-img') as HTMLImageElement[];
  images[activeIndex.value].style.transform = `scale(${activeIndexZoom.value})`;
}

const onDialogClose = () => {
  dialogVisble.value = false
  if (swiper) {
    swiper.destroy();
    swiper = null;
  }
}
</script>

<template>
  <img class="dyn-component--vue dyn-image" :src="src" @click="onClick" v-bind="$attrs"></img>
  <dyn-web-dialog ref="dialogRef" :open="dialogVisble" fullscreen modal closeable @close="onDialogClose">
    <div class="preview-container">
      <div ref="previewSwiperRef" class="swiper preview-swiper">
        <div class="swiper-wrapper">
          <template v-for="previewItem in props.previewSrcList">
            <div class="swiper-slide">
              <div class="swiper-zoom-container"> 
                <img ref="previewImagesRef" class="preview-img" :src="previewItem" :alt="previewItem" @wheel="onImageWheel">
              </div>
            </div>
          </template>
        </div>
        <div class="swiper-button-next"></div>
        <div class="swiper-button-prev"></div>
        <div class="swiper-pagination"></div>
      </div>
      <div class="toolbar">
        <div>
          <span class="prev" @click="onPre"></span>
          <span class="count">{{ activeIndex + 1 }} / {{ previewSrcList.length }}</span>
          <span class="next" @click="onNext"></span>
        </div>
        <div class="zoom">
          <span class="zoom-in" @click="onZoomIn">-</span>
          <span class="zoom-percent">{{ Math.round(activeIndexZoom * 100) }}%</span>
          <span class="zoom-out" @click="onZoomOut">+</span>
        </div>
      </div>
    </div> 
  </dyn-web-dialog>
</template>

<style lang="scss" scoped>
.preview-container, .swiper {
  height: 100%;
  user-select: none;
} 

.swiper {
  .preview-img {
    transform-origin: center center;
    transform: scale(1);
    transition: all 0.3s ease;
  }
}

.toolbar {
  position: absolute;
  bottom: 20px;
  left: 50%;
  display: inline-flex;
  justify-content: center;
  align-items: center;
  padding: 10px;
  transform: translate(-50%, 0);
  z-index: 100;

  .prev, .next {
    display: inline-block;
    font-family: swiper-icons;
    cursor: pointer;
    padding: 5px;
    margin: 10px;
    &:hover {
      filter: brightness(1.5);
    }
  }
  .prev {
    &:after {
      content: 'prev';
    }
  }
  .next {
    &:after {
      content: 'next';
    }
  }

  .zoom-in, .zoom-out {
    display: inline-block;
    cursor: pointer;
    padding: 5px;
    margin: 10px;
    font-size: 15px;
    font-weight: bold;
    &:hover {
      filter: brightness(1.5);
    }
  }
}
</style>
