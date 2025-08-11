---
dg-publish: true
---
![Анимация.gif](/img/user/Files/Image/%D0%90%D0%BD%D0%B8%D0%BC%D0%B0%D1%86%D0%B8%D1%8F.gif)

Часть переиспользуемого функционала может быть вынесена в отдельный миксин(данные,методы).

При использование миксина происходит слияние с компонентом и все методы, пропсы, события определённые в этом миксине становятся доступными в компоненте.

Создаём
```js
// minins/toggleMixin.js
export default {
  data() {
    return {};
  },

  props: {
    show: {
      type: Boolean,
      default: false,
    },
  },

  methods: {
      hideDialog() {
        this.$emit('update:show', false);
      },
  },

  mounted() {},
};
```

Использование
```js
<script>
import toggleMixin from '@/mixins/toggleMixin';
export default {
  name: 'my-dialog',
  mixins: [toggleMixin],
};

</script>
```