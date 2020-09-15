---
layout: layout.njk
eleventyNavigation:
  key: recipes-debugging
  title: 🛠️ Debugging
  order: 2
---

Поскольку Parcel автоматически генерирует исходные карты по умолчанию, настройка отладки с помощью Parcel по большей части требует минимальных усилий.

## Инструменты разработчика Chrome

Предполагая, что исходные карты включены, дополнительная настройка здесь не требуется.

Например, предположим, что у вас есть структура папок, подобная следующей:

{% sample "src/index.html" %}
{% samplefile %}

```
├── package.json
├── src
│   ├── index.html
│   └── index.ts
└── yarn.lock
```

{% endsamplefile %}
{% samplefile "src/index.html" %}

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Chrome Debugging Example</title>
  </head>
  <body>
    <h1 id="greeting"></h1>
    <script src="./index.ts"></script>
  </body>
</html>
```

{% endsamplefile %}
{% samplefile "src/index.ts" %}

```ts
const variable: string = "Hello, World!";

document.getElementById("greeting").innerHTML = variable;
```

{% endsamplefile %}
{% endsample %}

(`package.json` only has `parcel-bundler` installed)

При такой настройке вы можете запустить `parcel src/index.html` и установить точки останова в исходном коде, как показано ниже:

![Example Chrome Breakpoints](../debugging1.png)

## Visual Studio Code

Предполагая, что структура папок/файлов аналогична показанной выше для инструментов разработчика Chrome, следующий `launch.json` можно использовать с [Debugger for Chrome](https://marketplace.visualstudio.com/items?itemName=msjsdiag.debugger-for-chrome) расширением:

{% sample %}
{% samplefile "launch.json" %}

```json
{
  // Используйте IntelliSense, чтобы узнать о возможных атрибутах.
  // Наведите указатель мыши на описание существующих атрибутов.
  // Для получения дополнительной информации посетите: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome against localhost",
      "url": "http://localhost:1234",
      "webRoot": "${workspaceFolder}",
      "breakOnLoad": true,
      "sourceMapPathOverrides": {
        "../*": "${webRoot}/*"
      }
    }
  ]
}
```

{% endsamplefile %}
{% endsample %}

Затем вам нужно будет запустить сервер parcel dev с вашей точкой входа, которая здесь `index.html`:

```
$ parcel src/index.html
```

Последний шаг здесь - фактически запустить процесс отладки, щелкнув зеленую стрелку на панели отладки. Теперь вы можете устанавливать точки останова в своем коде. Окончательный результат будет выглядеть примерно так:

![Example Chrome Breakpoints](../debugging2.png)
