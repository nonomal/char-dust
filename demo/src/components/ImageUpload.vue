<template>
  <el-form :inline="true" :model="config">
    <el-form-item label="图片大小">
      <el-input-number
        v-model="config.size"
        @change="handleChange"
        :min="0"
        :step="step"
        label="图片大小"
      ></el-input-number>
    </el-form-item>
    <el-form-item label="转换字符">
      <el-input v-model="config.char" placeholder="转换字符"></el-input>
    </el-form-item>
    <el-form-item>
      <el-button type="primary" @click="convert">转换</el-button>
    </el-form-item>
  </el-form>

  <hr />
  <br />

  <el-upload
    ref="imageUploadRef"
    drag
    action="/"
    :limit="1"
    :accept="'image/*'"
    :auto-upload="false"
    :on-change="onChange"
    :before-upload="beforeImageUpload"
  >
    <i class="el-icon-upload"></i>
    <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
    <div class="el-upload__tip">
      只能上传 {{ supportedFormats.join("/") }} 文件，且尽量不要过大
    </div>
    <img class="preview-image" :src="image.src" />
  </el-upload>

  <el-slider v-model="config.scale" show-input> </el-slider>

  <el-input
    type="textarea"
    autosize
    placeholder="ASCII 画"
    v-model="textarea"
    :style="textareaStyle"
  >
  </el-input>
</template>

<script lang="ts" setup>
import { ref, computed, reactive } from "vue";

import { UploadFile } from "element-plus/lib/components/upload/src/upload.type";
import { checkImageType } from "../utils/imageCommon";

import { imageToText, getImageData, DEFAULT_AVAILABLE_CHARS } from "char-dust";
import { ElMessage } from "element-plus";

const config = reactive({
  size: 300,
  char: DEFAULT_AVAILABLE_CHARS,
  scale: 100,
});

const step = ref(50);
const image = ref(new Image());
const supportedFormats = ["jpg", "png", "gif"];
const textarea = ref("");
const imageUploadRef = ref();

const textareaStyle = computed(() => {
  return {
    width: 100 / (config.scale / 100) + "%",
    transformOrigin: "left top",
    transform: `scale(${config.scale / 100})`,
  };
});

function handleChange() {
  if (image.value) {
    scaleImageContainer(image.value);
  }
}

function convert() {
  const imageData = getImageData(image.value);
  const text = imageToText(imageData, config.char);
  textarea.value = text.join("\n");
  console.log(text);
  ElMessage.success("转化完成");
}

/**
 * 当改变设置的图片大小时
 */
function onChange(file: UploadFile) {
  if (file) {
    previewImage(file);
    (imageUploadRef.value as any).clearFiles();
  }
}

function beforeImageUpload(file: File) {
  if (!checkImageType(file.type)) {
    ElMessage.error(`目前只支持 ${supportedFormats.join("/")} 格式的文件`);
    return false;
  }
  return true;
}

/**
 * 预览图片
 */
function previewImage(file: UploadFile) {
  const reader = new FileReader();
  reader.readAsDataURL(file.raw);
  reader.onload = (event) => {
    image.value = new Image();
    image.value.onload = () => {
      if (image.value) {
        scaleImageContainer(image.value);
      }
    };
    if (event.target) {
      image.value.src = event.target.result as string;
    }
  };
}

/**
 * 缩放 Upload Image 容器大小
 */
function scaleImageContainer(image: HTMLImageElement) {
  const uploadContainer = document.querySelector(
    ".el-upload-dragger"
  ) as HTMLElement;

  let targetWidth = config.size;
  if (!targetWidth && image.width) {
    targetWidth = image.width;
  }

  const ratio = image.width / targetWidth;
  const targetHeight = image.height / ratio;

  image.width = targetWidth;
  image.height = targetHeight;

  uploadContainer.style.width = targetWidth + "px";
  uploadContainer.style.height = targetHeight + "px";
}
</script>

<style lang="scss">
.preview-image {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}

.preview-image[src=""] {
  visibility: hidden;
}
</style>
