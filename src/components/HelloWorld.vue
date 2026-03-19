<template>
  <div class="content">
    <div class="container">
      <div class="footer">
        <button class="btn" @click="iniciarGrabacion" :disabled="escuchando">
          🎙️ Iniciar
        </button>

        <div v-if="escuchando" class="recording-indicator">
          <span class="dot"></span>
          Escuchando...
        </div>
      </div>

      <div v-for="(msg, i) in mensajes" :key="i">
        <b>{{ msg.tipo === "usuario" ? "Tú:" : "Gemini:" }}</b>
        {{ msg.texto }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

const mensajes = ref([]);
const escuchando = ref(false);

let recognition;

const iniciarGrabacion = () => {
  recognition = new webkitSpeechRecognition();

  escuchando.value = true;

  recognition.lang = "es-MX";
  recognition.continuous = false;
  recognition.interimResults = false;

  recognition.onresult = async (event) => {
    const texto = event.results[0][0].transcript;

    mensajes.value.push({
      tipo: "usuario",
      texto,
    });

    const response = await fetch("http://localhost:3000/mensaje", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({ mensaje: texto }),
    });

    const data = await response.json();

    mensajes.value.push({
      tipo: "bot",
      texto: data.botText,
    });

    hablar(data.botText);
  };

  recognition.onend = () => {
    escuchando.value = false;
  };

  recognition.start();
};

const hablar = (texto) => {
  const speech = new SpeechSynthesisUtterance(texto);
  speech.lang = "es-MX";
  speech.rate = 1;

  window.speechSynthesis.speak(speech);
};
</script>
