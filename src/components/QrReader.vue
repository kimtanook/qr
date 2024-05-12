<template>
  <div>
    <video
      class="media"
      ref="myMediaRef"
      id="video"
      autoplay
      playsinline
    ></video>
    <select @change="changeCamera">
      <option>asd</option>
      <option v-for="item in videoInput" :value="item.deviceId">
        {{ item.label }}
      </option>
    </select>
  </div>
</template>

<script setup lang="ts">
import {onMounted, ref} from "vue";

const myStream = ref();
const myMediaRef = ref();
const videoInput = ref();

// 미디어 가져오기
const getMedia = async (deviceId: string | undefined) => {
  const initialConstrains = {
    audio: false,
    video: {facingMode: "user", width: 640, FrameRate: 15},
  };
  const cameraConstraints = {
    audio: false,
    video: {deviceId: {exact: deviceId}, width: 320},
  };

  try {
    myStream.value = await navigator.mediaDevices.getUserMedia(
      deviceId ? cameraConstraints : initialConstrains
    );

    if (myMediaRef?.value) {
      myMediaRef.value.srcObject = myStream.value;
    }
    if (!deviceId) {
      await getCamera();
    }
  } catch (e) {
    console.log(e);
  }
};

// 카메라 가져오기
const getCamera = async () => {
  try {
    const myCameras = await navigator.mediaDevices.enumerateDevices();
    const videoInputData = myCameras.filter(
      (item) => item.kind === "videoinput"
    );
    const currentCamera = myStream.value?.getVideoTracks()[0];
    videoInput.value = videoInputData;
  } catch (e) {
    console.log("e : ", e);
  }
};

// 카메라 변경
const changeCamera = (e: Event) => {
  const value = (e.target as HTMLSelectElement).value;
  getMedia(value);
};

onMounted(() => {
  getMedia("");
});
</script>
<style scoped>
.media {
  width: 90vw;
  height: 50vh;
}
</style>
