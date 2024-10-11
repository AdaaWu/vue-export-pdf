<template>
  <div id="content">
    <div class="chart-container">
      <!-- 畫面只顯示一張圖表 -->
      Title
      <VueApexCharts
        ref="chartRef"
        :type="chart.type"
        :options="chart.options"
        :series="chart.series"
      />
    </div>
    <button @click="exportChartsToPDF">匯出 PDF (20 張圖表)</button>
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
    const chart = ref({
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
    });

    const chartRef = ref(null);
    const exportChartsToPDF = async () => {
      await nextTick(); // 確保圖表已經渲染

      const pdfDoc = await PDFDocument.create();
      pdfDoc.registerFontkit(fontkit);

      // 讀取自定義字型
      const fontBytes = await fetch(NotoSansTC).then((res) => res.arrayBuffer());
      const customFont = await pdfDoc.embedFont(fontBytes);

      // 獲取圖表的 Base64 圖片
      const chartInstance = chartRef.value?.chart; // 確保獲取到正確的 chart 實例

      if (!chartInstance) {
        console.error('Chart instance not found.');
        return;
      }

      const chartDataURI = await chartInstance.dataURI();
      const img = await pdfDoc.embedPng(chartDataURI.imgURI);

      // 生成 20 張相同的圖表頁面
      for (let i = 0; i < 100; i++) {
        const page = pdfDoc.addPage([600, 400]);
        page.drawImage(img, {
          x: 50,
          y: 150,
          width: img.width * 0.5,
          height: img.height * 0.5,
        });
        page.drawText(`Title ${i + 1}`, {
          x: 50,
          y: 350,
          size: 24,
          font: customFont,
          color: rgb(0, 0, 0),
        });
      }

      // 匯出 PDF
      const pdfBytes = await pdfDoc.save();
      const blob = new Blob([pdfBytes], { type: 'application/pdf' });
      const url = URL.createObjectURL(blob);
      const link = document.createElement('a');
      link.href = url;
      link.download = 'chartswait.pdf';
      link.click();
      URL.revokeObjectURL(url);
    };

    return {
      chart,
      chartRef,
      exportChartsToPDF,
    };
  },
};
</script>

<style scoped>
.chart-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.chart {
  width: 60%;
}

button {
  margin-top: 20px;
}
</style>
