<template>
  <div class="container">
    <div class="content">
      <!-- Tu contenido -->
      <p>Mensajes o contenido aquí...</p>
      <div class="audio-bar" v-if="audioUrl">
        <audio v-if="audioUrl" :src="audioUrl" controls></audio>
      </div>
    </div>

    <div class="footer">
      <button class="btn" @click="iniciarGrabacion">Iniciar🎤</button>
      <button class="btn" @click="detenerGrabacion">Detener</button>
    </div>
  </div>
</template>
<script>
export default {
  data() {
    return {
      mediaRecorder: null,
      audioChunks: [],
      audioUrl: null,
    };
  },
  methods: {
    async iniciarGrabacion() {
      const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
      this.mediaRecorder = new MediaRecorder(stream);

      this.mediaRecorder.ondataavailable = (e) => {
        this.audioChunks.push(e.data);
      };

      this.mediaRecorder.onstop = () => {
        const blob = new Blob(this.audioChunks, { type: "audio/wav" });
        this.audioUrl = URL.createObjectURL(blob);
        this.audioChunks = [];
      };

      this.mediaRecorder.start();
    },
    detenerGrabacion() {
      if (this.mediaRecorder) {
        this.mediaRecorder.stop();
      }
    },
  },
};
</script>
