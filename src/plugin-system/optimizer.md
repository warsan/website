---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-optimizer
  title: Optimizer
  order: 9
summary: "Тип плагина: внесите изменения в готовый комплект"
---

Оптимизаторы похожи на преобразователи, но они принимают пакет вместо одного ресурса. На этом этапе все AST уже сериализованы.

```js
import { Optimizer } from "@parcel/plugin";

export default new Optimizer({
  async optimize({ bundle, contents, map }) {
    let result = minifyCode(contents, map);
    return { contents: result.contents, map: result.contents };
  },
});
```

## Соответствующий API

{% include "../../api/optimizer.html" %}
