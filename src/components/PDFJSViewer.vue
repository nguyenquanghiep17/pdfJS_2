<template>
    <div>
        <div class="wrap-canvas">
            <canvas id="1" ></canvas>
        </div>
        <div class="wrap-canvas">
            <canvas id="2" ></canvas>
        </div>
        <div class="wrap-canvas">
            <canvas id="3" ></canvas>
        </div>
        <div class="wrap-canvas">
            <canvas id="4" ></canvas>
        </div>
        
    </div>
</template>
    
<script>
import * as pdfjsLib from "../../public/pdfjs-dist/build/pdf.js";
import "../../public/pdfjs-dist/web/pdf_viewer.css";
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
            objReq.url = "https://raw.githubusercontent.com/mozilla/pdf.js/ba2edeae/web/compressed.tracemonkey-pldi-09.pdf";
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
                        canvas.height = 1359;
                        canvas.width = 1050;
                        canvas.style.width = viewport.width + "px";
                        canvas.style.height = viewport.height + "px";

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
    .wrap-canvas {
        width: 394px;
        height: 511px;
    }
</style>