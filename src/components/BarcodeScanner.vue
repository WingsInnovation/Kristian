<template>
  <QrcodeStream @detect="onDetect" :formats="['ean_13']"></QrcodeStream>
</template>

<script setup lang="ts">
import {QrcodeStream} from "vue-qrcode-reader";

type DetectResult = Object & {
  boundingBox: DOMRectReadOnly;
  cornerPoints: {
    x: number;
    y: number;
  }[];
  format: string;
  rawValue: string;
}

function onDetect(detected: DetectResult[]) {
  console.log(detected);
  alert(detected[0].rawValue)
}

</script>

<style scoped lang="scss">

</style>