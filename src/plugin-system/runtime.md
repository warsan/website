---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-runtime
  title: Runtime
  order: 7
summary: "Тип плагина: программно вставляйте ресурсы из воздуха в пакеты."
---

Среды выполнения принимают пакет и возвращают ресурсы для вставки в этот пакет.

```js
import { Runtime } from "@parcel/runtime";

export default new Runtime({
  async apply({ bundle, bundleGraph }) {
    // ...
    return assets;
  },
});
```

## Соответствующий API

{% include "../../api/runtime.html" %}
