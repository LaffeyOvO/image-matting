<template>
    <div class="h-full flex flex-col items-center justify-center p-4">
      <div class="text-center">
        <h1 class="text-3xl font-bold mb-4">{{t('ai_inpainting.inpainting_home.title')}}</h1>
        <p class="mb-8">{{t('ai_inpainting.inpainting_home.desc')}}</p>
        <button class="bg-green-500 btn text-white px-4 py-2 rounded-full btn-md" @click="openFileDialog">
          <i class="fa-solid fa-image mr-2"></i>
          {{t('ai_inpainting.inpainting_home.upload_btn')}}
        </button>

        <p class="text-gray-500 mb-4 mt-8">
         {{ t('ai_inpainting.inpainting_home.tips') }}
        </p>
      </div>
      <div class="flex justify-center items-center">
      <span class="mx-2">———</span>
      <span class="mx-2">{{ t('ai_matting.matting_home.tips') }}</span>
      <span class="mx-2">———</span>
    </div>
    <div class="flex justify-center items-center mt-8 h-50">

      <img v-for="(item, index) in imageList" @click="tryItNow(item)" :key="index" class="w-40 h-40 rounded-lg object-cover mx-2"
        :src="item" alt="demo image">
    </div>
    </div>
  </template>

<script setup>
import { onMounted, onUnmounted } from 'vue'
import baseAPI from '@/api/base'
import { useRouter } from 'vue-router'
import message from '@/utils/message';
import { useI18n } from 'vue-i18n'
import imageList from '@/utils/demoImages'


const { t } = useI18n()

const router = useRouter()

// 打开file dialog
const openFileDialog = async () => {
  const res = await baseAPI('open_file_dialog', false)
  console.log(res)
  if (res.data['file_path']) {
    // 上传文件
    // 路由跳转到simpe-img
    router.push({ name: 'AIInpaintingEditor', query: { filePath: res.data['file_path'], imgType: 'local' } })
  }
}
// 试试看
const tryItNow = (base64Img) => {
  router.push({ name: 'AIInpaintingEditor', query: { filePath: base64Img, imgType: 'base64' } })
}

const handlePaste = (event) => {
  // 阻止默认行为
  event.preventDefault();

  // 获取粘贴板数据
  const clipboardData = event.clipboardData || window.Clipboard;
  const items = clipboardData.items;

  for (let i = 0; i < items.length; i++) {
    const item = items[i];

    if (item.kind === 'string') {
      item.getAsString((text) => {
        if (isValidUrl(text)) {
          console.log('粘贴的是URL:', text);
          router.push({ name: 'AIInpaintingEditor', query: { filePath: text, imgType: 'url' } })
        } else {
          message.error('请粘贴图片链接或图片文件');
          return;
        }
      });
    } else if (item.kind === 'file') {
      //  判断是否是图片文件
      const type = item.type.split('/')[0];
      if (!['image'].includes(type)) {
        // 不是图片文件
        message.error('请粘贴图片文件');
        return;
      }
      const file = item.getAsFile();
      const reader = new FileReader();
      reader.onload = (e) => {
        // base64编码的图片数据
        console.log('粘贴的是图片:', e.target.result);
        router.push({ name: 'AIInpaintingEditor', query: { filePath: e.target.result, imgType: 'base64' } })
      };
      reader.readAsDataURL(file);
    }
  }
};

// 验证是否是有效的URL
const isValidUrl = (string) => {
  try {
    new URL(string);
    return true;
  } catch (_) {
    return false;
  }
};

function handleDrop(event) {
  event.preventDefault();
  const files = event.dataTransfer.files;
  if (files.length) {
    const file = files[0];
    if (file.type.match('image.*')) {
      const reader = new FileReader();
      reader.onload = function (e) {
        router.push({ name: 'AIInpaintingEditor', query: { filePath: e.target.result, imgType: 'base64' } })
      };
      reader.readAsDataURL(file);
    }
  }
}

const handleDragOver = (event) => {
  event.preventDefault();
};

onMounted(() => {
  document.addEventListener('paste', handlePaste);
  document.addEventListener('dragover', handleDragOver);
  document.addEventListener('drop', handleDrop);
});
onUnmounted(() => {
  document.removeEventListener('paste', handlePaste);
  document.removeEventListener('dragover', handleDragOver);
  document.removeEventListener('drop', handleDrop);
});
</script>