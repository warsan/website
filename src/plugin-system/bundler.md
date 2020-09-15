---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-bundler
  title: Bundler
  order: 5
summary: "Тип плагина: превращает диаграмму активов в диаграмму комплекта."
---

Компоновщики принимают весь граф активов и модифицируют его, добавляя узлы комплекта,  
которые группируют активы в выходные пакеты.

```js
import { Bundler } from "@parcel/plugin";

export default new Bundler({
  async bundle({ graph }) {
    // ...
  },

  async optimize({ graph }) {
    // ...
  },
});
```

## Соответствующий API

{% include "../../api/bundler.html" %}
