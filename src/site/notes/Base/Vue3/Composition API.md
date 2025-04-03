---
{"dg-publish":true,"permalink":"/base/vue3/composition-api/"}
---

```js
<template>
    <h1>{{ likes }}</h1>
    <button v-on:click="addLike">Like</button>
</template>

<script>
export default {
  setup(props) {
    const likes = ref(0);
    const addLike = () => {
      likes.value += 1;
    };
    return {
      likes,
      addLike,
    };
  },
};
</script>
```