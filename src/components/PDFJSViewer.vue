<template>
    <div>
        <canvas id="1"></canvas>
        <canvas id="2"></canvas>
        <canvas id="3"></canvas>
        <canvas id="4"></canvas>
    </div>
</template>
    
<script>
import * as pdfjsLib from "../../public/pdfjs-dist/legacy/build/pdf.js";
import "../../public/pdfjs-dist/legacy/web/pdf_viewer.css";
import pdfjsWorker from "../../public/pdfjs-dist/build/pdf.worker.entry.js";
pdfjsLib.GlobalWorkerOptions.workerSrc = pdfjsWorker;
    export default {
        name: 'PDFJSViewer',
        data() {
            return {
                width: 595,
                height: 841,
                pdf: {},
                defaultWidth: 784,
                defaultConvertRatio: 1
            }
        },
        created() {
            if (window.outerWidth < this.defaultWidth) {
                this.defaultWidth = window.outerWidth - 6;
            }
            var self = this;
            let objReq = {};
            objReq.url = "https://amisapp.misa.vn/wesign/api/file-management/api/General/file-pdf/stream?bucketName=wesign-production&objectId=39c0fd2c-e973-4f65-b0a3-73dd427c41f7&fileName=Template-Labor-Contract-2022.docx";
            objReq.withCredentials= false;

            var loadingTask = pdfjsLib.getDocument(objReq);

            loadingTask.promise
                .then(async (pdf) => {
                    self.pdf = pdf;
                    self.defaultConvertRatio = self.defaultWidth / self.width;
                    for (let i = 1; i < 5; i++) {
                        var page = await self.pdf.getPage(i);
                        var viewport = page.getViewport({ scale: self.defaultConvertRatio });
                        var canvas = document.getElementById(i + "");
                        canvas.height = viewport.height;
                        canvas.width = viewport.width;

                        page
                            .render({
                                canvasContext: canvas.getContext("2d"),
                                viewport: viewport,
                                intent: 'print',
                            })
                            .promise.then(() => {

                                return "render succeeded";
                            })
                            .catch((response) => {
                                console.error("Error while rendering canvas" + response);
                                throw "Error while rendering canvas" + response;
                            });
                    }
                    // pdf
                    //     .getPage(1)
                    //     .then((page) => {
                    //         var viewport = page.getViewport({ scale: 1 });
                    //         self.width = viewport.width;
                    //         self.height = viewport.height;
                    //         self.defaultConvertRatio = self.defaultWidth / self.width;
                    //         console.log(self.defaultConvertRatio, self.width, self.height)
                    //     })
                    //     .catch((ex) => {
                    //         console.error("Get first page: ", ex);
                    //     });

                })
                .catch((response) => {
                    console.error("Loi ROI " + response);
                });
        }
    }
</script>
<style scoped>
</style>