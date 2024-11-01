<template>
  <div>
    <input type="file" @change="onFileChange" multiple accept="application/pdf" />
    <button @click="mergeAndDownloadPDF">合并并下载PDF</button>
  </div>
</template>

<script>
import { PDFDocument } from 'pdf-lib';

export default {
  data() {
    return {
      pdfFiles: [],
    };
  },
  methods: {
    onFileChange(event) {
      this.pdfFiles = Array.from(event.target.files);
    },
    async mergeAndDownloadPDF() {
      if (this.pdfFiles.length < 2) {
        alert('请选择至少两个PDF文件');
        return;
      }
      try {
        const mergedPdfBytes = await this.mergePDFs(this.pdfFiles);

        // 创建下载链接
        const blob = new Blob([mergedPdfBytes], { type: 'application/pdf' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = 'merged.pdf';
        a.click();
        URL.revokeObjectURL(url);
      } catch (error) {
        console.error('合并PDF时出错:', error);
      }
    },
    async mergePDFs(pdfFiles) {
      const mergedPdf = await PDFDocument.create();
      for (const pdfFile of pdfFiles) {
        const pdfBytes = await pdfFile.arrayBuffer();
        const pdfDoc = await PDFDocument.load(pdfBytes);
        const pages = await mergedPdf.copyPages(pdfDoc, pdfDoc.getPageIndices());
        pages.forEach((page) => mergedPdf.addPage(page));
      }
      return mergedPdf.save();
    },
  },
};
</script>
