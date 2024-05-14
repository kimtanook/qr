<template>
  <a :href="result">{{ result }}</a>
  <div
    v-for="item in trackFunctionOptions"
    @click="trackFunctionSelected = item"
  >
    {{ item.text }}
  </div>
  <div>
    <qrcode-stream
      :constraints="{deviceId: selectedDevice}"
      :track="trackFunctionSelected.value"
      :formats="selectedBarcodeFormats"
      @error="onError"
      @detect="onDetect"
      v-if="selectedDevice !== null"
    />
    <p v-else class="error">No cameras on this device</p>
    <div v-for="item in cameraList" @click="selectedDevice = item.deviceId">
      {{ item.label }}
    </div>
  </div>
</template>

<script setup lang="ts">
import {computed, onMounted, ref} from "vue";
import {QrcodeStream} from "vue-qrcode-reader";

const result = ref(""); // qr코드 감지 결과

// 트랙
const trackFunctionOptions = [
  {text: "nothing (default)", value: undefined},
  {text: "outline", value: paintOutline},
  {text: "centered text", value: paintCenterText},
  {text: "bounding box", value: paintBoundingBox},
];
const trackFunctionSelected = ref(trackFunctionOptions[1]); // 선택한 트랙. (바운더리, 텍스트 등)

// 포맷
const barcodeFormats: any = ref({
  aztec: false,
  code_128: false,
  code_39: false,
  code_93: false,
  codabar: false,
  databar: false,
  databar_expanded: false,
  data_matrix: false,
  dx_film_edge: false,
  ean_13: false,
  ean_8: false,
  itf: false,
  maxi_code: false,
  micro_qr_code: false,
  pdf417: false,
  qr_code: true,
  rm_qr_code: true,
  upc_a: false,
  upc_e: false,
  linear_codes: false,
  matrix_codes: false,
});
const selectedBarcodeFormats = computed(() => {
  return Object.keys(barcodeFormats.value).filter(
    (format) => barcodeFormats.value[format]
  );
});

// 카메라
const selectedDevice: any = ref(null);
const devices: any = ref([]);
const cameraList: any = ref([]);

// 에러
const error = ref("");

// 감지
function onDetect(detectedCodes: any) {
  console.log(detectedCodes);
  JSON.stringify(
    detectedCodes.map((code: any) => {
      result.value = code.rawValue;
    })
  );
}

// 트랙
function paintOutline(detectedCodes: any, ctx: any) {
  for (const detectedCode of detectedCodes) {
    const [firstPoint, ...otherPoints] = detectedCode.cornerPoints;

    ctx.strokeStyle = "red";

    ctx.beginPath();
    ctx.moveTo(firstPoint.x, firstPoint.y);
    for (const {x, y} of otherPoints) {
      ctx.lineTo(x, y);
    }
    ctx.lineTo(firstPoint.x, firstPoint.y);
    ctx.closePath();
    ctx.stroke();
  }
}
function paintBoundingBox(detectedCodes: any, ctx: any) {
  for (const detectedCode of detectedCodes) {
    const {
      boundingBox: {x, y, width, height},
    } = detectedCode;

    ctx.lineWidth = 2;
    ctx.strokeStyle = "#007bff";
    ctx.strokeRect(x, y, width, height);
  }
}
function paintCenterText(detectedCodes: any, ctx: any) {
  for (const detectedCode of detectedCodes) {
    const {boundingBox, rawValue} = detectedCode;

    const centerX = boundingBox.x + boundingBox.width / 2;
    const centerY = boundingBox.y + boundingBox.height / 2;

    const fontSize = Math.max(12, (50 * boundingBox.width) / ctx.canvas.width);

    ctx.font = `bold ${fontSize}px sans-serif`;
    ctx.textAlign = "center";

    ctx.lineWidth = 3;
    ctx.strokeStyle = "#35495e";
    ctx.strokeText(detectedCode.rawValue, centerX, centerY);

    ctx.fillStyle = "#5cb984";
    ctx.fillText(rawValue, centerX, centerY);
  }
}

// 에러 핸들링
function onError(err: any) {
  error.value = `[${err.name}]: `;

  if (err.name === "NotAllowedError") {
    error.value += "you need to grant camera access permission";
  } else if (err.name === "NotFoundError") {
    error.value += "no camera on this device";
  } else if (err.name === "NotSupportedError") {
    error.value += "secure context required (HTTPS, localhost)";
  } else if (err.name === "NotReadableError") {
    error.value += "is the camera already in use?";
  } else if (err.name === "OverconstrainedError") {
    error.value += "installed cameras are not suitable";
  } else if (err.name === "StreamApiNotSupportedError") {
    error.value += "Stream API is not supported in this browser";
  } else if (err.name === "InsecureContextError") {
    error.value +=
      "Camera access is only permitted in secure context. Use HTTPS or localhost rather than HTTP.";
  } else {
    error.value += err.message;
  }
}

onMounted(async () => {
  devices.value = (await navigator.mediaDevices.enumerateDevices()).filter(
    ({kind}) => kind === "videoinput"
  );
  if (devices.value.length > 0) {
    cameraList.value = devices.value;
    selectedDevice.value = devices.value[1].deviceId;
  }
});
</script>

<style scoped>
.error {
  font-weight: bold;
  color: red;
}
.barcode-format-checkbox {
  margin-right: 10px;
  white-space: nowrap;
}
</style>
