---
layout: layout.njk
eleventyNavigation:
  key: plugin-system-source-maps
  title: Source Maps
  order: 15
---

Parcel использует пакет `@parcel/source-maps` для обработки всех исходных карт, чтобы обеспечить производительность и надежность при манипулировании исходными картами через плагины и ядро ​​Parcel. Эта библиотека была написана с нуля на C++ с учетом как манипулирования исходной картой, так и конкатенации, и дала нам 20-кратное повышение производительности по сравнению с нашим старым решением с использованием Mozilla. [`source-map`](https://github.com/mozilla/source-map) библиотека и некоторые внутренние утилиты. Это улучшение производительности в основном связано с оптимизацией структур данных и способом кэширования исходных карт.

## Как пользоваться библиотекой

Чтобы использовать библиотеку, вы начинаете с создания экземпляра экспортированного класса SourceMap, для которого вы можете вызывать различные функции для добавления и редактирования сопоставлений источников.

Ниже приведен пример, охватывающий все способы добавления сопоставлений в экземпляр `SourceMap`:

```js
import SourceMap from "@parcel/source-map";

let sourcemap = new SourceMap();

// Каждая функция, которая добавляет сопоставления, имеет необязательные аргументы смещения.
// Их можно использовать для компенсации сгенерированных сопоставлений на определенную величину.
let lineOffset = 0;
let columnOffset = 0;

// Добавить индексированные сопоставления
// это сопоставления, которые иногда можно извлечь из библиотеки еще до того, как они будут преобразованы в сопоставления VLQ
sourcemap.addIndexedMappings(
  [
    {
      generated: {
        // индекс строки начинается с 1
        line: 1,
        // индекс столбца начинается с 0
        column: 4,
      },
      original: {
        // индекс строки начинается с 1
        line: 1,
        // индекс столбца начинается с 0
        column: 4,
      },
      source: "index.js",
      // Имя не обязательно
      name: "A",
    },
  ],
  lineOffset,
  columnOffset
);

// Добавьте необработанные сопоставления, это то, что будет выводиться в исходную карту в кодировке vlq
sourcemap.addRawMappings(
  {
    file: "min.js",
    names: ["bar", "baz", "n"],
    sources: ["one.js", "two.js"],
    sourceRoot: "/the/root",
    mappings:
      "CAAC,IAAI,IAAM,SAAUA,GAClB,OAAOC,IAAID;CCDb,IAAI,IAAM,SAAUE,GAClB,OAAOA",
  },
  lineOffset,
  columnOffset
);

// Исходные карты можно сохранять в виде буферов (плоских буферов), это то, что мы используем для кеширования в Parcel.
// Вы можете создать экземпляр SourceMap с этими значениями буфера, используя функцию `addBufferMappings`.
let originalMapBuffer = new Buffer();
sourcemap.addBufferMappings(originalMapBuffer, lineOffset, columnOffset);
```

## Преобразования/манипуляции

Если ваш плагин выполняет какие-либо манипуляции с кодом, вы должны убедиться, что он создает правильные сопоставления с исходным исходным кодом, чтобы гарантировать, что мы все равно создадим точную карту исходного кода в конце процесса объединения. Ожидается, что вы вернете экземпляр SourceMap в конце преобразования в плагине [Transformer](/plugin-system/transformer/). Мы также предоставляем исходную карту из предыдущего преобразования, чтобы гарантировать соответствие исходному  коду, а не только выходным данным предыдущего преобразования.

Значение `asset`, которое передается в функциях `parse`, `transform` и `generate` плагина-преобразователя, содержит функцию, называемую `getMap()` и `getMapBuffer()`, эти функции могут использоваться для получения Экземпляр SourceMap (`getMap()`) и кешированный буфер SourceMap (`getMapBuffer()`).

Вы можете свободно манипулировать исходной картой на любом из этих шагов в преобразователе до тех пор, пока вы убедитесь, что исходная карта, которая возвращается в `generate`, правильно отображает исходный файл.

Ниже приведен пример того, как манипулировать исходными картами в плагине трансформатора:

```js
import { Transformer } from "@parcel/plugin";
import SourceMap from "@parcel/source-map";

export default new Transformer({
  // ...

  async generate({ asset, ast, resolve, options }) {
    let compilationResult = dummyCompiler(await asset.getAST());

    let map = null;
    if (compilationResult.map) {
      // Если compilationResult вернул карту, мы преобразуем ее в экземпляр Parcel SourceMap.
      map = new SourceMap();

      // Фиктивный компилятор вернул полную закодированную исходную карту с сопоставлениями vlq
      // Некоторые компиляторы могут иметь возможность возвращать indexedMappings, что может улучшить производительность (например, Babel).
      // в общем, каждый компилятор может возвращать rawMappings, поэтому всегда лучше использовать это
      map.addRawMappings(compilationResult.map);

      // Получаем исходный буфер карты из ассета, чтобы расширить наши сопоставления поверх него 
      // и чтобы убедиться, что мы сопоставляемся с исходным источником вместо предыдущего преобразования
      let originalMapBuffer = await asset.getMapBuffer();
      if (originalMapBuffer) {
        // Функция `extends` использует предоставленную карту для переназначения исходных исходных позиций карты, на которой она вызывается.
        // Таким образом, в этом случае исходные исходные позиции `map` переназначаются на позиции в `originalMapBuffer`
        map.extends(originalMapBuffer);
      }
    }

    return {
      code: compilationResult.code,
      // Обязательно верните карту
      // это нужно для объединения исходных карт вместе в исходную карту финального пакета
      map,
    };
  },
});
```

Если ваш компилятор поддерживает возможность передачи существующей карты источников, вы также можете использовать ее, поскольку это может привести к получению более точных/лучших карт источников, чем использование метода в предыдущем примере.

Пример того, как это будет работать:

```js
import { Transformer } from "@parcel/plugin";
import SourceMap from "@parcel/source-map";

export default new Transformer({
  // ...

  async generate({ asset, ast, resolve, options }) {
    // Получите исходную карту из актива
    let originalMap = await asset.getMap();
    let compilationResult = dummyCompiler(await asset.getAST(), {
      // Передайте кодированную VLQ версию originalMap компилятору
      originalMap: originalMap.toVLQ(),
    });

    // В этом случае компилятор отвечает за отображение на исходные позиции, указанные в originalMap.
    // поэтому мы можем просто преобразовать его в Parcel SourceMap и вернуть
    let map = new SourceMap();
    if (compilationResult.map) {
      map.addRawMappings(compilationResult.map);
    }

    return {
      code: compilationResult.code,
      map,
    };
  },
});
```

## Concatenating sourcemaps in Packagers

Если вы пишете собственный упаковщик, вы несете ответственность за объединение исходных карт всех ресурсов при их упаковке. Это делается путем создания нового экземпляра `SourceMap` и добавления к нему новых сопоставлений с помощью функции `addBufferMappings(buffer, lineOffset, columnOffset)`. `lineOffset` должен быть равен индексу строки, с которой начинается вывод актива.

Ниже приведен пример того, как это сделать:

```js
import { Packager } from "@parcel/plugin";
import SourceMap from "@parcel/source-map";

export default new Packager({
  async package({ bundle, options }) {
    // Мы создаем экземпляр переменной содержимого, которая будет содержать строку, представляющую весь выходной пакет.
    let contents = "";

    // Мы создаем новую карту SourceMap, в которую добавим все карты ресурсов.
    let map = new SourceMap();

    // Это очередь, которая считывает все содержимое файла и отображает и сохраняет их для использования в фактической упаковке.
    let queue = new PromiseQueue({ maxConcurrent: 32 });
    bundle.traverse((node) => {
      if (node.type === "asset") {
        queue.add(async () => {
          let [code, mapBuffer] = await Promise.all([
            node.value.getCode(),
            bundle.target.sourceMap && node.value.getMapBuffer(),
          ]);
          return { code, mapBuffer };
        });
      }
    });

    let i = 0;
    // Обработать всю очередь...
    let results = await queue.run();

    // Мы проходим по бандлу и добавляем содержимое каждого актива к содержимому, а mapBuffer - к карте.
    bundle.traverse((node) => {
      if (node.type === "asset") {
        // Получить данные из очереди результатов
        let { code, mapBuffer } = results[i];

        // Добавьте вывод к содержимому
        let output = code || "";
        contents += output;

        // Если Parcel требует исходных карт, мы добавляем mapBuffer на карту
        if (options.sourceMaps) {
          if (mapBuffer) {
            // мы добавляем mapBuffer на карту с помощью lineOffset
            // LineOffset равен строке, в которой начинается содержимое актива.
            // что такое же, как длина содержимого до добавления этого ресурса
            map.addBufferMappings(mapBuffer, lineOffset);
          }

          // Добавляем количество строк текущего актива в lineOffset
          // таким образом мы узнаем длину `contents`, не пересчитывая ее каждый раз
          lineOffset += countLines(output) + 1;
        }

        i++;
      }
    });

    // Верните содержимое и карту, чтобы Parcel Core мог сохранить их на диск или подвергнуть пост-обработке оптимизаторам.
    return { contents, map };
  },
});
```

### Объединение AST

Если вы объединяете AST вместо исходного содержимого, у вас уже есть сопоставления источников, встроенные в AST, которые вы можете использовать для генерации окончательной исходной карты. Однако вы должны убедиться, что эти сопоставления остаются нетронутыми при редактировании узлов, иногда это может быть довольно сложно, если вы делаете много изменений.

Пример того, как это работает:

```js
import { Packager } from "@parcel/plugin";
import SourceMap from "@parcel/source-map";

export default new Packager({
  async package({ bundle, options }) {
    // Выполните конкатенацию AST и верните скомпилированный результат
    let compilationResult = concatAndCompile(bundle);

    // Создайте окончательную упакованную исходную карту
    let map = new SourceMap();
    if (compilationResult.map) {
      map.addRawMappings(compilationResult.map);
    }

    // Верните скомпилированный код и карту
    return {
      code: compilationResult.code,
      map,
    };
  },
});
```

## Постобработка исходных карт в оптимизаторах

Использование исходных карт в оптимизаторах идентично тому, как вы используете их в преобразователях, поскольку вы получаете один файл в качестве ввода и, как ожидается, вернете тот же файл в качестве вывода, но оптимизированный.

Единственная разница с оптимизаторами заключается в том, что карта предоставляется не как часть актива, а как отдельный параметр/опция, как вы можете видеть во фрагменте кода ниже. Как всегда, карта является экземпляром класса `SourceMap`.

```js
// Содержание и карта передаются отдельно
async optimize({ bundle, contents, map }) {
  return { contents, map }
}
```

## Диагностика проблем

If you encounter incorrect mappings and want to debug these mappings we have built tools that can help you diagnose these issues. By running a specific reporter (`@parcel/reporter-sourcemap-visualiser`), Parcel create a `sourcemap-info.json` file with all the necessary information to visualize all the mappings and source content.

Если вы сталкиваетесь с неправильными сопоставлениями и хотите отладить эти сопоставления, мы создали инструменты, которые помогут вам диагностировать эти проблемы. Запустив определенный репортер (`@parcel/reporter-sourcemap-visualiser`), Parcel создает файл `sourcemap-info.json` со всей необходимой информацией для визуализации всех сопоставлений и исходного содержимого.

Чтобы включить его, добавьте собственный файл `.parcelrc`:

```json
{
  "extends": "@parcel/config-default",
  "reporters": ["...", "@parcel/reporter-sourcemap-visualiser"]
}
```

После того, как репортер создал файл `sourcemap-info.json`, вы можете загрузить его в [sourcemap visualizer](https://sourcemap-visualiser.now.sh/)

## `@parcel/source-maps`: API

{% include "../../api/source-map.html" %}
