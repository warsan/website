---
layout: layout.njk
eleventyNavigation:
  key: languages-typescript
  title: <img src="/assets/lang-icons/typescript.svg" alt=""/> TypeScript
  order: 2
summary: Объясняет различные способы передачи TypeScript.
---

Машинопись сразу работает с Parcel, как есть...

## Babel

Babel может удалять аннотации типов TypeScript, используя `@babel/preset-typescript`, это способ по умолчанию, которым Parcel переносит TypeScript, потому что он обычно быстрее в нашем конвейере. Однако у этого способа есть несколько недостатков:

- Нет проверки типа
- `tsconfig.json` игнорируется, поэтому такие языковые функции, как свойства класса и декораторы, должны предоставляться плагином Babel.
- `const enum` не поддерживается

Чтобы увидеть полный список, взгляните на [Babel документацию](https://babeljs.io/docs/en/babel-plugin-transform-typescript#caveats).

## TypeScript's `tsc`

Если вы используете более продвинутые функции TypeScript, которые включают пользовательские настройки конфигурации в `tsconfig.js`, вы можете использовать преобразователь `@parcel/transformer-typescript-tsc`, который использует официальный компилятор TypeScript:

{% sample %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "transformers": {
    "*.{ts,tsx}": ["@parcel/transformer-typescript-tsc"]
  }
}
```

{% endsamplefile %}
{% endsample %}

Поскольку Parcel обрабатывает каждый файл индивидуально, он неявно устанавливает `isolatedModules: true` в параметрах tsc, это также имеет ограничения, а именно: константа `const enum` тоже не поддерживается.

- Нет проверки типа
- `const enum` не поддерживается
- Некоторые параметры `tsconfig.json` (такие как `paths`) в настоящее время не поддерживаются конфигурацией по умолчанию.

## Проверка типа

Ни преобразователь Babel, ни преобразователь tsc не выполняют проверку типов, они просто удаляют аннотации типов. Единственный встроенный способ проверить типы - использовать валидатор tsc, который запускается после создания пакетов. Вам необходимо добавить файл `tsconfig.json`, который включает ваши исходные файлы (хотя валидатор по-прежнему работает только с активами, обработанными Parcel). TODO

{% sample %}
{% samplefile ".parcelrc" %}

```json/3
{
  "extends": "@parcel/config-default",
  "validators": {
    "*.{ts,tsx}": ["@parcel/validator-typescript"]
  }
}
```

{% endsamplefile %}
{% samplefile "tsconfig.json" %}

```json
{
  "include": ["src/**/*"]
}
```

{% endsamplefile %}
{% endsample %}

## Генерация типизации

Parcel может извлечь типизацию вашей точки входа, указав package.json # types (автоматическая цель, например `main`)

{% sample %}
{% samplefile "package.json" %}

```json/3
{
  "source": "src/index.ts",
  "module": "dist/index.js",
  "types": "dist/index.d.ts"
}
```

{% endsamplefile %}
{% endsample %}

(Эта функциональность предоставляется `@parcel/transformer-babel` или `@parcel/transformer-typescript-tsc`.)
