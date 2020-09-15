---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-reporter
  title: Reporter
  order: 10
summary: "Тип плагина: прослушивание событий сборки"
---

Репортеры получают события по мере их возникновения и могут выводить их в stdout/stderr,
или выполнить другие действия.

```js
import {Reporter} from '@parcel/plugin';

export default new Reporter({
  async report({ event: { type, ... } }) {
    // ...
  }
});
```

## Соответствующий API

{% include "../../api/reporter.html" %}
