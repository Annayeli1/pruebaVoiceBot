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
      🎤 {{ tiempoFormateado }}
    </div>

    <!-- 🔘 Botones -->
    <div class="footer">
      <button class="btn" @click="iniciarGrabacion">🎤 Iniciar</button>
      <button class="btn stop" @click="detenerGrabacion">⏹ Detener</button>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";

const mediaRecorder = ref(null);
const audioChunks = ref([]);
const audioUrl = ref(null);
const grabando = ref(false);

const segundos = ref(0);
let intervalo = null;

// ⏱️ Formato mm:ss
const tiempoFormateado = computed(() => {
  const min = String(Math.floor(segundos.value / 60)).padStart(2, "0");
  const sec = String(segundos.value % 60).padStart(2, "0");
  return `${min}:${sec}`;
});

const iniciarGrabacion = async () => {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  mediaRecorder.value = new MediaRecorder(stream);

  audioChunks.value = [];
  grabando.value = true;
  segundos.value = 0;

  // ⏱️ iniciar contador
  intervalo = setInterval(() => {
    segundos.value++;
  }, 1000);

  mediaRecorder.value.ondataavailable = (e) => {
    audioChunks.value.push(e.data);
  };

  mediaRecorder.value.onstop = () => {
    const blob = new Blob(audioChunks.value, { type: "audio/wav" });
    audioUrl.value = URL.createObjectURL(blob);
    grabando.value = false;

    clearInterval(intervalo);
  };

  mediaRecorder.value.start();
};

const detenerGrabacion = () => {
  if (mediaRecorder.value) {
    mediaRecorder.value.stop();
  }
};
</script>
