<script setup lang="ts">
defineProps<{ suit: Suits; rank: Ranks; revealed: Boolean }>();
</script>

<template>
  <div class="card">
    <div class="card-inner" :class="cardObject">
      <div class="card-front">
        <img src="../assets/red.svg" />
      </div>
      <div class="card-back">
        <img :src="cardSuit" />
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
    cardObject(): { "card-revealed": boolean } {
      return { "card-revealed": !!this.revealed };
    },
  },
};
</script>

<style scoped>
.card {
  perspective: 1000px;
  width: 234px;
  height: 333px;
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
