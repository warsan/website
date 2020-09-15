---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-configuration
  title: Configuration
  order: 12
summary: "Не совсем тип плагина: многоразовый пакет '.parcelrc'"
---

Это просто файл [`.parcelrc`](/configuration/plugin-configuration/), завернутый в опубликованный пакет, `main` указывающий на файл конфигурации.

```json
{
  "name": "parcel-config-foo",
  "main": "index.json",
  "version": "1.0.0",
  "engines": {
    "parcel": "2.x"
  }
}
```

Один из вариантов использования для этого - система с несколькими плагинами.
