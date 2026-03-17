<template>
  <div class="container">
    <div class="content">
      <p>Mensajes o contenido aquí...</p>

      <div class="audio-bar" v-if="audioUrl">
        <audio :src="audioUrl" controls></audio>
      </div>
    </div>

    <div v-if="grabando" class="recording-indicator">
      <span class="dot"></span>
      Grabando...
    </div>

    <!-- 🔘 Botones -->
    <div class="footer">
      <button class="btn" @click="iniciarGrabacion">🎤 Iniciar</button>
      <button class="btn stop" @click="detenerGrabacion">⏹ Detener</button>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const mediaRecorder = ref(null);
const audioChunks = ref([]);
const audioUrl = ref(null);
const grabando = ref(false);

const iniciarGrabacion = async () => {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  mediaRecorder.value = new MediaRecorder(stream);

  audioChunks.value = [];
  grabando.value = true;

  mediaRecorder.value.ondataavailable = (e) => {
    audioChunks.value.push(e.data);
  };

  mediaRecorder.value.onstop = () => {
    const blob = new Blob(audioChunks.value, { type: "audio/wav" });
    audioUrl.value = URL.createObjectURL(blob);
    grabando.value = false;
  };

  mediaRecorder.value.start();
};

const detenerGrabacion = () => {
  if (mediaRecorder.value) {
    mediaRecorder.value.stop();
  }
};
</script>
