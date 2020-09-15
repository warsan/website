---
layout: layout.njk
eleventyNavigation:
  key: features-module-resolution
  title: 📔 Module Resolution
  order: 5
summary: Как разрешаются зависимости
---

Решатель Parcel реализует модифицированную версию алгоритма [the node_modules resolution](https://nodejs.org/api/modules.html#modules_all_together).

- **root directory**: каталог точки входа, указанной для Parcel, или общий корень (ближайший общий родительский каталог), если указано несколько точек входа.
- **project root**: ближайший к корневому каталогу каталог, содержащий файл блокировки(`yarn.lock`, `package-lock.json`, `pnpm-lock.yaml`) или папка VCS (`.git`, `.hg`)

### Абсолютные пути

`/foo` разрешает `foo` относительно **project root**.

### Тильды Пути

`~/foo` разрешает `foo` относительно ближайшего **`node_modules`** каталога, ближайшего каталога с **`package.json`** или корня проекта - в зависимости от того, что наступит раньше.

### package.json `browser` поле

Если в комплект входит [package.browser field](https://docs.npmjs.com/files/package.json#browser) (и это строка), Parcel будет использовать это вместо записи `package.main`.

Если это объект, он ведет себя так же, как [`aliases`](#aliases), но имеет более высокий приоритет, когда [`target.context === "browser"`](/configuration/package-json/#context).

### Псевдонимы

Псевдонимы поддерживаются через поле `alias` в` package.json`.

В этом примере псевдонимы реагируют (`react`) на `preact` и некоторые локальные настраиваемые модули, которых нет в `node_modules`.

Они также могут отображаться в глобальные переменные, которые, как ожидается, будут существовать во время выполнения. Это может быть полезно для замены зависимости, например, версией, загруженной из CDN.

{% sample %}
{% samplefile "package.json" %}

```json/5-11
{
  "name": "some-package",
  "devDependencies": {
    "parcel-bundler": "^2.0.0-beta.1"
  },
  "alias": {
    "react": "preact/compat",
    "react-dom": "preact/compat",
    "local-module": "./custom/modules",
    "other-local-module": { "fileName": "./custom/other-module" },
    "lodash": { "global": "_" }
  }
}
```

{% endsamplefile %}
{% endsample %}

Избегайте использования каких-либо специальных символов в своих псевдонимах, поскольку некоторые из них могут использоваться Parcel, а другие - сторонними инструментами или расширениями. Например:

- `~` используется Parcel для разрешения [tilde paths](#tilde-paths).
- `@` используется npm для пакетов от npm-организаций.

Мы советуем быть явным при определении псевдонимов, поэтому, пожалуйста, **указывайте расширения файлов**, в противном случае посылку нужно будет угадывать. См. [JavaScript Named Exports](#javascript-named-exports) для примера этого.

Добавление к глобальным псевдонимам префикса `window` или `globalThis` (например `window.jQuery`) может дать сбой при многоплатформенных сборках и не рекомендуется.  
Хотя псевдонимами файлов могут быть `"path/to/file"` вместо `{ "fileName": "path/to/file" }`, глобальные переменные должны использовать формат `{ "global": "name" }`.

### Внешние

Внешние элементы должны быть настроены для каждой цели с помощью [`includeNodeModules`](/configuration/package-json#includenodemodules). Как и глобальные переменные, внешние элементы не будут объединяться, но вместо этого они будут импортированы во время выполнения.

{% sample %}
{% samplefile "package.json" %}

```json/3-5
{
  "targets": {
    "app": {
      "includeNodeModules": {
        "react": false
      }
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

### Поля ввода пакета

Когда подъем области видимости включен, только указанное значение (например, `lodash`) разрешается в следующем порядке (первое указанное поле и указывает на существующий файл):

- `package.json#source`
- `package.json#browser`
- `package.json#module`
- `package.json#main`
- `index.{js, json}`

Однако без поднятия области видимости `main` предпочтительнее `module` для лучшей производительности:

- `package.json#source`
- `package.json#browser`
- `package.json#main`
- `package.json#module`
- `index.{js, json}`

## Общие проблемы

### Javascript Named Экспорт

Сопоставления псевдонимов применяются ко многим типам ресурсов и специально не поддерживают сопоставление именованных экспортов JavaScript. Если вы хотите сопоставить JS именованный экспорт, вы можете повторно экспортировать именованный экспорт в файл с псевдонимом:

{% sample %}
{% samplefile "package.json" %}

```json5/3
{
  name: "some-package",
  alias: {
    ipcRenderer: "./electron-ipc.js", // указать расширение файла
  },
}
```

{% endsamplefile %}
{% samplefile "electron-ipc.js" %}

```js
module.exports = require("electron").ipcRenderer;
```

{% endsamplefile %}
{% endsample %}

### Поток с абсолютным разрешением или разрешением тильды

При использовании абсолютного пути или разрешения модуля пути тильды вы должны настроить Flow с помощью [module.name_mapper](https://flow.org/en/docs/config/options/#toc-module-name-mapper-regex-string) характерная черта.

Учитывая проект с такой структурой и `src/index.html` в качестве точки входа, **корень записи** является папкой `src/`.

```
├── package.json
├── .flowconfig
└── src/
    ├── index.html
    ├── index.js
    └── components/
        ├── apple.js
        └── banana.js
```

Therefore, to map this import correctly, Flow should replace the leading `/` in `'/components/apple'` with `src/`, resulting in `'src/components/apple'`. That is achieved by the following setting in your `.flowconfig`.

Следовательно, чтобы правильно сопоставить этот импорт, Flow должен заменить начальный `/` в `'/components/apple'` на `src/`, в результате получится `'src/components/apple'`. Это достигается следующей настройкой в ​​вашем `.flowconfig`.

{% sample %}
{% samplefile "index.js" %}

```js
import Apple from "/components/apple";
```

{% endsamplefile %}
{% samplefile ".flowconfig" %}

```ini
[options]
module.name_mapper='^\/\(.*\)$' -> '<PROJECT_ROOT>/src/\1'
```

`<PROJECT_ROOT>` является идентификатором потока, указывающим местоположение вашего `.flowconfig`.
{% endsamplefile %}
{% endsample %}

Примечание: `module.name_mapper` может иметь несколько записей. Это позволило поддерживать [absolute](#absolute-paths) или [tilde](#~-tilde-paths) Разрешение пути в дополнение к поддержке [local module aliasing](#aliases).

### TypeScript ~ Разрешение

TypeScript должен будет знать о вашем использовании разрешения модуля `~` или сопоставлений псевдонимов. Пожалуйста, обратитесь к [TypeScript Module Resolution docs](https://www.typescriptlang.org/docs/handbook/module-resolution.html) для получения дополнительной информации.

{% sample %}
{% samplefile "tsconfig.json" %}

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "~*": ["./src/*"]
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

### Монорельсовое Разрешение

TODO?

Это рекомендуемые способы использования монорецепторов в настоящее время:

Рекомендуемое использование:

- используйте относительные пути.
- используйте `/` для корневого пути, если требуется корневой путь.
- **избегайте** `~` в пределах монорельса.

Если вы являетесь пользователем monorepo и хотите внести свой вклад в эти рекомендации, пожалуйста, предоставьте примеры repos при открытии вопросов для поддержки обсуждения.
