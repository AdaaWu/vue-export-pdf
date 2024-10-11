<template>
  <div id="content">
    <div class="chart-container">
      <VueApexCharts
        ref="chart1Ref"
        :type="charts[0].type"
        :options="charts[0].options"
        :series="charts[0].series"
      />
      <VueApexCharts
        ref="chart2Ref"
        :type="charts[1].type"
        :options="charts[1].options"
        :series="charts[1].series"
      />
    </div>
    <button @click="exportChartsToPDF">匯出所有圖表為 PDF</button>
  </div>
</template>

<script>
import { ref, nextTick } from 'vue';
import VueApexCharts from 'vue3-apexcharts';
import { PDFDocument, rgb } from 'pdf-lib';
import fontkit from '@pdf-lib/fontkit';
import NotoSansTC from '../../public/fonts/NotoSansTC.ttf';

export default {
  components: { VueApexCharts },
  setup() {
    const charts = ref([
      {
        type: 'line',
        options: {
          chart: {
            id: 'chart1',
            height: 350,
          },
          title: {
            text: '圖表 1',
          },
          xaxis: {
            categories: [1991, 1992, 1993, 1994, 1995, 1996],
          },
        },
        series: [
          {
            name: 'Series 1',
            data: [30, 40, 35, 50, 49, 60],
          },
        ],
      },
      {
        type: 'bar',
        options: {
          chart: {
            id: 'chart2',
            height: 350,
          },
          title: {
            text: '圖表 2',
          },
          xaxis: {
            categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'],
          },
        },
        series: [
          {
            name: 'Series 2',
            data: [20, 30, 45, 60, 50, 40],
          },
        ],
      },
    ]);

    // 使用 ref 來獲取圖表實例
    const chart1Ref = ref(null);
    const chart2Ref = ref(null);

    const exportChartsToPDF = async () => {
      await nextTick(); // 確保圖表已經渲染

      const pdfDoc = await PDFDocument.create();
      pdfDoc.registerFontkit(fontkit);

      // 讀取自定義字型
      const fontBytes = await fetch(NotoSansTC).then((res) => res.arrayBuffer());
      const customFont = await pdfDoc.embedFont(fontBytes);

      // 獲取圖表的 Base64 圖片
      const chart1 = chart1Ref.value?.chart; // 修改為正確的 chart 實例
      const chart2 = chart2Ref.value?.chart; // 修改為正確的 chart 實例

      if (!chart1 || !chart2) {
        console.error('Chart instances not found.');
        return;
      }

      const chart1DataURI = await chart1.dataURI();
      const chart2DataURI = await chart2.dataURI();

      // 嵌入第一個圖表
      const img1 = await pdfDoc.embedPng(chart1DataURI.imgURI);
      const page1 = pdfDoc.addPage([600, 400]);
      page1.drawImage(img1, {
        x: 50,
        y: 150,
        width: img1.width * 0.5,
        height: img1.height * 0.5,
      });
      page1.drawText('圖表 1', {
        x: 50,
        y: 350,
        size: 24,
        font: customFont,
        color: rgb(0, 0, 0),
      });

      // 嵌入第二個圖表
      const img2 = await pdfDoc.embedPng(chart2DataURI.imgURI);
      const page2 = pdfDoc.addPage([600, 400]);
      page2.drawImage(img2, {
        x: 50,
        y: 150,
        width: img2.width * 0.5,
        height: img2.height * 0.5,
      });
      page2.drawText('圖表 2', {
        x: 50,
        y: 350,
        size: 24,
        font: customFont,
        color: rgb(0, 0, 0),
      });

      // 匯出 PDF
      const pdfBytes = await pdfDoc.save();
      const blob = new Blob([pdfBytes], { type: 'application/pdf' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = 'charts.pdf';
      link.click();
      URL.revokeObjectURL(url);
    };

    return {
      charts,
      chart1Ref,
      chart2Ref,
      exportChartsToPDF,
    };
  },
};
</script>

<style scoped>
.chart-container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  margin-bottom: 20px;
}

.chart {
  width: 45%;
}

button {
  margin-top: 20px;
}
</style>
