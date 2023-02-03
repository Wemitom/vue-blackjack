<script setup lang="ts">
import { ref } from "vue";
import CardComp from "./components/Card-comp.vue";
import { deck, shuffle } from "./utils";

const shuffledDeck = shuffle(deck);

const showCard = ref(false);
const turnCard = () => (showCard.value = !showCard.value);

const cards = ref(1);
const drawCard = () => cards.value++;
</script>

<template>
  <main>
    <div class="cards">
      <CardComp
        v-for="card in shuffledDeck.slice(0, cards)"
        :key="card.suit + '_' + card.rank"
        :suit="card.suit"
        :rank="card.rank"
        :revealed="showCard"
      />
    </div>
    <button @click="turnCard">Turn</button>
    <button @click="drawCard">Draw</button>
  </main>
</template>

<style>
.cards {
  display: flex;
  flex-direction: row;
}
</style>
