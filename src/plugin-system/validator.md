---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-validator
  title: Validator
  order: 11
summary: "Тип плагина: анализировать ресурсы и выдавать предупреждения и ошибки."
---

Валидаторы получают актив и могут выдавать ошибки, если этот актив недействителен.
каким-то образом, например ошибки типа или ошибки линтинга.

```js
import { Validator } from "@parcel/plugin";

export default new Validator({
  async validate({ asset }) {
    // ...
    throw error;
  },
});
```

Некоторые валидаторы (такие как `@parcel/validator-typescript`) могут пожелать поддерживать кеш всего проекта для повышения эффективности. В этих случаях целесообразно использовать другой интерфейс, в котором посылки передают _все_ файлы валидатору одновременно:

```js
import { Validator } from "@parcel/plugin";

export default new Validator({
  async validateAll({ assets }) {
    // ...
    throw error;
  },
});
```

Если ваш плагин реализует `validateAll`, Parcel всегда будет вызывать этот метод в одном потоке (чтобы состояние вашего кеша было доступно).

## Соответствующий API

{% include "../../api/validator.html" %}
