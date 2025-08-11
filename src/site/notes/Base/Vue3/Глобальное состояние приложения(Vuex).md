---
dg-publish: true
---
![Pasted image 20250131184016.png](/img/user/Files/Image/Pasted%20image%2020250131184016.png)
https://vuex.vuejs.org/api/#actions
https://v3.vuex.vuejs.org/ru/guide/mutations.html
Левая часть: передача данный по цепочке вверх или вниз до родительского компонента
Правая часть: через глобальное состояние



```js
//store/postModule.js
import { createStore } from 'vuex';
export const postModule = {
  state: { //state - это функция
    //Описываем данные, которые будут в нашем приложение
    like: 1,
    isAuth: false,
  },
  getters: {
    //Аналог compute свойств
    //Результаты геттера кэшируются, на основе его зависимостей и пересчитываются только при изменении одной из зависимостей.
    doubleLikes(state) {
      return state.like * 2;
    },
  },


  mutations: {
    //Состояния можно менять с помощью мутаций т.к. изменять состояние напрямую мы не можем. 

    incrementLikes(state) {
      state.like += 1;
    },

    decrementLikes(state) {
      state.like -= 1;
    },
  },

  actions: {
    //Для вызова мутаций,для изменения состояний
  },

  modules: {
    //Изолированный кусочек состояния. Со своими  getters, mutations,actions
  },
  namespaced: true, //Указываем, что нужно использовать пространство имён
};
```

```js
//store/index.js
import { createStore } from 'vuex';
import { postModule } from './postModule';

export default createStore({
  state: {//Явно глобальные данные
    isAuth: false,
  },
  modules: {
    //Изолированный кусочек состояния. Со своими  getters, mutations,actions
    post: postModule,
  },
});
```

```js
<template>
  <h1>{{ $store.state.isAuth }}</h1>
  <h1>{{ $store.state.post.limit }}</h1>
</template>
```

- Мутация **—** единственный **способ** изменить состояние
- Мутация не заботится о бизнес-логике, ее волнует только «состояние **»**
- Действие **—** это бизнес-логика
- Действие может **совершать** более одной мутации за раз, оно просто реализует бизнес-логику, его не волнует изменение данных (которое управляется мутацией) **.**
- Мутации синхронны, тогда как действия могут быть асинхронными.

Подключаем
```js
//main.js
import store from './store';
app.use(store);
```

Используем
```js
<h1>
  {{
	$store.state.isAuth
	  ? 'Пользователь авторизован'
	  : 'Пользователь не авторизован'
  }}
</h1>
<h1>Like {{ $store.state.like }}</h1>
<h1>DoubleLikeGetter {{ $store.getters.doubleLikes }}</h1>
 <my-button v-on:click="$store.commit('incrementLikes')">Лайк</my-button>
 <my-button v-on:click="$store.commit('decrementLikes')">Дизлайк</my-button>
</div>
```

Если мы задали `namespace:true`
```js
<h1>{{ console.log($store.state.post.limit) }}</h1>
```
Упрощение процесса использования:

