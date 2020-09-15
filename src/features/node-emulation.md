---
layout: layout.njk
eleventyNavigation:
  key: features-node-emulation
  title: 🐢 Node Emulation
  order: 6
summary: Некоторые функции, которые в конечном итоге имитируют Node.js's API
---

## 🌳 Переменные среды

Parcel использует [dotenv](https://github.com/motdotla/dotenv) для поддержки загрузки переменных среды из файлов `.env`.

`.env` файлы должны храниться вместе с файлом `package.json`, который содержит вашу зависимость `parcel-bundler`.

Parcel загружает файлы `.env` с этими именами для следующих значений `NODE_ENV`:

| valid `.env` filenames   | `NODE_ENV=*` | `NODE_ENV=test` |
| ------------------------ | ------------ | --------------- |
| `.env`                   | ✔️           | ✔️              |
| `.env.local`             | ✔️           | ✖️              |
| `.env.${NODE_ENV}`       | ✔️           | ✔️              |
| `.env.${NODE_ENV}.local` | ✔️           | ✔️              |

В частности:

- `NODE_ENV` по умолчанию `development`.
- `.env.local` не загружается, когда `NODE_ENV=test` поскольку [тесты должны давать одинаковые результаты для всех](https://github.com/parcel-bundler/parcel/blob/28df546a2249b6aac1e529dd629f506ba6b0a4bb/src/utils/env.js#L9)
- Иногда введение нового файла .env не срабатывает сразу. В этом случае попробуйте удалить каталог .cache/.
- Доступ напрямую к объекту `process.env` [не поддерживается] (https://github.com/parcel-bundler/parcel/issues/2299#issuecomment-439768971), но доступ к конкретным переменным на нем, например, `process.env.API_KEY` предоставит ожидаемое значение.
- Используйте встроенный в Node.js глобальный `process`, т.е. не `import process from "process"`. Если вы используете TypeScript, вы, вероятно, захотите установить `@types/node` для его компиляции.

## 🕳️ Полифиллинг и исключение встроенного Node Modules

Когда (или, что более вероятно, зависимость) импортируют пакеты, такие как `crypto`,` fs` или `process`, Parcel либо автоматически использует один из перечисленных полифиллов, либо исключает этот модуль. Ты можешь использовать [поле `aliases`  в своём `package.json`](/features/module-resolution/#aliases)

| native module | npm replacement            | native module  | npm replacement      |
| ------------- | -------------------------- | -------------- | -------------------- |
| assert        | `assert`                   | process        | `process/browser.js` |
| buffer        | `buffer`                   | punycode       | `punycode`           |
| console       | `console-browserify`       | querystring    | `querystring-es3`    |
| constants     | `constants-browserify`     | stream         | `readable-stream`    |
| crypto        | `crypto-browserify`        | string_decoder | `string_decoder`     |
| domain        | `domain-browser`           | sys            | `util/util.js`       |
| events        | `events`                   | timers         | `timers-browserify`  |
| http          | `stream-http`              | tty            | `tty-browserify`     |
| https         | `https-browserify`         | url            | `url`                |
| os            | `os-browserify/browser.js` | util           | `util/util.js`       |
| path          | `path-browserify`          | vm             | `vm-browserify`      |
| zlib          | `browserify-zlib`          |

## 📄 Inlining fs.readFileSync

Calls to `fs.readFileSync` are replaced with the file's contents if the filepath is statically determinable and inside the [project root](/features/module-resolution/)

- `fs.readFileSync(..., "utf8")`: with the contants as string with (or any other valid encoding)
- `fs.readFileSync(...)`: a Buffer object (e.g. `Buffer.from(....)` together with the an potentionally necessary polyfill)

{% sample "build index.js" %}
{% samplefile "index.js" %}

```js/3
import fs from "fs";
import path from "path";

const data = fs.readFileSync(path.join(__dirname, "data.json"), "utf8");
console.log("data");
```

{% endsamplefile %}
{% samplefile "data.json" %}

```json
{
  "foo": "bar"
}
```

{% endsamplefile %}
{% endsample %}

## 🔧 Disabling These Features

Inlining of [environment variables](#🌳-environment-variables) and [`readFileSync` calls](#%F0%9F%93%84-inlining-fs.readfilesync) can be disabled via a `@parcel/transformer-js` key in `package.json`:

{% sample %}
{% samplefile "package.json" %}

```json5
{
  "name": "my-project",
  "dependencies": {
    ...
  },
  "@parcel/transformer-js": {
    inlineFS: false,
    inlineEnvironment: false
  }
}
```

{% endsamplefile %}
{% endsample %}

`inlineEnvironment` can also be an array of glob strings:

```json5
{
  "name": "my-project",
  "dependencies": {
    ...
  },
  "@parcel/transformer-js": {
    inlineEnvironment: ["SENTRY_*"]
  }
}
```

`inlineFS` applies to `readFileSync` calls and `inlineEnvironment` to `process.env.SOMETHING`:

```ts
{
  inlineFS: boolean,
  inlineEnvironment: boolean | Array<Glob>
}
```

(This functionality is provided by `@parcel/transformer-js` and `@parcel/resolver-default`.)
