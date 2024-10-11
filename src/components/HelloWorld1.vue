<template>
  <div id="content">
    <div class="convert-section">
      <h2>這是一個要轉換為 PDF 的區塊</h2>
      <p>使用 pdf-lib 生成包含簡單內容的 PDF。</p>
      <button @click="exportToPDF">匯出為 PDF</button>
    </div>
  </div>
</template>

<script>
import { PDFDocument, rgb } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';

export default {
  setup() {
    const exportToPDF = async () => {
      // 建立新的 PDF 文件
      const pdfDoc = await PDFDocument.create();
      pdfDoc.registerFontkit(fontkit);

      // 使用 fetch 透過 URL 載入字型檔案
      const fontUrl = '/fonts/NotoSansTC.ttf'; // 引用 public 資料夾下的字型
      const fontBytes = await fetch(fontUrl).then((res) => res.arrayBuffer());
      const customFont = await pdfDoc.embedFont(fontBytes);

      // 添加頁面並使用嵌入字型繪製中文
      const page = pdfDoc.addPage([600, 400]);
      page.drawText('簡報標題', {
        x: 50,
        y: 350,
        size: 30,
        font: customFont,
        color: rgb(0, 0, 0),
      });

      // 保存並下載 PDF
      const pdfBytes = await pdfDoc.save();
      const blob = new Blob([pdfBytes], { type: 'application/pdf' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = 'output.pdf';
      link.click();
      URL.revokeObjectURL(url);
    };

    return {
      exportToPDF,
    };
  },
};

</script>

<style scoped>
.convert-section {
  border: 1px solid #ddd;
  padding: 20px;
  margin-bottom: 20px;
  background-color: #f9f9f9;
}

button {
  margin-top: 10px;
}
</style>
