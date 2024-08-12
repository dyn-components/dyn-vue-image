<script lang="tsx" setup>
import './styles/index.scss'
import 'swiper/css';
import 'swiper/css/navigation';
import 'swiper/css/pagination';
import 'swiper/css/zoom';
import Swiper from 'swiper';
import { Navigation, Pagination, Zoom } from 'swiper/modules';
import { ref } from 'vue';

const dialogRef = ref()
const previewSwiperRef = ref()
const dialogVisble = ref(false)

const props = withDefaults(defineProps<{
  src: string,
  previewSrcList: string[]
}>(), {
  previewSrcList: () => []
}); 

const onClick = (_event: MouseEvent) => {
  dialogVisble.value = true

  if (dialogRef.value?.dialog) {
    dialogRef.value.dialog.style.backgroundColor = `rgba(0, 0, 0, 0.8)`;
    const swiper = new Swiper(previewSwiperRef.value, {
      modules: [Navigation, Pagination, Zoom],
      zoom: false, // 双击缩放
      loop: true,  // 启用无限循环轮播
      pagination: {
        el: ".swiper-pagination",
        // type: "progressbar",
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
      const images = document.querySelectorAll('img.preview-img') as any as HTMLImageElement[];
  for (const image of images) {
    image.style.transform = 'scale(1)';
  }
    });
  }
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
}
</script>

<template>
  <img class="dyn-component--vue dyn-image" :src="src" @click="onClick" v-bind="$attrs"></img>
  <dyn-web-dialog ref="dialogRef" :open="dialogVisble" fullscreen modal closeable @close="dialogVisble = false">
    <div class="preview-container">
      <div ref="previewSwiperRef" class="swiper preview-swiper">
        <div class="swiper-wrapper">
          <template v-for="previewItem in props.previewSrcList">
            <div class="swiper-slide">
              <div class="swiper-zoom-container"> 
                <img class="preview-img" :src="previewItem" :alt="previewItem" @wheel="onImageWheel">
              </div>
            </div>
          </template>
        </div>
        <div class="swiper-button-next"></div>
        <div class="swiper-button-prev"></div>
        <div class="swiper-pagination"></div>
      </div>
    </div> 
  </dyn-web-dialog>
</template>

<style lang="scss" scoped>
  .preview-container, .swiper {
    height: 100%;
  } 

  .swiper {
    .preview-img {
      transform-origin: center center;
      transform: scale(1);
      transition: all 0.3s ease;
    }
  }
</style>
