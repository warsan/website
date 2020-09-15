---
layout: layout.njk
eleventyNavigation:
  key: configuration-plugin-configuration
  title: 🔌 Plugin Configuration
  order: 2
summary: Как использовать плагины и создавать именованные конвейеры
---

{% note %}
Вопреки тому, что может предполагать название этой страницы, речь идет не о настройке отдельных плагинов, а о том, как сообщить Parcel, какой плагин отвечает (среди прочего) за какой тип файла(ов).
{% endnote %}

Parcel спроектирован так, чтобы быть очень модульным, сам `@parcel/core` (почти) не специфичен для объединения Javascript или веб-страниц. Чтобы фактически указать поведение, существуют различные плагины (см. [Plugin System](/plugin-system/)).

Вот отрывок из конфигурации по умолчанию, которую использует CLI `parcel`. Как правило, существуют три категории типов плагинов (в зависимости от конфигурации):

- только один плагин на всю сборку (бандлер)
- список плагинов, которые запускаются последовательно (namers/resolvers/reporters)
- подключаемые модули указываются для каждого типа актива/пакета (transformers/packagers/optimizers)
- время выполнения здесь является исключением, потому что они указаны для [context](/getting-started/configuration/#targets-2).

{% sample %}
{% samplefile ".parcelrc" %}

```json/3,14,17
{
  "bundler": "@parcel/bundler-default",
  "transformers": {
    "*.{js,jsx,ts,tsx}": [
      "@parcel/transformer-babel",
      "@parcel/transformer-js"
    ],
    "url:*": ["@parcel/transformer-raw"]
  },
  "namers": ["@parcel/namer-default"],
  "runtimes": {
    "browser": ["@parcel/runtime-js", "@parcel/runtime-browser-hmr"],
    "service-worker": ["@parcel/runtime-js"],
    "web-worker": ["@parcel/runtime-js"],
    "node": ["@parcel/runtime-js"]
  },
  "optimizers": {
    "*.js": ["@parcel/optimizer-terser"]
  },
  "packagers": {
    "*.html": "@parcel/packager-html",
    "*": "@parcel/packager-raw"
  },
  "resolvers": ["@parcel/resolver-default"],
  "reporters": ["@parcel/reporter-cli"]
}
```

{% endsamplefile %}
{% endsample %}

Тип файла указывается глобусом, который сопоставляется с _whole filepath_ (_pipelines_ сопоставляются в порядке объявления), поэтому вы можете использовать разные плагины в зависимости от пути к файлу ввода/вывода:

- Глобусы для трансформаторов сопоставляются с активом (входным) путём.
- Глобусы для упаковщиков и оптимизаторов сопоставляются с путями расслоения (вывода).

### Расширение конфигов

Распространенным вариантом использования является расширение конфигурации по умолчанию, по этой причине поле `extends` может быть пакетом конфигурации или массивом пакетов конфигурации для расширения.

{% sample %}
{% samplefile ".parcelrc" %}

```json/1
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"]
  }
}
```

{% endsamplefile %}
{% endsample %}

### Конвейеры

Внимательный читатель мог заметить, что последний пример конфигурации не включал `@parcel/transformer-js`, который требуется для `@parcel/runtime-js` и `@parcel/runtime-packager`.

Это решается с помощью _pipelines_. Ресурс Typescript сначала обрабатывается конвейером `ts`, и как только плагин `@parcel/transformer-typescript-ts` устанавливает тип актива (который по существу эквивалентен расширению файла) на `js`, Parcel повторно оценивает, как ресурс подлежат дальнейшей обработке. В этом случае он будет помещен в конвейер `js`, указанный в`@parcel/config-default`. Таким образом, `@parcel/transformer-js` по-прежнему будет выполняться.

{% warning %}

Как только преобразователь устанавливает тип актива в тип, который не охвачен текущим конвейером, актив будет либо помещен в другой конвейер, либо преобразование будет завершено.  
_Трансформеры, которые после этого должны запускаться в соответствии с текущим конвейером, запускаться не будут._

{% endwarning %}

Если преобразователь не меняет тип актива, а вы все еще хотите продолжить обработку этого актива, добавьте `"..."`, чтобы продолжить преобразование (в расширенной конфигурации). Это может быть полезно, если вы хотите изменить актив без изменения его типа и позволить уже определенному конвейеру обрабатывать перевод/регистрацию зависимости.

{% sample  %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.js": ["parcel-transformer-add-comment", "..."]
  }
}
```

{% endsamplefile %}
{% endsample %}

### Именованные конвейеры

В дополнение к конвейерам на основе типов активов существуют _ named pipelines_, которые позволяют вам импортировать один тип актива различными способами (например, форматами).

Именованные конвейеры указываются с использованием синтаксиса, подобного прокотолу, например `import myLogo from "url:./logo.png";`

Вот пример того, как вы достигли зависимости URL-адреса, которая не создает новый пакет, а скорее встроена в URL-адрес данных.

{% sample undefined, "column" %}
{% samplefile ".parcelrc" %}

(_Примечание: эта конфигурация уже содержится в `@parcel/config-default`, поэтому здесь приведена  только для иллюстрации._)

```json/3,6
{
  "extends": "@parcel/config-default",
  "transformers": {
    "data-url:*": ["@parcel/transformer-inline-string", "..."]
  },
  "optimizers": {
    "data-url:*": ["...", "@parcel/optimizer-data-url"]
  }
}
```

{% endsamplefile %}
{% samplefile "index.js" %}

```js/2
import x from "./other.js";

