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

// 引入外部字型
import NotoSans from '../../public/fonts/NotoSansTC.ttf'; // 確保字型路徑正確

export default {
  setup() {
    const exportToPDF = async () => {
      const pdfDoc = await PDFDocument.create();
      pdfDoc.registerFontkit(fontkit);

      // 透過 fetch 載入字型檔案
      const fontBytes = await fetch(NotoSans).then((res) => res.arrayBuffer());
      const customFont = await pdfDoc.embedFont(fontBytes);

      const page = pdfDoc.addPage([600, 400]);

      page.drawText('簡報標題', {
        x: 50,
        y: 350,
        size: 30,
        font: customFont,
        color: rgb(0, 0, 0),
      });

      page.drawRectangle({
        x: 50,
        y: 200,
        width: 500,
        height: 100,
        borderColor: rgb(0, 0, 0),
        borderWidth: 1,
      });

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
