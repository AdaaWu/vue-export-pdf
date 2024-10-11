<template>
    <div>
        <div ref="pdfContent" class="pdf-content">
            <h1>Hello, Vue!</h1>
            <p>This is a content that will be exported to PDF.</p>
            <img src="https://kinsta.com/wp-content/uploads/2019/08/jpg-vs-jpeg.jpg" alt="Sample Image" />
        </div>
        <button @click="exportComponentToPdf">Export to PDF</button>

        
            <HiddenComponent />
 

    </div>
</template>

<script setup>
import { ref, createApp } from 'vue';
import { PDFDocument, rgb } from 'pdf-lib';
import * as htmlToImage from 'html-to-image'; // 正確的引用方式
import HiddenComponent from './HiddenComponent.vue';

const exportComponentToPdf1 = async () => {
    if (!pdfContent.value) return;

    try {
        // 使用 html-to-image 生成 PNG 資料 URL
        const dataUrl = await htmlToImage.toPng(pdfContent.value);

        // 移除 data URL 的前綴，僅保留 base64 資料
        const base64 = dataUrl.replace(/^data:image\/png;base64,/, '');

        // 將 base64 資料轉換為二進位資料
        const pngImageBytes = Uint8Array.from(atob(base64), c => c.charCodeAt(0));

        // 建立新的 PDF 文件
        const pdfDoc = await PDFDocument.create();
        const page = pdfDoc.addPage([595.28, 841.89]);

        // 將圖片嵌入到 PDF
        const pngImage = await pdfDoc.embedPng(pngImageBytes);
        const imageWidth = 595.28;
        const imageHeight = (pngImage.height / pngImage.width) * imageWidth;

        page.drawImage(pngImage, {
            x: 0,
            y: page.getHeight() - imageHeight,
            width: imageWidth,
            height: imageHeight,
        });

        // 生成 PDF 文件並轉為 Blob
        const pdfBytes = await pdfDoc.save();
        const blob = new Blob([pdfBytes], { type: 'application/pdf' });
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = 'exported-hidden-component.pdf';
        link.click();
    } catch (error) {
        console.error('Error exporting hidden component to PDF:', error);
    }
};



/////////////////////////////////////
const pdfContent = ref(null);

const exportComponentToPdf = async () => {
    if (!pdfContent.value) return;

    try {
        // 將 Vue component 轉換成圖片
        const dataUrl = await htmlToImage.toPng(pdfContent.value);

        // 建立新的 PDF 文件
        const pdfDoc = await PDFDocument.create();
        const page = pdfDoc.addPage([595.28, 841.89]); // 設定頁面大小為 A4

        // 將圖片添加到 PDF
        const pngImage = await pdfDoc.embedPng(dataUrl);
        const imageWidth = 595.28; // 圖片寬度調整為 A4 寬度
        const imageHeight = (pngImage.height / pngImage.width) * imageWidth; // 依比例調整高度
        page.drawImage(pngImage, {
            x: 0, // 將圖片靠左對齊
            y: page.getHeight() - imageHeight, // 將圖片靠上對齊
            width: imageWidth,
            height: imageHeight,
        });

        // 生成 PDF 文件並轉為 Blob
        const pdfBytes = await pdfDoc.save();
        const blob = new Blob([pdfBytes], { type: 'application/pdf' });

        // 下載 PDF 文件
        const link = document.createElement('a');
        link.href = URL.createObjectURL(blob);
        link.download = 'exported-component.pdf';
        link.click();
    } catch (error) {
        console.error('Error exporting component to PDF:', error);
    }
};
</script>

<style scoped>
.pdf-content {
    padding: 20px;
    background-color: white;
    width: 500px;
    /* 根據需要調整大小 */
}
</style>