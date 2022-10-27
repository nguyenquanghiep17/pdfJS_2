<template>
    <div>
        <div class="wrap" role="region" id="wrap-1">
            <div class="wrap-canvas" id="wrap-canvas-1">
                <canvas id="1" role="presentation"></canvas>
            </div>
        </div>
        <div class="wrap" role="region" id="wrap-2">
            <div class="wrap-canvas" id="wrap-canvas-2">
                <canvas id="2" role="presentation"></canvas>
            </div>
        </div>
        <div class="wrap" role="region" id="wrap-3">
            <div class="wrap-canvas" id="wrap-canvas-3">
                <canvas id="3" role="presentation"></canvas>
            </div>
        </div>

        <div class="wrap" role="region" id="wrap-4">
            <div class="wrap-canvas" id="wrap-canvas-4">
                <canvas id="4" role="presentation"></canvas>
            </div>
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
                    // self.defaultConvertRatio = 1;
                    for (let i = 1; i < 5; i++) {
                        var page = await self.pdf.getPage(i);
                        var viewport = page.getViewport({ scale: self.defaultConvertRatio });
                        var wrap = document.getElementById("wrap-" + i);
                        wrap.style.width = Math.floor(viewport.width) + "px";
                        wrap.style.height = Math.floor(viewport.height) + "px";
                        var wrapCanvas = document.getElementById("wrap-canvas-" + i);
                        wrapCanvas.style.width = Math.floor(viewport.width) + "px";
                        wrapCanvas.style.height = Math.floor(viewport.height) + "px";

                        var canvas = document.getElementById(i + "");

                        canvas.width = viewport.width;
                        canvas.height = viewport.height;


                        page
                            .render({
                                canvasContext: canvas.getContext("2d", { alpha: false }),
                                transform: null,
                                viewport: viewport,
                                annotationMode: 2,
                                pageColors: null
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