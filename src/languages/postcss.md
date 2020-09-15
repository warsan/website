---
layout: layout.njk
eleventyNavigation:
  key: languages-postcss
  title: <img src="/assets/lang-icons/postcss.svg" alt=""/> CSS (PostCSS)
  order: 5
---

Чтобы мотивировать некоторые из следующих советов, вот обзор того, как Parcel обрабатывает файлы CSS (в указанном порядке):

- `@parcel/transformer-postcss`:
  Применяет `.postcssrc` и может генерировать карту модулей CSS
- `@parcel/transformer-css`:
  Регистрирует `@import ...` и `url(...)` в графике Parcel
- `@parcel/packager-css`:
  Объединяет все ресурсы CSS в один пакет.
- `@parcel/optimizer-cssnano`:
  Минимизирует вывод пакета из `@parcel/packager-css`.

Как видите, каждый актив обрабатывается PostCSS индивидуально и впоследствии объединяется с другими.

Parcel считывает PostCSS из этих файлов (с указанным приоритетом): `.postcssrc`, `.postcssrc.json`, `.postcssrc.js`, `postcss.config.js`.

## Модули CSS

Есть два способа включить модули CSS:

- Либо глобально в файле конфигурации PostCSS (таким образом вы также можете настроить базовый плагин `postcss-modules`).

{% sample %}
{% samplefile ".postcssrc" %}

```json/1
{
  "modules": true,
  "plugins": {
    "postcss-modules": {
      "generateScopedName": "_[name]__[local]"
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

- Или для каждого файла: с помощью расширения файла `.module.css` (или `.module.scss`, etc.).

{% sample %}
{% samplefile "app.module.css" %}

```css
.main {
  color: grey;
}
```

{% endsamplefile %}
{% samplefile "index.js" %}

```jsx/1
import { main } from "./app.module.css";

export function App() {
  return <div className={main} />;
}
```

{% endsamplefile %}
{% endsample %}

## С помощью `postcss-import`

<!-- https://github.com/parcel-bundler/parcel/issues/1165 -->

Некоторым плагинам PostCSS (например, `postcss-custom-properties`) потенциально необходим доступ к объявлениям из других CSS-ресурсов, импортируемых через `@import`ed CSS.

Если вы хотите запустить PostCSS для всего пакета, мы рекомендуем использовать `postcss-import` (встроить `@imports`) и `postcss-url` (переписать в `url(...)` соответствующие выражения).

По умолчанию это не делается, поскольку это может отрицательно повлиять на кеширование (Parcel больше не может повторно использовать неизмененные файлы CSS).

{% sample "index.html", "column" %}
{% samplefile "index.html" %}

```html
<link rel="stylesheet" type="text/css" href="./app.css" />
<div id="icon"></div>
```

{% endsamplefile %}

{% samplefile ".postcssrc" %}

```json
{
  "plugins": {
    "postcss-import": {},
    "postcss-url": {},
    "postcss-custom-properties": {}
  }
}
```

{% endsamplefile %}
{% samplefile "app.css" %}

```css
@import "./config/index.css";

html {
  background-color: var(--varColor);
}
.icon {
  width: 50px;
  height: 50px;
  background-image: var(--varIcon);
}
```

{% endsamplefile %}
{% samplefile "config/index.css" %}

```css
:root {
  --varColor: red;
  --varIcon: url("../icon.svg");
}
```

{% endsamplefile %}
{% endsample %}
