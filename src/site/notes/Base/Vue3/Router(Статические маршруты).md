---
dg-publish: true
---
`router/router.js`

`const routes` - массив содержащий маршруты к компонентам
```js
import About from '@/Page/About.vue';
import MainPage from '@/Page/MainPage.vue';
import PostPage from '@/Page/PostPage.vue';
import { createRouter, createWebHistory } from 'vue-router';
const routes = [
  {
    path: '/',
    component: MainPage,
  },
  {
    path: '/post',
    component: PostPage,
  },
  {
    path: '/about',
    component: About,
  },
];

const router = createRouter({
  routes,
  history: createWebHistory(process.env.BASE_URL),
});

export default router;
```

Регистрация `router`

```js
import { createApp } from 'vue';
import App from './App.vue';
import components from './components/UI';
import router from './router/router';

const app = createApp(App);

components.forEach((component) => {
  app.component(component.name, component);
});


app.use(router);//Регистрация

app.mount('#app');
```

Чтобы роутер в зависимости от строки запроса мог отрисовывать соответствующие страницы нужно добавить компонет `router-view`
```js
<template>
  <NavBar> </NavBar>
  <div class="app">
    <router-view> </router-view>
  </div>
</template>
```
Именно в него будут встраиваться компоненты, которые мы указывали в `const routes`

Переход по этим компонентам можно выполнять с помощью `router-link`
```js
<template>
  <div>
    <h1>Главна страница</h1>
    <RouterLink to="/post">Посты</RouterLink>
    <RouterLink to="/about">О программе</RouterLink>
  </div>
</template>
```

либо же с помощью `$router.push`:
```js
<template>
  <div class="navbar">
    <div v-on:click="$router.push('/')">Главная</div>
    <div class="navbar__btns">
      <my-button v-on:click="$router.push('/post')"> Посты </my-button>
      <my-button
        style="margin-left: 20px"
        v-on:click="$router.push('/about')"
      >
        О программе
      </my-button>
    </div>
  </div>
</template>
```
