---
layout: layout.njk
eleventyNavigation:
  key: features-cli
  title: ⌨️ CLI
  order: 2
summary: Команда "parcel" 
---

## Команды

«Записи» во всех командах могут быть:

- один или несколько файлов
- один или несколько шариков
- один или несколько директорий (см. [Specifying Entrypoints](/getting-started/configuration/#specifying-entrypoints))

{% warning %}

Если шаблон глобуса содержит подстановочный знак `*`, убедитесь, что шаблон заключен в одинарные кавычки. Это гарантирует, что Parcel правильно прочитает шаблон, не подвергаясь манипуляциям со стороны вашей оболочки.

```bash
# OK
parcel './**/*.html'
parcel './img/**/*'

# Not OK
parcel ./**/*.html
parcel ./img/**/*
```

{% endwarning %}

### `parcel [serve] <entries>`

Запускает сервер разработки, который автоматически перестраивает ваше приложение по мере изменения файлов и поддерживает [hot module replacement](/features/hmr/).
Вы также можете передать глобус или список глобусов для нескольких точек входа:

```bash
parcel index.html
parcel a.html b.html
parcel './**/*.html'
```

{% warning %}

Если вы указали несколько точек входа HTML и ни одна из них не имеет пути вывода `/index.html`, сервер разработки ответит на `localhost:1234/` кодом 404, поскольку Parcel не знает, какой HTML-пакет является индекс.

В этом случае загрузите файл напрямую, например `http://localhost:1234/a.html` и `http://localhost:1234/b.html`

{% endwarning %}

### `parcel watch <entries>`

Команда watch похожа на `serve`, но только с HMR-сервером и без HTTP (dev) сервера.

```bash
parcel index.html
```

### `parcel build <entries>`

Builds the assets once, it also enabled minification and sets the NODE_ENV=production environment variable. See Production for more details.

```bash
parcel build index.html
```

As opposed to `serve` and `watch`, `build` has [scope hoisting](/features/scope-hoisting) enabled by default (so the other commmands implicity specify `--no-scope-hoist`).

## Parameters

### General parameters

| Format                                       | Description                                                                                                             |
| -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| `--cache-dir <path>`                         | Sets the cache directory. defaults to `.parcel-cache`                                                                   |
| `--log-level (none|error|warn|info|verbose)` | Sets the log level                                                                                                      |
| `--no-autoinstall`                           | Disables autoinstall                                                                                                    |
| `--no-cache`                                 | Disables reading from the filesystem cache                                                                              |
| `--no-source-maps`                           | Disables sourcemaps, <br> Overrides [`targets.*.sourceMap`](/configuration/package-json/#sourcemap)                     |
| `--profile`                                  | Profiles the build (a flamechart can be generated)                                                                      |
| `--public-url <url>`                         | The path prefix for absolute urls. <br> Default value for [`targets.*.publicUrl`](/configuration/package-json/#targets) |
| `--target [name]`                            | Only build the specified target(s)                                                                                      |
| `-V, --version`                              | Outputs the version number                                                                                              |

### Parameters related to the dev server/watch mode (`serve` and `watch`)

| Format              | Description                                                                           |
| ------------------- | ------------------------------------------------------------------------------------- |
| `--no-hmr`          | Disables [hot module replacement](/features/hmr)                                      |
| `-p, --port <port>` | The port for the HMR and HTTP server (the default port is `process.env.PORT` or 1234) |
| `--host <host>`     | Sets the host to listen on, defaults to listening on all interfaces                   |
| `--https`           | Serves files over HTTPS                                                               |
| `--cert <path>`     | Path to a certificate to use with HTTPS                                               |
| `--key <path>`      | Path to a private key to use with HTTPS                                               |
| `--watch-for-stdin` | Stop Parcel once stdin is closed                                                      |

### Parameters specific to `serve`

| Format             | Description                                                                    |
| ------------------ | ------------------------------------------------------------------------------ |
| `--open [browser]` | Automatically opens the entry in your browser, defaults to the default browser |

### Parameters specific to the non-server commands (`watch` and `build`)

| Format             | Description                                                                                                                                  |
| ------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `--dist-dir <dir>` | Output directory to write to when unspecified by targets. <br> Default value for [`targets.*.distDir`](/configuration/package-json/#targets) |

### Parameters specific to `build`

| Format                      | Description                                                                                                                               |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `--no-minify`               | Disables minification (exact behaviour is determined by plugins). <br> Related [`targets.*.minify`](/configuration/package-json/#targets) |
| `--no-scope-hoist`          | Disables scope hoisting. <br> Related: [`targets.*.scopeHoist`](/configuration/package-json/#targets)                                     |
| `--detailed-report [depth]` | Displays the largest 10 (number configurable with `depth`) assets per bundle in the CLI report                                            |
