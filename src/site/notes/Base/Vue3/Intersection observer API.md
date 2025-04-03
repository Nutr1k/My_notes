---
{"dg-publish":true,"permalink":"/base/vue3/intersection-observer-api/"}
---

Определение когда пользователь долистал страницу.

Следить мы будет за блоком:
```js
  <div
    ref="observer"
    class="observer"
  ></div>
```

Чтобы его было видно сделаем его зелёным
```js
  .observer{
    height: 30px;
    background: green;
  }
```

Отслеживание:
```js
 mounted() {
    var options = {
      rootMargin: '0px',
      threshold: 1.0,
    };
    var callback = (entries, observer) => {
      if (entries[0].isIntersecting && this.page < this.totalPage) {
        console.log('Пересечение');
        this.loadMorePost();
      }
    };
    var observer = new IntersectionObserver(callback, options);
    observer.observe(this.$refs.observer);
  },
```