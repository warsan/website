---
layout: layout.njk
eleventyNavigation:
  key: features-parcel-api
  title: 📚 Parcel API
  order: 7
summary: Как программно использовать @parcel/core
---

### Минимальный пример (или "Что делает CLI `parcel`")

{% sample %}
{% samplefile %}

```js
import path from "path";
import defaultConfigContents from "@parcel/config-default";
import Parcel from "@parcel/core";

(async () => {
  let bundler = new Parcel({
    entries: path.join(__dirname, "src/index.js"),
    defaultConfig: {
      ...defaultConfigContents,
      filePath: require.resolve("@parcel/config-default"),
    },
    defaultEngines: {
      browsers: ["last 1 Chrome version"],
      node: "10",
    },
  });

  await bundler.run();
})();
```

{% endsamplefile %}
{% endsample %}

### Вывод в файловую систему в памяти

{% sample %}
{% samplefile %}

```js
import path from "path";
import Parcel, { createWorkerFarm } from "@parcel/core";
import defaultConfigContents from "@parcel/config-default";
import { NodeFS, MemoryFS } from "@parcel/fs";

const DIST_DIR = "/dist";

(async () => {
  let workerFarm = createWorkerFarm();
  let inputFS = new NodeFS();
  let outputFS = new MemoryFS(workerFarm);

  await outputFS.mkdirp(DIST_DIR);

  try {
    let b = new Parcel({
      entries: [path.join(__dirname, "src", "index.html")],
      defaultConfig: {
        ...defaultConfigContents,
        reporters: [],
        filePath: require.resolve("@parcel/config-default"),
      },
      inputFS: inputFS,
      outputFS: outputFS,
      workerFarm,
      defaultEngines: {
        browsers: ["last 1 Chrome version"],
        node: "8",
      },
      distDir: DIST_DIR,
      patchConsole: false,
    });

    await b.run();

    for (let file of await outputFS.readdir(DIST_DIR)) {
      console.log("---------", file, "---------");
      console.log(await outputFS.readFile(path.join(DIST_DIR, file), "utf8"));
    }
  } catch (e) {
    console.error(e);
  } finally {
    await workerFarm.end();
  }
})();
```

{% endsamplefile %}
{% endsample %}
