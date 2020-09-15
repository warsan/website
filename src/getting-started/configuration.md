---
layout: layout.njk
eleventyNavigation:
  key: getting-started-no-configuration
  title: 🗒️ (No) configuration
  order: 3
summary: Как далеко вы можете пройти без какой-либо конфигурации и как настроить Parcel
---

(Значение символа `~` в этом разделе описано в [Module Resolution](/features/module-resolution/#tilde-paths).)

### Цели

Когда Parcel запускается, он может одновременно создавать файлы несколькими способами. Их называют «мишенями» (targets).

Например, у вас может быть «современная» цель, нацеленная на новые браузеры, и «устаревшая» цель для старых браузеров.

Каждая точка входа будет обработана (и выведена) один раз для каждой цели.

### Указание точек входа

Это файлы, которые содержат исходный код вашего приложения до того, как
составляется Parcel и забирает их:

1. [`$ parcel <entries>`](/features/cli/)
2. `$ parcel <folder(s)>` uses [`<folder>/package.json#source`](/configuration/package-json/#source) (respectively)
3. `./src/index.*`
4. `./index.*`

Отсюда все, от чего зависят эти активы, будет считаться "источником" для
Пакета.

### Установка пути вывода

Путь, по которому должны быть размещены выходные пакеты, можно указать (в этом порядке), используя поле верхнего уровня в `package.json` (см. [common targets](/configuration/package-json/#main-%2F-module-%2F-browser) и [custom targets](/configuration/package-json/#custom-targets)), с помощью [`targets.*.distDir`](/configuration/package-json/#targets) или [`--dist-dir`](</features/cli/#parameters-specific-to-the-non-server-commands-(watch-and-build)>) CLI параметр.

Значения по умолчанию для выходной папки

- для общих целей `path.dirname(package.json#${targetName})`
- для настраиваемых целей `path.dirname(package.json#${targetName})` или `~/dist/${targetName}/`.

Неявная цель по умолчанию имеет выходную папку `~/dist/`.

С несколькими точками входа вы должны использовать явный `distDir` вместо целевых полей верхнего уровня, потому что Parcel не будет знать, какой пакет должен иметь указанное имя:

{% sample "a.html b.html" %}
{% samplefile "package.json" %}

```json
{
  "targets": {
    "app": {
      "distDir": "./www"
    }
  }
}
```

{% endsamplefile %}
{% endsample %}

### Среды

Среды сообщают Parcel, как преобразовать и связать каждый актив. Они говорят
Parcel, где актив будет запускаться, в браузере или в Node/Electron.

Они также сообщают плагинам Parcel, каким должен быть их вывод, указывая, на какие
браузеры (версии), ориентирована ваша сборка
(например [Babel](http://babeljs.io/docs/en/babel-preset-env#targetsbrowsers) или
[Autoprefixer](https://github.com/postcss/autoprefixer#browsers)).

Вы можете настроить среды через [`targets#context` and `targets#engines`](/configuration/package-json/#targets) и [`engines / browserslist`](/configuration/package-json/#engines-%2F-browserslist).

### Настройка Parcel

Когда вам действительно нужно настроить Parcel, это будет в одном из трех мест.

- Если вам нужно настроить CLI, это будет [CLI flag](/features/cli/)
- Если вам нужно настроить свой пакет, он будет в [`package.json`](/configuration/package-json/)
- Если вам нужно что-то настроить с вашими файлами или активом Parcel
  трубопровод, он будет в [`.parcelrc`](/configuration/plugin-configuration/)
