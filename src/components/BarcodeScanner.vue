<template>
  <div ref="scanner" id="scanner" class="viewport">
    <video preload="auto" autoplay muted playsinline></video>
    <canvas class="drawingBuffer"></canvas>
  </div>
</template>

<script setup lang="ts">
import Quagga, { QuaggaJSConfigObject, QuaggaJSResultObject } from "@ericblade/quagga2";
import { onMounted, onUnmounted, ref } from "vue";

const scanner = ref<null | HTMLDivElement>(null);

const config = ref<QuaggaJSConfigObject>({
  inputStream: {
    type: 'LiveStream',
    willReadFrequently: true,
    target: '#scanner',
    constraints: {
      width: 6400,
      height: 4800
    }
  },
  locator: {
    halfSample: true,
    willReadFrequently: true,
    patchSize: 'x-small'
  },
  // numOfWorkers: 4,
  // frequency: 100,
  decoder: {
    // readers: [{
    //   format: 'ean_reader',
    //   config: {
    //     supplements: [
    //       'ean_13_reader'
    //     ]
    //   }
    // }],
    readers: ['ean_reader'],
    debug: {
      drawBoundingBox: true,
      showFrequency: true,
      drawScanline: true,
      showPattern: true
    },
    multiple: false
  },
  locate: true,
  debug: true
});

onMounted(() => {
  // config.value.inputStream!.constraints = {
  //   width: scanner.value!.offsetWidth,
  //   height: scanner.value!.offsetHeight,
  //   // facingMode: 'environment',
  //   aspectRatio: scanner.value!.offsetWidth / scanner.value!.offsetHeight
  // };

  Quagga.init(config.value, (err: any) => {
    if (err) {
      return console.error(err);
    }
    Quagga.start();
    Quagga.onDetected(onDetected);
    Quagga.onProcessed(onProcessed);
  });
});

onUnmounted(() => {
  // Quagga.offDetected(onDetected);
  Quagga.stop();
});

const onDetected = (result: QuaggaJSResultObject) => {
  console.log(result);
  alert(result.codeResult.code);
};

const onProcessed = (result: QuaggaJSResultObject) => {
  let drawingCtx = Quagga.canvas.ctx.overlay;
  let drawingCanvas = Quagga.canvas.dom.overlay;

  if (result) {
    if (result.boxes) {
      drawingCtx.clearRect(0, 0,
        parseInt(drawingCanvas.getAttribute('width') || '0'),
        parseInt(drawingCanvas.getAttribute('height') || '0'));

      result.boxes
        .filter((box: number[][]) => box !== result.box)
        .forEach((box: number[][]) =>
          Quagga.ImageDebug.drawPath(box, { x: 0, y: 1 }, drawingCtx, {
            color: 'green',
            lineWidth: 2
          })
        );

      if (result.box) {
        Quagga.ImageDebug.drawPath(result.box, { x: 0, y: 1 }, drawingCtx, {
          color: 'blue',
          lineWidth: 2
        });
      }

      if (result.codeResult && result.codeResult.code) {
        Quagga.ImageDebug.drawPath(
          result.line, { x: 'x', y: 'y' }, drawingCtx, {
            color: 'red',
            lineWidth: 3
          });
      }
    }
  }
}
</script>

<style scoped lang="scss">
#scanner {
  position: relative;
  text-align: center;

  video {
    margin: 0 auto;
  }

  canvas.drawingBuffer {
    position: absolute;
    left: 0;
    top: 0;
    right: 0;
    bottom: 0;
    margin: 0 auto;
  }
}
</style>