new Worker("data-url:./worker.js");

// ...
```

{% endsamplefile %}
{% endsample %}

Как видите, `...` теперь используется, чтобы убедиться, что `data-url: ./worker.js` по-прежнему будет обрабатываться конвейером `js` (именованный описатель конвейера применяется только для первого совпадения конвейера).

{% note %}

Если вам интересно, как этого можно достичь без более глубокой интеграции с ядром Parcel:

`@parcel/transformer-inline-string` отмечает, что актив является встроенным. `@parcel/packager-js` затем встраивает этот встроенный пакет (как строку `"${contents}"`). Этот встроенный пакет ранее обрабатывался `@parcel/optimizer-data-url`, кодирующий код JS в URL-адрес данных.

{% endnote %}

Именованные конвейеры в настоящее время реализованы для преобразователей и оптимизаторов (эти конвейеры унаследованы от начального актива).

#### Предопределенные (официальные) именованные конвейеры

- `data-url:` См. Пример выше. Он заменяется не URL-адресом нового пакета, а изолированным URL-адресом данных.
- `url:` Требуется, например, когда импорт «обычных» ресурсов, таких как медиафайлы, в виде URL-адреса

{% sample %}
{% samplefile "index.js" %}

```js/0
import logo from "url:./logo.svg";

document.body.innerHTML = `<img src="${logo}">`;
```

{% endsamplefile %}
{% endsample %}

{% note %}

Вы можете спросить, почему мы решили использовать этот явный синтаксис. Причина в том, что таким образом добавление нового типа ресурса в Parcel больше не является критическим изменением (это происходило в прошлом, когда `import foo from "./other.html"` чаще возвращал не URL-адрес, а содержимое HTML).

Также можно изменить конфигурацию ссылки, чтобы выбрать старое поведение: см. [Migration](/getting-started/migration/#importing-non-code-assets-from-javascript).

{% endnote %}

- `bundle-text:` Может использоваться, например, для импорта содержимого файла CSS (или МЕНЬШЕ!) в Javascript (необходимо для некоторых "фреймворков")

{% sample %}
{% samplefile "style.less" %}

```less
@myColor: #143352;

span {
  color: @myColor;
}
```

{% endsamplefile %}
{% samplefile "index.js" %}

```js/0,9
import style from "bundle-text:./logo.less";

class MyTest extends HTMLElement {
  constructor() {
    super();

    let shadow = this.attachShadow({ mode: "open" });

    let style = document.createElement("style");
    style.textContent = style;
    shadow.appendChild(style);

    let info = document.createElement("span");
    info.textContent = this.getAttribute("label");
    shadow.appendChild(info);
  }
}

customElements.define("my-test", MyTest);
```

{% endsamplefile %}
{% endsample %}
