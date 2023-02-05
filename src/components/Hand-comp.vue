<script setup lang="ts">
import type { Card } from "@/utils";
import CardComp from "./Card-comp.vue";
import { ref } from "vue";

const props = defineProps<{
  deck: Card[];
  cards: number[];
  dealerTurn: boolean;
  sum: number;
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
        :revealed="player || cardInd === 2 || dealerTurn ? true : showCard"
      />
    </div>
    <div role="menu" v-if="player">
      <button @click="$emit('update:turn')">Stand</button>
      <button @click="$emit('update:cards')">Hit</button>
    </div>
    <p v-if="player || dealerTurn ? true : showCard">Sum: {{ sum }}</p>
  </div>
</template>

<script lang="ts">
const showCard = ref(false);
const turnCard = () => (showCard.value = !showCard.value);
</script>

<style>
.cards {
  display: flex;
  flex-direction: row;
  width: 80%;
}
</style>
