---
layout: layout.njk
eleventyNavigation:
  key: recipes-react
  title: <img src="/assets/lang-icons/react.svg" alt=""/> React
  order: 6
---

По сравнению с Webpack парадигма Parcel заключается в использовании вашего HTML-файла в качестве точки входа, а не просто основного скрипта:

{% sample "parcel index.html" %}
{% samplefile "index.html" %}

```html
<div id="app"></div>
<script src="./index.js"></script>
```

{% endsamplefile %}
{% samplefile "index.js" %}

```jsx
import React from "react";
import ReactDom from "react-dom";

const App = () => <h1>Hello!</h1>;

ReactDom.render(<App />, document.getElementById("app"));
```

{% endsamplefile %}
{% endsample %}

## HMR (Быстрое обновление)

Parcel имеет первоклассную поддержку для _React Fast Refresh_ (который заменяет [react-hot-loader](https://github.com/gaearon/react-hot-loader), плагин пользовательского пространства, который испортил поддержку HMR на React). Он (в большинстве случаев) может сохранять состояние между перезагрузками (даже после ошибки).

Для получения дополнительной информации взгляните на [официальную документацию](https://reactnative.dev/docs/fast-refresh).

#### Избранные ограничения

##### Состояние компонентов класса сбрасывается между перезагрузками

Поскольку компоненты класса постепенно устаревают, их состояние не сохраняется.

##### Объявление экспорта по умолчанию с использованием выражения функции не распознаётся

Редактирование этого компонента приведет к сбросу значения, потому что плагин Fast Refresh Babel не может инструментировать декларацию экспорта по умолчанию.

{% sample %}
{% samplefile "Component.js" %}

```jsx/2
import React, { useState } from "react";

export default () => {
  const [value] = useState(Date.now());

  return <h1>Hello! {value}</h1>;
};
```

{% endsamplefile %}
{% samplefile "index.js" %}

```jsx
import React from "react";
import ReactDom from "react-dom";
import Component from "./Component.js";

const App = (
  <h1>
    <Component />
  </h1>
);

ReactDom.render(<App />, ...);
```

{% endsamplefile %}
{% endsample %}

[See also Dan Abramov's tweet](https://twitter.com/dan_abramov/status/1255229440860262400)

##### Экспорт значений, не являющихся компонентами, приведет к сбросу состояния:

Редактирование `Component` приведет к сбросу состояния `value` из-за другого экспорта, не являющегося компонентом.

{% sample %}
{% samplefile "Component.js" %}

```jsx/5,9
import React, { useState } from "react";

const Component = () => {
  const [value] = useState(Date.now());

  return <h1>Hello! {value}</h1>;
};
export default Component;

export function utility() {
  return Date.now();
}
```

{% endsamplefile %}
{% samplefile "index.js" %}

```jsx
import React from "react";
import ReactDom from "react-dom";
import Component, { utility } from "./Component.js";

console.log(utility());

const App = (
  <h1>
    <Component />
  </h1>
);

ReactDom.render(<App />, ...);
```

{% endsamplefile %}
{% endsample %}

##### Изменение актива, вызывающего рендеринг, приведёт к сбросу всего состояния:

Понятно, что изменение `App` снова вызовет `ReactDom.render`:

{% sample %}
{% samplefile "index.js" %}

```jsx/3,5
import React from "react";
import ReactDom from "react-dom";

const App = () => <h1>Hello!</h1>;

ReactDom.render(<App />, ...);
```

{% endsamplefile %}
{% endsample %}

(Функциональность HMR обеспечивается `@parcel/transformer-react-refresh-babel`, `@parcel/transformer-react-refresh-wrap` и `@parcel/runtime-react-refresh`.)
