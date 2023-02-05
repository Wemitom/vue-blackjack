<script setup lang="ts">
import { ref, watch } from "vue";
import HandComp from "./components/Hand-comp.vue";
import { deck, getCardValue, shuffle } from "./utils";
</script>

<template>
  <main class="hands">
    <HandComp
      :deck="shuffledDeck"
      :cards="hands.dealer"
      :dealer-turn="dealerTurn"
      :sum="maxSum.dealer ?? Math.max(...sums.dealer)"
      :game-status="gameStatus"
      @update:cards="addCard()"
    />
    <HandComp
      :deck="shuffledDeck"
      :cards="hands.player"
      :dealer-turn="dealerTurn"
      :sum="maxSum.player ?? Math.max(...sums.player)"
      :game-status="gameStatus"
      @update:cards="addCard(true)"
      @update:turn="changeTurn()"
      player
    />
    <button @click="reset()" v-if="gameStatus !== 'play'">Reset</button>
    Status: {{ gameStatus }}
  </main>
</template>

<script lang="ts">
const shuffledDeck = ref(shuffle(deck));

const nextCard = ref(4);
const drawCard = () => nextCard.value++;

const gameStatus = ref<"play" | "win" | "lose" | "draw">("play");

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
      getCardValue(shuffledDeck.value[cardInd].rank)
    )),
  { deep: true, immediate: true }
);
watch(
  () => hands.value.player,
  (newCards) =>
    (cardValues.value.player = newCards.map((cardInd) =>
      getCardValue(shuffledDeck.value[cardInd].rank)
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

const dealerHit = ref(0);
const dealerTurn = ref(false);
const changeTurn = () => (dealerTurn.value = true);
watch([dealerTurn, dealerHit], () => {
  console.log("aa");
  if (dealerTurn.value) {
    if (maxSum.value.dealer && maxSum.value.dealer < 17) {
      addCard();
    } else {
      if (
        (maxSum.value.player === 21 && maxSum.value.dealer !== 21) ||
        (maxSum.value.player &&
          (!maxSum.value.dealer || maxSum.value.player > maxSum.value.dealer))
      ) {
        gameStatus.value = "win";
      } else if (
        (maxSum.value.dealer === 21 && maxSum.value.player !== 21) ||
        (maxSum.value.dealer &&
          (!maxSum.value.player ||
            maxSum.value.player < maxSum.value.dealer)) ||
        !maxSum.value.player
      ) {
        gameStatus.value = "lose";
      } else if (maxSum.value.dealer === maxSum.value.player) {
        gameStatus.value = "draw";
      }
    }
  }
});

const maxSum = ref<{ dealer?: number; player?: number }>({
  dealer: 0,
  player: 0,
});
watch(
  () => sums.value.dealer,
  (newSums) =>
    (maxSum.value.dealer =
      newSums.filter((v) => v <= 21)[0] &&
      Math.max(...newSums.filter((v) => v <= 21))),
  { deep: true, immediate: true }
);
watch(
  () => sums.value.player,
  (newSums) =>
    (maxSum.value.player =
      newSums.filter((v) => v <= 21)[0] &&
      Math.max(...newSums.filter((v) => v <= 21))),
  { deep: true, immediate: true }
);

watch(
  () => maxSum.value,
  (newMaxSum) => {
    if (nextCard.value === 4) {
      if (newMaxSum.player !== newMaxSum.dealer) {
        if (newMaxSum.player === 21) {
          console.log("a");
          changeTurn();
        } else if (newMaxSum.dealer === 21) {
          gameStatus.value = "lose";
        }
      } else {
        if (newMaxSum.player === 21) {
          gameStatus.value = "draw";
        }
      }
    }
  },
  { deep: true }
);

watch(
  () => maxSum.value.player,
  (newMaxSum) => {
    if (newMaxSum === 21) {
      if (newMaxSum !== maxSum.value.dealer) {
        changeTurn();
      } else {
        gameStatus.value = "draw";
      }
    } else if (!newMaxSum) {
      changeTurn();
    }
  },
  { deep: true }
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

const reset = () => (gameStatus.value = "play");
watch(gameStatus, (newStatus) => {
  if (newStatus === "play") {
    dealerTurn.value = false;
    nextCard.value = 4;
    hands.value = {
      dealer: [2, 3],
      player: [0, 1],
    };
    shuffledDeck.value = shuffle(deck);
  }
});
</script>

<style>
.hands {
  display: flex;
  flex-direction: column;
  gap: 2rem;
  align-items: center;
}
</style>
