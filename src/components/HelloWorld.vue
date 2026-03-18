<template>
  <div>
    <div class="footer">
      <button class="btn" @click="iniciarGrabacion">🎤 Hablar</button>
    </div>

    <div v-for="(msg, i) in mensajes" :key="i">
      <b>{{ msg.tipo === "usuario" ? "Tú:" : "Bot:" }}</b>
      {{ msg.texto }}
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
  recognition.lang = "es-MX";
  recognition.continuous = false;
  recognition.interimResults = false;

  escuchando.value = true;

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
