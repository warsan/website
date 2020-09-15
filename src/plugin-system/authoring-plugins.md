---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-authoring-plugins
  title: Authoring Plugins
  order: 2
summary: "Что нужно учитывать при создании плагина"
---

## Плагин API

Есть несколько разных типов плагинов. Все они очень похожи, но
хранятся отдельно, поэтому у нас могут быть строгие контракты, по одному, что разрешено каждому
делать.

Есть несколько правил, которые следует соблюдать для каждого типа плагина:

- **Stateless** — Избегайте любых состояний, они могут быть источником ошибок для ваших пользователей. Например, одно и то же преобразование может существовать у многих работников, которым запрещено общаться друг с другом, - заявляю, что это не будет работать должным образом.
- **Pure** — Учитывая тот же ввод, плагин должен производить тот же вывод, и
  у вас не должно быть никаких наблюдаемых побочных эффектов или неявных зависимостей.
  В противном случае кеширование Parcel сломается, и вашим пользователям будет грустно.  
  Вам не следует указывать пользователям удалять их кеши.

Все API-интерфейсы плагинов имеют общую форму:

```js
import { NameOfPluginType } from "@parcel/plugin";

export default new NameOfPluginType({
  async methodName(opts: JSONObject): Promise<JSONObject> {
    return result;
  },
});
```

Они состоят из модулей с хорошо известными именованными экспортами асинхронных функций,
которые:

- Принимают строго проверенный JSON-сериализуемый объект `opts`.
- Возвращают строго проверенный JSON-сериализуемый объект `vals`.

Если что-то, что вам нужно, не проходит через `opts`, пожалуйста, поговорите с
командой Parcel об этом. Избегайте попыток получить информацию из других
исходников, особенно из файловой системы.

## Именование

Все плагины должны соответствовать системе именования:

<div style="font-size: 0.9em">

|              | Official package             | Community packages          | Private company/scoped team packages |
| ------------ | ---------------------------- | --------------------------- | ------------------------------------ |
| Configs      | `@parcel/config-{name}`      | `parcel-config-{name}`      | `@scope/parcel-config[-{name}]`      |
| Resolvers    | `@parcel/resolver-{name}`    | `parcel-resolver-{name}`    | `@scope/parcel-resolver[-{name}]`    |
| Transformers | `@parcel/transformer-{name}` | `parcel-transformer-{name}` | `@scope/parcel-transformer[-{name}]` |
| Bundlers     | `@parcel/bundler-{name}`     | `parcel-bundler-{name}`     | `@scope/parcel-bundler[-{name}]`     |
| Namers       | `@parcel/namer-{name}`       | `parcel-namer-{name}`       | `@scope/parcel-namer[-{name}]`       |
| Runtimes     | `@parcel/runtime-{name}`     | `parcel-runtime-{name}`     | `@scope/parcel-runtime[-{name}]`     |
| Packagers    | `@parcel/packager-{name}`    | `parcel-packager-{name}`    | `@scope/parcel-packager[-{name}]`    |
| Optimizers   | `@parcel/optimizer-{name}`   | `parcel-optimizer-{name}`   | `@scope/parcel-optimizer[-{name}]`   |
| Reporters    | `@parcel/reporter-{name}`    | `parcel-reporter-{name}`    | `@scope/parcel-reporter[-{name}]`    |
| Validators   | `@parcel/validator-{name}`   | `parcel-validator-{name}`   | `@scope/parcel-validator[-{name}]`   |

</div>

`{Name}` должен быть описательным и иметь прямое отношение к цели
пакета. Кто-то должен иметь представление о том, что делает пакет, просто
прочитав имя в файле `.parcelrc` или `package.json#devDependencies`.

```
parcel-transformer-posthtml
parcel-packager-wasm
parcel-reporter-graph-visualizer
```

Если ваш плагин добавляет поддержку определенного инструмента, используйте имя
инструмента.

```
parcel-transformer-es6 (bad)
parcel-transformer-babel (good)
```

Если ваш плагин является повторной реализацией чего-то существующего, попробуйте назвать его
именем, объясняющим причину его создания:

```
parcel-transformer-better-typescript (bad)
parcel-transformer-typescript-server (good)
```

Мы просим членов сообщества работать вместе, и когда форки могут пробовать, - 
разрешите их. Если кто-то сделал более удачную версию вашего плагина, - рассмотрите его,
давая лучшее имя пакета, попросите их сделать более крупную версию, и
перенаправьте этих людей на новый инструмент.

## Управление версиями

Вы должны следовать семантическому управлению версиями (насколько это возможно).  
- Нет, это не столь идеальная система, но она самая лучшая, и люди действительно от нее зависят.

Если авторы плагина намеренно не следуют семантическому управлению версиями, Parcel может
начать предупреждать пользователей о том, что они должны заблокировать номер версии для
вашего плагина.

## Двигатели

Вы должны указать поле `package.json#engines.parcel` с диапазоном версий Parcel, которые поддерживают ваш плагин:

```json
{
  "name": "parcel-transform-imagemin",
  "engines": {
    "parcel": "2.x"
  }
}
```

Если вы не укажете это поле, Parcel выдаст предупреждение:

```
Warning: The plugin "parcel-transform-typescript" needs to specify a
`package.json#engines.parcel` field with the supported Parcel version range.
```

Если вы указали поле машины посылок, а пользователь использует несовместимую
версию Parcel, они увидят ошибку:

```
Error: The plugin "parcel-transform-typescript" is not compatible with the
current version of Parcel. Requires "2.x" but the current version is "3.1.4"
```

Parcel использует `node-semver` для сопоставления диапазонов версий.
