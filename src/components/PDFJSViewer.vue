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
                        let sx = window.devicePixelRatio || 1;
                        let sy = window.devicePixelRatio || 1;
                        const outputScale = {
                            sx: sx,
                            sy: sy,
                            scaled: sx !== 1 || sy !== 1
                        }
                        const sfx = self.approximateFraction(outputScale.sx);
                        const sfy = self.approximateFraction(outputScale.sy);
                        

                        canvas.width = self.roundToDivide(viewport.width * outputScale.sx, sfx[0]);
                        canvas.height = self.roundToDivide(viewport.height * outputScale.sy, sfy[0]);
                        // canvas.width = 1050;
                        // canvas.height = 1359;
                        canvas.style.width = self.roundToDivide(viewport.width, sfx[1]) + "px";
                        canvas.style.height = self.roundToDivide(viewport.height, sfy[1]) + "px";

                        var weakMap = new WeakMap();
                        weakMap.set(canvas, viewport);
                        const transform = outputScale.scaled
                        ? [outputScale.sx, 0, 0, outputScale.sy, 0, 0]
                        : null;
                        page
                            .render({
                                canvasContext: canvas.getContext("2d", { alpha: false }),
                                transform: transform,
                                viewport: viewport
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
        },
        methods: {
            approximateFraction(x) {
                // Fast paths for int numbers or their inversions.
                if (Math.floor(x) === x) {
                    return [x, 1];
                }
                const xinv = 1 / x;
                const limit = 8;
                if (xinv > limit) {
                    return [1, limit];
                } else if (Math.floor(xinv) === xinv) {
                    return [1, xinv];
                }

                const x_ = x > 1 ? xinv : x;
                // a/b and c/d are neighbours in Farey sequence.
                let a = 0,
                    b = 1,
                    c = 1,
                    d = 1;
                // Limiting search to order 8.
                while (true) {
                    // Generating next term in sequence (order of q).
                    const p = a + c,
                    q = b + d;
                    if (q > limit) {
                    break;
                    }
                    if (x_ <= p / q) {
                    c = p;
                    d = q;
                    } else {
                    a = p;
                    b = q;
                    }
                }
                let result;
                // Select closest of the neighbours to x.
                if (x_ - a / b < c / d - x_) {
                    result = x_ === x ? [a, b] : [b, a];
                } else {
                    result = x_ === x ? [c, d] : [d, c];
                }
                return result;
            },
            roundToDivide(x, div) {
                const r = x % div;
                return r === 0 ? x : Math.round(x - r + div);
            }
        }
    }
</script>
<style scoped>
</style>