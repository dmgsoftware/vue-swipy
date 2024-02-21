# Vue-Swipy

Vue swipeable component for Tinder-like apps.

🚀 [See live demo (without back button)](https://shard-labs.github.io/vue-swipy/)

## Installation

```bash
npm install vue-swipy
```

## Usage (with back button)

```html
<template>
  <div id="app">
    <div id="swipeable-container">
      <Swipeable
        v-for="(card, index) in cards"
        :key="card.id"
        v-on:swipe="onSwipe"
        :ref="'swipeable' + index"
        :style="{
        position: 'absolute',
        height: '100%',
        width: '100%',
        background: card.color,
        borderRadius: '8px',
        zIndex: -1 * index
      }"
      />
    </div>
    <button @click="back">Back</button>
  </div>
</template>

<script>
import { Swipeable } from "vue-swipy";

const SWIPE_NONE = "swipe-none";

function getRandomColor() {
  const letters = "0123456789ABCDEF";
  let color = "#";
  for (let i = 0; i < 6; i++) {
    color += letters[Math.floor(Math.random() * 16)];
  }
  return color;
}

export default {
  name: "App",
  components: {
    Swipeable,
  },
  data() {
    return {
      cards: [],
      currentIndex: 0
    };
  },
  mounted() {
    this.cards.push({ id: Math.random(), color: getRandomColor() });
    this.cards.push({ id: Math.random(), color: getRandomColor() });
    this.cards.push({ id: Math.random(), color: getRandomColor() });
  },
  methods: {
    onSwipe(direction) {
      console.log(direction);
      this.currentIndex++;

      if (this.cards.length < (this.currentIndex + 3)) {
        this.cards.push({ id: Math.random(), color: getRandomColor() });
      }
    },
    back() {
      if (this.currentIndex === 0) {
        console.log("Can't go back beyond 0.");
        return;
      }

      this.currentIndex--;

      const lastSwiped = this.$refs['swipeable' + this.currentIndex][0];
      lastSwiped.setStatus(SWIPE_NONE);
    }
  },
};
</script>

<style>
body {
  margin: 0;
}
#app {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: 500px;
  width: 100vw;
}
#swipeable-container {
  position: relative;
  height: 400px;
  width: 250px;
  margin: 2rem;
  z-index: 0;
}
</style>
```
