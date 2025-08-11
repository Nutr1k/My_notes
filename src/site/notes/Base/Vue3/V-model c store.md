---
dg-publish: true
---
Т.к. при использовании глобального хранилища для установки состояний мы должны пользоваться `мутаторами(mutations)`, а v-model об этом не знает, то нужна ручная привязка через обычные пропсы

Было
```js
    <my-input
      v-model="searchQuery"
      placeholder="Поиск..."
    >
```

Стало
```js
    <my-input
      v-bind:modelValue="searchQuery"
      v-on:update:modelValue="setSearchQuery"
      placeholder="Поиск..."
    >
```

`my-input` принимает `props modelValue` и генерирует событие `update:modelValue`


`searchQuery` - состояние
```js
  computed: {
    ...mapState({
      searchQuery: (state) => state.post.searchQuery,
      sortOptions: (state) => state.post.sortOptions,
    }),
```

`setSearchQuery` - мутатор
```js
...mapMutations({
  setSearchQuery: 'post/setSearchQuery',
  setPage: 'post/setPage',
}),
```