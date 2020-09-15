---
layout: layout.njk
eleventyNavigation:
  key: features-code-splitting
  title: ✂️ Code Splitting
  order: 3
summary: Загружайте код так, как вам нужно (если вам это нужно)
---

Parcel поддерживает разделение кода нулевой конфигурации из коробки. Это позволяет разделить код приложения на отдельные пакеты, которые могут быть загружены по требованию, что означает меньшие начальные размеры пакетов и более быстрое время загрузки. Поскольку пользователь перемещается по вашему приложению и требуются модули, вы можете загружать дочерние пакеты по требованию.

Code splitting is controlled by use of the dynamic `import()` syntax, which works like a hybrid of the normal `import` statement and the `require` function, but returns a Promise. This means that the module can be loaded asynchronously.

Разделение кода контролируется с помощью динамического синтаксиса `import()`, который работает как гибрид обычного оператора `import` и функции `require`, но возвращает обещание. Это означает, что модуль может быть загружен асинхронно.

## Использование динамического импорта

В следующем примере показано, как можно использовать динамический импорт для загрузки подстраницы приложения по требованию.

{% sample %}
{% samplefile %}

```js
import("./pages/about").then(function (page) {
  // Страница рендеринга
  page.render();
});
```

{% endsamplefile %}
{% samplefile "pages/about.js" %}

```js
export function render() {
  // Отобразить страницу
}
```

{% endsamplefile %}
{% endsample %}

## Динамический импорт с помощью async/await

Поскольку функция `import ()` возвращает обещание,вы также можете использовать синтаксис async/await.

{% sample %}
{% samplefile %}

```js
async function load() {
  const page = await import("./pages/about");
  // Страница рендеринга
  page.render();
}
load();
```

{% endsamplefile %}
{% samplefile "pages/about.js" %}

```js
export function render() {
  // Render the page
}
```

{% endsamplefile %}
{% endsample %}

## "Интернализованные" Пакеты

Если ресурс импортируется как синхронно, так и асинхронно, то нет смысла создавать фактический асинхронный пакет (поскольку модуль все равно уже загружен).

В этой ситуации Parcel вместо этого превращает `import("foo")` в `Promise.resolve(require("foo"))`. Поэтому в более крупной сборке вы должны думать о dynamic/async импорте как о "Мне не нужен этот импорт синхронно", а не "Это станет новой связкой".

## Общие Пакеты

Во многих ситуациях (например, когда две HTML-записи с JavaScript `<script>` используют актив(ы) или когда два динамических импорта имеют общие активы) Parcel разбивает их на отдельный родственный пакет ("общий" пакет), чтобы минимизировать дублирование кода.

Параметры, когда это происходит, можно настроить в package.json:

{% sample %}
{% samplefile "package.json" %}

```json5
{
  "name": "my-project",
  "dependencies": {
    ...
  },
  "@parcel/bundler-default": {
    "minBundles": 1,
    "minBundleSize": 3000,
    "maxParallelRequests": 20
  }
}
```

{% endsamplefile %}
{% endsample %}

Параметры:

- **minBundles**: чтобы актив был разделен, он должен использоваться более чем пакетами `minBundles`
- **minBundleSize**: для создания разделяемого бандла он должен быть как минимум в байтах `minBundleSize` (до минификации/treehaking)
- **maxParallelRequests**: Чтобы предотвратить перегрузку сети слишком большим количеством одновременных запросов, это гарантирует, что данный пакет может иметь только одноуровневые пакеты `maxParallelRequests - 1` (которые были загружены вместе с фактическим пакетом).

- **http**: Это сокращение для установки вышеуказанных значений по умолчанию, которые оптимизированы для HTTP/1 или HTTP/2:

| HTTP version `version` | `minBundles` | `minBundleSize` | `maxParallelRequests` |
| ---------------------- | ------------ | --------------- | --------------------- |
| 1                      | 1            | 30000           | 6                     |
| 2 (default)            | 1            | 20000           | 25                    |

Вы можете прочитать больше об этой теме здесь на [web.dev](https://web.dev/granular-chunking-nextjs/)

<!--

## Пакетное разрешение

TODO ???

Parcel автоматически определяет местонахождение пакетов. Это делается в [bundle-url](https://github.com/parcel-bundler/parcel/blob/master/packages/core/parcel-bundler/src/builtins/bundle-url.js) модуле и использует трассировку стёка для определения пути, по которому был загружен исходный пакет.

Это означает, что вам не нужно настраивать, откуда должны быть загружены пакеты, но также означает, что вы должны обслуживать пакеты из одного и того же места.

В настоящее время Parcel разрешает пакеты по следующим протоколам: `http`, `https`, `file`, `ftp`, `chrome-extension` и `moz-extension`.

-->
