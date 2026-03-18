<template>
  <div class="container">
    <div class="content">
      <p>Mensajes...</p>

      <div class="audio-bar" v-if="audioUrl">
        <audio :src="audioUrl" controls></audio>
      </div>
    </div>

    <div v-if="grabando" class="recording-indicator">
      <span class="dot"></span>
      🎤 {{ tiempoFormateado }}
    </div>

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

  intervalo = setInterval(() => {
    segundos.value++;
  }, 1000);

  mediaRecorder.value.ondataavailable = (e) => {
    audioChunks.value.push(e.data);
  };

  mediaRecorder.value.onstop = async () => {
    console.log("Enviando archivo...");
    const blob = new Blob(audioChunks.value, { type: "audio/wav" });
    audioUrl.value = URL.createObjectURL(blob);
    grabando.value = false;

    clearInterval(intervalo);

    const formData = new FormData();
    formData.append("audio", blob, "grabacion.wav");

    try {
      const response = await fetch("http://localhost:3000/mensaje", {
        method: "POST",
        body: formData,
      });

      const data = await response.json();
      console.log("Respuesta del servidor:", data);
    } catch (error) {
      console.error("Error enviando audio:", error);
    }
  };

  mediaRecorder.value.start();
};

const detenerGrabacion = () => {
  if (mediaRecorder.value) {
    mediaRecorder.value.stop();
  }
};
</script>
