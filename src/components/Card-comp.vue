<script setup lang="ts">
import { ref, watch } from "vue";

const props = defineProps<{ suit: Suits; rank: Ranks; revealed: Boolean }>();

const revealCardClass = ref("");
const revealCardAsync = () =>
  setTimeout(
    () => (revealCardClass.value = props.revealed ? "card-revealed" : ""),
    300
  );

watch(() => props.revealed, revealCardAsync, { deep: true, immediate: true });
</script>

<template>
  <div class="card">
    <div class="card-inner" :class="revealCardClass">
      <div class="card-front">
        <img src="../assets/red.svg" width="140" height="200" />
      </div>
      <div class="card-back">
        <img :src="cardSuit" width="140" height="200" />
      </div>
    </div>
  </div>
</template>

<script lang="ts">
export type Suits = "clubs" | "hearts" | "spades" | "diamonds";
export type Ranks =
  | "2"
  | "3"
  | "4"
  | "5"
  | "6"
  | "7"
  | "8"
  | "9"
  | "10"
  | "jack"
  | "king"
  | "queen"
  | "ace";

export default {
  computed: {
    cardSuit(): string {
      return new URL(
        "../assets/fronts/" + this.suit + "_" + this.rank + ".svg",
        import.meta.url
      ).href;
    },
  },
};
</script>

<style scoped>
.card {
  perspective: 1000px;
  width: 140px;
  height: 200px;
}

.card-inner {
  position: relative;
  width: 100%;
  height: 100%;
  transition: transform 0.5s;
  transform-style: preserve-3d;
}

.card-back,
.card-front {
  position: absolute;
  backface-visibility: hidden;
  width: 100%;
  height: 100%;
}

.card-revealed {
  transform: rotateY(180deg);
}

.card-back {
  transform: rotateY(180deg);
}
</style>
