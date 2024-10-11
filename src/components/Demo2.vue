<template>
    <div>
      <h1>這是主要元件的標題</h1>
      <button @click="exportToPDF">匯出 PDF</button>
  
      <!-- 隱藏的 Vue 子元件，使用特定的 CSS class 來控制可見性 -->
      <div ref="hiddenComponentWrapper" :class="{ hidden: !isExporting }">
        <HiddenComponent />
        test
      </div>
    </div>
  </template>
  
  <script>
  import { ref, onMounted } from "vue";
  import html2pdf from "html2pdf.js";
  import HiddenComponent from "./HiddenComponent.vue";
  
  // 定義 ref 和狀態
  const hiddenComponentWrapper = ref(null);
  const isExporting = ref(false);
  
  // 匯出 PDF 的函數
  const exportToPDF = function () {
    isExporting.value = true; // 將內容顯示
    setTimeout(() => {
      const content = hiddenComponentWrapper.value;
      if (content) {
        html2pdf()
          .from(content)
          .save()
          .then(() => {
            isExporting.value = false; // 匯出完成後再次隱藏
          })
          .catch(() => {
            isExporting.value = false; // 如果發生錯誤，也要隱藏
          });
      } else {
        console.error("hiddenComponentWrapper is not available.");
        isExporting.value = false;
      }
    }, 100); // 稍微延遲，確保顯示後才匯出
  };
  
  export default {
    components: {
      HiddenComponent,
    },
    setup() {
      return {
        hiddenComponentWrapper,
        isExporting,
        exportToPDF,
      };
    },
  };
  </script>
  
  <style>
  .hidden {
    display: none;
  }
  </style>
  