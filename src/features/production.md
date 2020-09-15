---
layout: layout.njk
eleventyNavigation:
  key: features-production
  title: 🏭 Production
  order: 8
summary: Как Parcel помогает вам оптимизировать ваш проект для производства
---

TODO

## Проверка размера пакета

Parcel имеет встроенные плагины для нескольких инструментов, помогающих анализировать размер пакета.

### Пакетный анализатор

Чтобы сгенерировать HTML-файл для каждого пакета, установите переменную среды `PARCEL_BUNDLE_ANALYZER`.

{% sample "PARCEL_BUNDLE_ANALYZER=1 parcel build src/index.html" %}
{% endsample %}

Это создает папку parcel-bundle-reports в корне вашего проекта с файлом HTML для каждой цели:

<div style="border: 1px solid black">

![Скриншот вывода анализатора пакетов](/assets/bundle-analyzer.png)

</div>

### Связка друзей

Установите переменную окружения `BUNDLE_BUDDY`

{% sample "BUNDLE_BUDDY=1 parcel build src/index.html" %}
{% endsample %}

и используйте файлы (в каталоге dist) на [веб-сайте Bundle Buddy](https://bundle-buddy.com/parcel).

<div style="border: 1px solid black">

![Скриншот сайта Bundle Buddy с загруженным проектом](/assets/bundle-buddy.png)

</div>
