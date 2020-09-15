---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-packager
  title: Packager
  order: 8
summary: "Тип плагина: превратите группу ресурсов в файл пакета."
---

Упаковщики определяют, как объединить различные типы активов в один комплект.

```js
import { Packager } from "@parcel/plugin";

export default new Packager({
  async package({ bundle }) {
    // ...
    return { contents, map };
  },
});
```

## Соответствующий API

{% include "../../api/packager.html" %}
