<script setup lang="ts">
import { ref, watch } from "vue";
import HandComp from "./components/Hand-comp.vue";
import { deck, getCardValue, shuffle } from "./utils";
</script>

<template>
  <main>
    <HandComp
      :deck="shuffledDeck"
      :cards="hands.dealer"
      :dealer-turn="dealerTurn"
      :sum="maxSum.dealer ?? Math.max(...sums.dealer)"
      @update:cards="addCard()"
    />
    <HandComp
      :deck="shuffledDeck"
      :cards="hands.player"
      :dealer-turn="dealerTurn"
      :sum="maxSum.player ?? Math.max(...sums.player)"
      @update:cards="addCard(true)"
      @update:turn="changeTurn()"
      player
    />
  </main>
</template>

<script lang="ts">
const shuffledDeck = shuffle(deck);

const nextCard = ref(4);
const drawCard = () => nextCard.value++;

const hands = ref<{ dealer: number[]; player: number[] }>({
  dealer: [2, 3],
  player: [0, 1],
});
const addCard = (toPlayer?: boolean) => {
  if (toPlayer) {
    hands.value.player.push(nextCard.value);
  } else {
    hands.value.dealer.push(nextCard.value);
  }
  drawCard();
};

const cardValues = ref<{
  dealer: (number | number[])[];
  player: (number | number[])[];
}>({
  dealer: [],
  player: [],
});
watch(
  () => hands.value.dealer,
  (newCards) =>
    (cardValues.value.dealer = newCards.map((cardInd) =>
      getCardValue(shuffledDeck[cardInd].rank)
    )),
  { deep: true, immediate: true }
);
watch(
  () => hands.value.player,
  (newCards) =>
    (cardValues.value.player = newCards.map((cardInd) =>
      getCardValue(shuffledDeck[cardInd].rank)
    )),
  { deep: true, immediate: true }
);

const sums = ref({
  dealer: [0],
  player: [0],
});
const calculateSums = (sum: number[], val: number | number[]) => {
  if (typeof val === "number") {
    sum = sum.map((s) => s + val);
  } else {
    sum.push(...sum);
    val.map((num, index) => {
      sum = sum.map((s, i) => {
        if (!index) {
          s = i < sum.length / 2 ? s + num : s;
        } else {
          s = i < sum.length / 2 ? s : s + num;
        }

        return s;
      });
    });
  }

  return sum;
};

const maxSum = ref<{ dealer?: number; player?: number }>({
  dealer: 0,
  player: 0,
});
watch(
  () => sums.value.dealer,
  () =>
    (maxSum.value.dealer =
      sums.value.dealer.filter((v) => v <= 21)[0] &&
      Math.max(...sums.value.dealer.filter((v) => v <= 21))),
  { deep: true, immediate: true }
);
watch(
  () => sums.value.player,
  () =>
    (maxSum.value.player =
      sums.value.player.filter((v) => v <= 21)[0] &&
      Math.max(...sums.value.player.filter((v) => v <= 21))),
  { deep: true, immediate: true }
);

const dealerHit = ref(0);
const dealerTurn = ref(false);
const changeTurn = () => (dealerTurn.value = true);
watch(
  [dealerTurn, dealerHit],
  () => {
    if (maxSum.value.dealer && maxSum.value.dealer < 17) {
      addCard();
    }
  },
  { flush: "post" }
);

// Cos card value can be either number or number[](ace) we get an
// array of sums and then display the max one if it`s also <= 21
watch(
  () => cardValues.value.dealer,
  (newCardValues) => {
    sums.value.dealer = newCardValues.reduce(calculateSums, [0]);
    if (dealerTurn.value) {
      dealerHit.value++;
    }
  },

  { deep: true, immediate: true }
);
watch(
  () => cardValues.value.player,
  (newCardValues) =>
    (sums.value.player = newCardValues.reduce(calculateSums, [0])),
  { deep: true, immediate: true }
);
</script>
