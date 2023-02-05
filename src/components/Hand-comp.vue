<script setup lang="ts">
import type { Card } from "@/utils";
import CardComp from "./Card-comp.vue";

defineProps<{
  deck: Card[];
  cards: number[];
  dealerTurn: boolean;
  sum: number;
  gameStatus: "play" | "win" | "lose" | "draw";
  player?: boolean;
}>();
defineEmits(["update:cards", "update:turn"]);
</script>

<template>
  <div>
    <div class="cards">
      <CardComp
        v-for="cardInd in cards"
        :key="deck[cardInd].suit + '_' + deck[cardInd].rank"
        :suit="deck[cardInd].suit"
        :rank="deck[cardInd].rank"
        :revealed="player || cardInd === 2 || (gameStatus !== 'play' && true)"
      />
    </div>
    <div role="menu" v-if="player && gameStatus === 'play'">
      <button @click="$emit('update:turn')">Stand</button>
      <button @click="$emit('update:cards')">Hit</button>
    </div>
    <p v-if="player || (gameStatus !== 'play' && true)">Sum: {{ sum }}</p>
  </div>
</template>

<style>
.cards {
  display: flex;
  flex-direction: row;
  width: 80%;
}
</style>
