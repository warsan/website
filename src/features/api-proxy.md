---
layout: layout.njk
eleventyNavigation:
  key: features-api-proxy
  title: 🚇 API Proxy
  order: 1
summary: Настройте встроенный сервер разработки для пересылки определенных путей на другой сервер
---

Чтобы лучше эмулировать реальную производственную среду при разработке веб-приложений, вы можете указать пути, которые должны быть проксированы на другой сервер (например, ваш реальный сервер API или локальный сервер тестирования) в файле `.proxyrc` или `.proxyrc.js`.

### `.proxyrc`

В этом файле JSON вы указываете объект, где каждый ключ является шаблоном, с которым сопоставляется URL-адрес, а значение - [`http-proxy-middleware` options](https://github.com/chimurai/http-proxy-middleware#options) объект:

{% sample %}
{% samplefile ".proxyrc.js" %}

```js
{
  "/api": {
    "target": "http://localhost:8000/",
    "pathRewrite": {
      "^/api": ""
    }
  }
}

```

{% endsamplefile %}
{% endsample %}

Это приведет к тому, что http://localhost:1234/api/endpoint будет проксироваться на `http://localhost:8000/endpoint`.

### `.proxyrc.js`

Для более сложных конфигураций файл `.proxyrc.js` позволяет вам подключать любое (совместимое с подключением) промежуточное ПО, этот пример имеет то же поведение, что и версия `.proxyrc` выше.

{% sample %}
{% samplefile ".proxyrc.js" %}

```js
const { createProxyMiddleware } = require("http-proxy-middleware");

module.exports = function (app) {
  app.use(
    createProxyMiddleware("/api", {
      target: "http://localhost:8000/",
      pathRewrite: {
        "^/api": "",
      },
    })
  );
};
```

{% endsamplefile %}
{% endsample %}

(Эта функциональность предоставляется `@parcel/reporter-dev-server`.)
