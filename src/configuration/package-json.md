---
layout: layout.njk
eleventyNavigation:
  key: configuration-package-json
  title: ❓ package.json
  order: 1
summary: Целевой объект в package.json
---

### Поля

Это поля, которые Parcel использует для своей конфигурации:

#### `main` / `module` / `browser`

Это общие поля, которые также используются другими инструментами,

{% sample %}
{% samplefile "package.json" %}

```json
{
  "main": "dist/main/index.js",
  "module": "dist/module/index.js",
  "browser": "dist/browser/index.js"
}
```

{% endsamplefile %}
{% endsample %}

По умолчанию они работают в режиме библиотеки (что означает, что они не связывают зависимости): (см. Также [`targets`](#targets))

- `main` (and `module`) являются стандартными точками входа в вашу библиотеку, по умолчанию для `module` используется вывод модуля ESM.
- `browser` предназначен для сборки для конкретного браузера (например, без некоторых встроенных функций).

Если указано одно из этих полей, Parcel создаст цель для этого поля (свойство в [`targets`](#targets) не требуется)

Чтобы Parcel игнорировал одно из этих полей, укажите `false` в `target.(main|browser|module)`:

{% sample %}
{% samplefile "package.json" %}

```json
{
  "main": "unrelated.js",
  "targets": {
    "main": false
  }
}
```

{% endsamplefile %}
{% endsample %}

Если поле `browser` является [объектом](/features/module-resolution/#package.json-browser-field),`package.json#browser[pkgName]` может использоваться вместо `package.json#browser`.

#### Специальные цели

Чтобы создать свою собственную цель (без какой-либо семантики [common target](#main-%2F-module-%2F-browser), описанной ранее), добавьте поле верхнего уровня с именем вашей цели и путем вывода. Вам также необходимо добавить его в [`targets`](#targets), чтобы Parcel распознал это поле.

{% sample %}
{% samplefile "package.json" %}

```json
{
  "app": "www/index.js",
  "targets": {
    "app": {}
  }
}
```

{% endsamplefile %}
{% endsample %}

#### `source`

Укажите точки входа для исходного кода, который сопоставляется с вашими целями и может быть строкой или массивом.

{% sample %}
{% samplefile "package.json" %}

```json
{
  "source": "src/index.js"
}
```

{% endsamplefile %}

{% samplefile "package.json" %}

```json
{
  "source": ["src/index.js", "src/index.html"]
}
```

{% endsamplefile %}
{% endsample %}

См. [Specifying Entrypoints](/getting-started/configuration/#specifying-entrypoints).

#### `targets`

Цели настраиваются через поле `package.json#targets`.

```json
{
  "app": "dist/browser/index.js",
  "appModern": "dist/browserModern/index.js",
  "targets": {
    "app": {
      "engines": {
        "browsers": "> 0.25%"
      }
    },
    "appModern": {
      "engines": {
        "browsers": "Chrome 70"
      }
    }
  }
}
```

Each target has a name which corresponds to a top-level `package.json` field
such as `package.json#main` or `package.json#app` which specify the primary
entry point for that target.

У каждой цели есть имя, соответствующее полю верхнего уровня `package.json`,
например `package.json#main` или `package.json#app`, которые определяют основную
точку входа для этой цели.

Каждая из этих целей содержит конфигурацию среды цели (все эти свойства не являются обязательными):

<div style="font-size: 0.9em">

| Option               | Possible values                                     | Description                                                                                                                                                                 |
| -------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `context`            | [see below](#context)                               | В какой среде выполнения пакеты должны работать.                                                                                                                                    |
| `distDir`            | `string`                                            | Укажите папку вывода (в отличие от файла вывода)                                                                                                                           |
| `engines`            | [`package.json#engines`](#engines-%2F-browserslist) | Более высокий приоритет, чем `package.json#engines`                                                                                                                                 |
| `includeNodeModules` | [see below](#includenodemodules)                    | Объединять ли все/нет/некоторые зависимости `node_module`                                                                                                                  |
| `isLibrary`          | `boolean`                                           | Библиотека как в "npm library"                                                                                                                                                 |
| `minify`             | `boolean`                                           | Включить ли минификацию (точное поведение определяется плагинами). <br> Установлено [`--no-minify`](/features/cli/#parameters-specific-to-build)                         |
| `outputFormat`       | `'global' | 'esmodule' | 'commonjs'`                | Какой тип импорта/экспорта должен быть выпущен                                                                                                                             |
| `publicUrl`          | `string`                                            | Общедоступный URL-адрес пакета во время выполнения                                                                                                                                     |
| `scopeHoist`         | `boolean`                                           | Разрешить ли подъем области видимости <br> Должен быть `true` для ESM и CommonJS `outputFormat`. <br> Установлено [`--no-scope-hoist`](/features/cli/#parameters-specific-to-build) |
| `sourceMap`          | [see below](#sourcemap)                             | Включить/выключить карту источников и установить параметры. <br> Заменено на [`--no-source-maps`](/features/cli/#general-parameters)                                                       |

</div>

Однако большая часть стандартной конфигурации для создания библиотеки уже предоставлена ​​вам по умолчанию:

```cs
targets = {
  main: {
    engines: {
      node: value("package.json#engines.node"),
      browsers: unless exists("package.json#browser") then value("package.json#browserlist")
    },
    isLibrary: true
  },
  module: {
    engines: {
      node: value("package.json#engines.node"),
      browsers: unless exists("package.json#browser") then value("package.json#browserlist")
    },
    isLibrary: true
  },
  browser: {
      engines: {
      browsers: value("package.json#browserslist")
    },
    isLibrary: true
  },
  ...value("package.json#targets"),
}
```

##### `context`

Возможные значения: `'node' | 'browser' | 'web-worker' | 'service-worker' | 'electron-main' | 'electron-renderer'`.

Эти значения могут использоваться подключаемыми модулями (например, URL-адрес сервис-воркера не должен содержать хеш-код, веб-работник может использовать `importScripts`).

Для [common targets](#main-%2F-module-%2F-browser) они выводятся из цели:

- У цели `main` есть контекст `node`, если есть цель `browser` или `engines.node != null && engines.browsers == null`, иначе `browser`.
- У цели `module` есть контекст `node`, если есть цель `browser`, иначе `browser`.
- У цели `browser` есть контекст` browser`.

##### `includeNodeModules`

По умолчанию это поле имеет значение false, если isLibrary имеет значение true. Возможные значения:

- `false`: не включать пакет `node_modules`
- an `array`: список имен пакетов или подстановочных знаков для _include_
- an `object`: `includeNodeModules[pkgName] ?? true` определяет, включен ли он. (например, `{ "lodash": false }`)

См. [Module Resolution](/features/module-resolution#externals) для получения дополнительной информации.

##### `sourceMap`

Может быть логическим (чтобы просто включить / отключить исходные карты) или параметром (который несколько эквивалентен " истине`):

| Option        | Default value                       | Description                                                                                                     |
| ------------- | ----------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| inline        | `false`                             | Включите исходную карту в качестве url-адреса данных в пакет (в `sourceMappingURL`)                                   |
| inlineSources | `false`                             | Если исходная карта содержит содержимое источников (в противном случае они будут загружены из `${sourceRoot}/$(name)`) |
| sourceRoot    | `path.relative(bundle, pojectRoot)` | По сути это публичный url для источников                                                                      |

Параметр CLI [--no-source-maps](/features/cli/#general-parameters) устанавливает значение по умолчанию на `false` (в отличие от` true`).

#### `engines` / `browserslist`

These top-level fields set the default value for `target.*.engines.browsers` and `target.*.engines`, respectively.

Эти поля верхнего уровня устанавливают значение по умолчанию для `target.*.engines.browsers` и `target.*.engines` соответственно.

Определяет [environment](/getting-started/configuration/#environments).
{% sample %}
{% samplefile "package.json" %}

```json
{
  "browserslist": ["> 0.2%", "not dead"]
}
```

{% endsamplefile %}
{% samplefile "package.json" %}

```json
{
  "engines": {
    "node": ">=4.x",
    "electron": ">=2.x",
    "browsers": "> 0.25%"
  }
}
```

{% endsamplefile %}
{% endsample %}

#### `alias`

См. [Module Resolution](/features/module-resolution/#aliases)

### Какой `package.json` используется при указании нескольких записей (пакетов)

Все пути относительно `/some/dir/my-monorepo`.

| cwd                  |  entries                      |  used `pkg.json#*` (fields described below)  |
| -------------------- | ----------------------------- | -------------------------------------------- |
| `..`                 | `packages/*/src/**/*.js`      | `package.json`                               |
| `.`                  | `packages/*/src/**/*.js`      | `package.json`                               |
| `packages/`          | `packages/*/src/**/*.js`      | `package.json`                               |
| `packages/pkg-a`     | `packages/pkg-a/src/index.js` | `packages/pkg-a/package.json`                |
| `packages/pkg-a/src` | `packages/pkg-a/src/index.js` | `packages/pkg-a/package.json`                |
