---
layout: layout.njk
eleventyNavigation:
  key: languages-vue
  title: <img src="/assets/lang-icons/vue.svg" alt=""/> Vue
  order: 4
---

Обратите внимание, что Parcel не поддерживает использование SFC с Vue 2, вы должны использовать [Vue 3 beta](https://github.com/vuejs/vue-next) или более позднюю.

Vue.js представляет собой прогрессивную, постепенно адаптируемую среду JavaScript для создания пользовательского интерфейса в Интернете. Parcel поддерживает Vue без необходимости дополнительной настройки.

{% sample %}
{% samplefile "index.html" %}

```html
<div id="app"></div>
<script src="./index.js"></script>
```

{% endsamplefile %}
{% samplefile "index.js" %}

```jsx
import { createApp } from "vue";
import App from "./App";

const app = createApp(App);
app.mount("#app");
```

{% endsamplefile %}
{% samplefile "App.vue" %}

```html
<template>
  <div>- Привет, {% raw %}{{ name }}{% endraw %}!</div>
</template>

<script>
  export default {
    data() {
      return {
        name: "Vue",
      };
    },
  };
</script>
```

{% endsamplefile %}
{% endsample %}

## HMR

Parcel использует ту же инъекцию HMR, что и официальный [vue-loader](https://github.com/vuejs/vue-loader) для Webpack, поэтому у вас будет быстрая реактивная разработка.

## Характеристики Vue 3 

Поскольку Parcel использует последнюю бета-версию Vue 3, вы можете использовать все функции Vue 3, такие как [Composition API](https://composition-api.vuejs.org/).

{% sample %}
{% samplefile "App.vue" %}

```html
<template>
  <button @click="increment">
    Count is: {% raw %}{{ state.count }}{% endraw %} Double is: {% raw %}{{
    state.double }}{% endraw %}
  </button>
</template>

<script>
  import { reactive, computed } from "vue";

  export default {
    setup() {
      const state = reactive({
        count: 0,
        double: computed(() => state.count * 2),
      });

      function increment() {
        state.count++;
      }

      return {
        state,
        increment,
      };
    },
  };
</script>
```

{% endsamplefile %}
{% endsample %}

## Language Support

Parcel поддерживает [JavaScript](/languages/babel), [TypeScript](/languages/typescript), и [CoffeeScript](/languages/coffeescript) в качестве скриптовых языков в Vue.

Практически любой шаблонный язык (все те, которые поддерживаются [консолидацией](https://www.npmjs.com/package/consolidate)) может быть использован.

Для укладки поддерживаются [Less](/languages/less), [Sass](/languages/sass), и [Stylus](/languages/stylus). Кроме того, с модификаторами [CSS Modules](/languages/postcss) и [scoped style](https://vue-loader.vuejs.org/guide/scoped-css.html) можно использовать с помощью модификаторов `module` и `scoped`.

{% sample %}
{% samplefile "App.vue" %}

```html
<style lang="scss" scoped>
  /* Этот стиль будет применяться только к этому модулю */
  $red: red;
  h1 {
    background: $red;
  }
</style>

<style lang="less">
  @green: green;
  h1 {
    color: @green;
  }
</style>

<style src="./App.module.css">
  /* Содержимое блоков с атрибутом src игнорируется и заменяется на
   содержание `src`. */
</style>

<template lang="pug">
div
  h1 This is the app
</template>

<script lang="coffee">
  module.exports =
    data: ->
      msg: 'Привет из coffee!'
</script>
```

{% endsamplefile %}
{% endsample %}

## Пользовательские блоки

Вы можете использовать пользовательские блоки в своих компонентах Vue, но должны настроить Vue с `.vuerc`, `vue.config.js`, и т.д., чтобы определить, как вы будете предварительно обрабатывать эти блоки.

{% sample %}
{% samplefile ".vuerc" %}

```json
{
  "customBlocks": {
    "docs": "./src/docs-preprocessor.js"
  }
}
```

{% endsamplefile %}
{% samplefile "src/docs-preprocessor.js" %}

```js
export default function (component, blockContent, blockAttrs) {
  if (blockAttrs.brief) {
    component.__briefDocs = blockContent;
  } else {
    component.__docs = blockContent;
  }
}
```

{% endsamplefile %}
{% samplefile "HomePage.vue" %}

```html
<template>
  <div>Домашняя страница</div>
</template>

<docs>
  Этот компонент представляет собой домашнюю страницу приложения.
</docs>

<docs brief>
  Домашняя страница
</docs>
```

{% endsamplefile %}
{% samplefile "App.vue" %}

```html
<template>
  <div>
    <child></child>
    docs: {% raw %}{{ docs.standard }}{% endraw %} in brief: {% raw %}{{
    docs.brief }}{% endraw %}
  </div>
</template>

<script>
  import Child from "./HomePage";
  export default {
    components: {
      child: Child,
    },
    data() {
      let docs = { standard: Child.__docs, brief: Child.__docsBrief };
      return { docs };
    },
  };
</script>
```

{% endsamplefile %}
{% endsample %}
