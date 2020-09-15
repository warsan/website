---
layout: layout.njk
eleventyNavigation:
  key: getting-started-migration
  title: 🚚 Migration
  order: 5
summary: Несколько советов по переходу с Parcel 1 на Parcel 2
---

По большей части, вам не придется много менять при обновлении до Parcel 2:

## Изменения кода

### Импорт ресурсов, не связанных с кодом из Javascript

Если вы хотите импортировать URL-адрес изображения (или звукового файла и т. Д.) Из Javascript, вам необходимо добавить `url:` к спецификатору модуля (подробнее об именованных конвейерах см. [Plugin Configuration](</configuration/plugin-configuration/#predefined-(offical)-named-pipelines>))

{% migration %}
{% samplefile "index.js" %}

```js/0
import logo from "./logo.svg";

document.body.innerHTML = `<img src="${logo}">`;
```

{% endsamplefile %}
{% samplefile %}

```js/0
import logo from "url:./logo.svg";

document.body.innerHTML = `<img src="${logo}">`;
```

{% endsamplefile %}
{% endmigration %}

В качестве альтернативы вы можете использовать собственный файл `.parcelrc`, чтобы выбрать старое поведение (добавьте дополнительные расширения ресурсов, если вы их используете):

{% migration %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{jpg,png,svg}": ["@parcel/transformer-raw"]
  }
}
```

{% endsamplefile %}
{% endmigration %}

### Typescript

Посылка 1 транслировала TypeScript с помощью `tsc` (официальный компилятор TypeScript). Вместо этого в Parcel 2 по умолчанию используется Babel (с использованием `@babel/preset-env`). Это имеет два важных последствия:

(Этот [TypeScript page](/languages/typescript) содержит дополнительную информацию - и ограничения - обработки Parcel TypeScript.)

###### `@babel/preset-typescript` не вставляется автоматически в пользовательский `.babelrc`.

В большинстве случаев транспиляции с использованием Babel достаточно, поэтому Parcel включает `@babel/preset-typescript` в свою конфигурацию Babel по умолчанию для ресурсов TypeScript. Однако, если вы используете собственный файл `.babelrc`, вам нужно указать его вручную:

{% migration %}
{% samplefile ".babelrc" %}

```json/1
{
  "presets": ["@babel/preset-env"],
  "plugins": ["babel-plugin-foo"]
}
```

{% endsamplefile %}
{% samplefile ".babelrc" %}

```json/1
{
  "presets": ["@babel/preset-env", "@babel/preset-typescript"],
  "plugins": ["babel-plugin-foo"]
}
```

{% endsamplefile %}
{% endmigration %}

###### Babel ре читает `tsconfig.json`

Если Babel у вас не работает (например, из-за расширенного `tsconfig.json`), вы можете использовать `tsc`:

{% migration %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"]
  }
}
```

{% endsamplefile %}
{% endmigration %}

{% warning %}
Ожидается, что это будет немного медленнее для больших сборок/ресурсов, поэтому транспиляция с использованием Babel является подходом по умолчанию.
{% endwarning %}

### Импорт GraphQL

При импорте файлов GraphQL (`.gql`), импорт все еще разрешен/встроен (с помощью `graphql-import-macro`), но теперь вы получаете обработанный запрос GraphQL в виде строки вместо Apollo AST.

{% migration %}
{% samplefile "DataComponent.js" %}

```js
import fetchDataQuery from "./fetchData.gql"; // fetchDataQuery - это проанализированный AST

const DataComponent = () => {
  const { data } = useQuery(test, {
    fetchPolicy: "cache-and-network",
  });

  // ...
};
```

{% endsamplefile %}
{% samplefile "DataComponent.js" %}

```js/5
import gql from "graphql-tag";

import fetchDataQuery from "./fetchData.gql"; // fetchDataQuery - это строка

// Преобразование в AST запроса Apollo
const parsedFetchDataQuery = gql(test);

const DataComponent = () => {
  const { data } = useQuery(parsedFetchDataQuery, {
    fetchPolicy: "cache-and-network",
  });

  // ...
};
```

{% endsamplefile %}
{% endmigration %}

{% note %}
С новой архитектурой плагинов Parcel 2 очень легко создать плагин, который анализирует строку в AST во время сборки (как это сделал Parcel 1).
{% endnote %}

## Конфигурация/CLI

### `package.json#main`

Многие `package.json` '(например, тот, который сгенерирован `npm init`) содержат `main: "index.js"`, который игнорируется большинством инструментов (для небиблиотечных проектов). Однако Parcel 2 будет использовать это значение в качестве пути вывода (см. [Configuration#main](/configuration/package-json/#main-%2F-module-%2F-browser)),
для большинства веб-приложений эту строку следует просто удалить.

### `--target`

Этот флаг CLI теперь выводится из вашего `package.json`, достаточно одного из этих трех свойств (число обозначает приоритет).

{% migration %}
{% samplefile %}

```bash
parcel build index.js --target node
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/3,5,10
{
  targets: {
    default: {
      context: "node", // <--- (1)
      engines: {
        node: "10", // <------ (2)
      },
    },
  },
  engines: {
    node: "10", // <---------- (3)
  },
}
```

{% endsamplefile %}
{% endmigration %}

### `--experimental-scope-hoisting`

В Parcel 2 по умолчанию включен подъем области действия; чтобы отключить его, добавьте `--no-scope-hoist`.

{% migration %}
{% samplefile %}

```bash
parcel build index.js --experimental-scope-hoisting
parcel build index.js
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.js
parcel build index.js --no-scope-hoist
```

{% endsamplefile %}
{% endmigration %}

### `--bundle-node-modules`

Чтобы связать пакеты из `node_modules` при таргетинге на Node.js, вы теперь должны указать это в целевой конфигурации:

{% migration %}
{% samplefile %}

```bash
parcel build index.js --target node --bundle-node-modules
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/8
{
  "default": "dist/index.js"
  "targets": {
    "default": {
      "includeNodeModules": true
    }
  },
  "engines": {
    "node": "10"
  }
}
```

{% endsamplefile %}
{% endmigration %}

{% note %}

Этот параметр более универсален, чем параметр CLI (вы также можете выборочно включать пакеты), все подробности см. В [Configuration#includeNodeModules](/configuration/package-json/#includenodemodules).

{% endnote %}

### `--out-dir`

To align `--out-dir` with the options in [`package.json#targets`](/configuration/package-json/#targets), that option was renamed to `--dist-dir`.

Чтобы согласовать `--out-dir` с параметрами в [`package.json#targets`](/configuration/package-json/#targets), этот параметр был переименован в `--dist-dir`.

{% migration %}
{% samplefile %}

```bash
parcel build index.html --out-dir www
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.html --dist-dir www
```

{% endsamplefile %}
{% endmigration %}

### `--out-file`

Этот флаг был удален, и вместо этого путь должен быть указан в `package.json` (см. [Configuration](/configuration/package-json/#custom-targets)).

{% migration %}
{% samplefile %}

```bash
parcel build index.js --out-file lib.js
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json5/1
{
  default: "lib.js",
  // ...
}
```

{% endsamplefile %}
{% endmigration %}

### `--log-level`

Уровни журнала теперь имеют имена, а не числа. (`none`, `error`, `warn`, `info`, `verbose`)

{% migration %}
{% samplefile %}

```bash
parcel build index.js --log-level 1
```

{% endsamplefile %}
{% samplefile %}

```bash
parcel build index.js --log-level error
```

{% endsamplefile %}
{% endmigration %}

### `--global`

Эта опция удалена без замены (пока).

{% migration %}
{% samplefile %}

```bash
parcel build index.js --global mylib
```

{% endsamplefile %}
{% samplefile %}
{% endsamplefile %}
{% endmigration %}
