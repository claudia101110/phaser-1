<script setup>
// 1. Importem ref (referència al DOM) i els hooks del cicle de vida de Vue 3
import { ref, watch, onMounted, onUnmounted } from "vue";
// 2. Importem el motor Phaser 4 i la nostra escena personalitzada
import Phaser from "phaser";
import { MainScene } from "../game/MainScene";
const props = defineProps(["nomUsuari", "colorAvatar", "pausat", "estat"]); // [NOU] Rep la prop des d'App.vue

const contenidor = ref(null); // Apuntarà al <div ref="contenidor"> de sota
let game = null; // Guardem la instància per destruir-la en sortir

// S'executa un sol cop quan el <div> ja existeix a la pàgina
onMounted(() => {
  game = new Phaser.Game({
    type: Phaser.AUTO, // Tria WebGL automàticament (o Canvas 2D)
    width: 640,
    height: 400, // Mides del llenç en píxels
    backgroundColor: "#1e293b", // Fons fosc per distingir l'àrea del joc
    parent: contenidor.value, // Injecta el <canvas> dins del nostre <div>
    scene: [MainScene], // Escena inicial que arrencarà el motor
  });
  game.events.once("ready", () => {
    game?.scene.getScene("MainScene")?.actualitzarNom(props.nomUsuari);
    game?.scene.getScene("MainScene")?.canviarColor(props.colorAvatar);
    game?.scene.getScene("MainScene")?.actualitzarEstat(props.estat);
    if (props.pausat == true) {
      game?.scene.pause("MainScene");
    }
  });
});

// IMPRESCINDIBLE: quan s'amaga (v-if="false"), atura el bucle a 60 FPS
onUnmounted(() => game?.destroy(true));
// [NOU] Al MATEIX nivell que onMounted (MAI dins d'onMounted)
watch(
  () => props.nomUsuari,
  (nouNom) => {
    const escena = game?.scene.getScene("MainScene");
    escena?.actualitzarNom(nouNom);
  },
);
watch(
  () => props.colorAvatar,
  (nouColor) => {
    game?.scene.getScene("MainScene")?.canviarColor(nouColor);
  },
);
watch(
  () => props.pausat,
  (valor) => {
    if (valor == true) {
      game?.scene.pause("MainScene");
    } else {
      game?.scene.resume("MainScene");
    }
  },
);
watch(
  () => props.estat,
  (nouEstat) => {
    game?.scene.getScene("MainScene")?.actualitzarEstat(nouEstat);
  },
);
</script>
<template><div ref="contenidor"></div></template>
