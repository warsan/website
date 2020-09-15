---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-resolver
  title: Resolver
  order: 4
summary: "Тип плагина: превратить запросы зависимостей в абсолютные пути (или исключить их)"
---

Резольверы вызываются с запросом актива (состоящим из пути к исходному файлу
и спецификатор того, что запрашивается), который затем пытается
разрешить. Если преобразователь не уверен, как обрабатывать запрос, он также может вернуть
`null` и передать его следующему преобразователю в цепочке.

```js
import { Resolver } from "@parcel/plugin";

export default new Resolver({
  async resolve({ filePath, dependency }) {
    if (!shouldHandle(filePath)) {
      return null;
    }
    // ...
    return {
      filePath: doResolve({ from: filePath, to: dependency.moduleSpecifier }),
    };
  },
});
```

The [result object](/plugin-system/api/#ResolveResult) can also contain `sideEffects` (which corresponds to `package.json#sideEffects`) `code` (used instead of `fs.readFile(filePath)`) and `isExcluded` (e.g. to exclude `node_modules`).

[Объект результата](/plugin-system/api/#ResolveResult) также может содержать `sideEffects` (который соответствует `package.json#sideEffects`) `code` (используется вместо `fs.readFile(filePath)`) и `isExcluded` (например, чтобы исключить `node_modules`).

## Соответствующий API

{% include "../../api/resolver.html" %}
