# edge-plugins

Plugins for [Frontcache](https://github.com/eternita/frontcache) nodes: code from a jar of its own that a node loads
from `FRONTCACHE_HOME/plugins/<id>/`, built against the Apache-2.0 plugin API
(`org.frontcache:frontcache-plugin-api`) and tested with the plugin testkit (`org.frontcache:frontcache-plugin-testkit`),
both from `https://repo.eternita.co/maven2`. Plugins need a Frontcache 2.13.0 node or later.

| directory | what |
|---|---|
| [plugin-template/](plugin-template/) | a starting point for a plugin of your own: one of each kind (decide, observe, serve), one shaded jar, a `sha256` task, a CI workflow. Its README is the guide to writing a plugin - the class-loader rule, the manifest, the kinds, the testkit |
| [plugins/debug/](plugins/debug/) | `/fc-x/debug/`: how the node sees your request - the resolved client address, client type, the headers a plugin's view shows - and the guard's recent decisions about your address (loopback only by default) |
| [plugins/crawler-feeder/](plugins/crawler-feeder/) | keeps `conf/lists/googlebot.txt` / `bingbot.txt` / `applebot.txt` current from the vendors' published ranges plus claimants confirmed by reverse DNS, for `client-ip:@lists/...` guard and bot rules |

Each directory is a **standalone Gradle build**: copy one out into a repository of its own and it still builds.

## Build

Java 25. From the root, every plugin:

```sh
./gradlew build          # compile, test (PluginContract included), each plugin's build/libs/<name>-<version>.jar
./gradlew sha256         # every jar's SHA-256, for a node that pins it (front-cache.plugins.<id>.sha256)
```

or one plugin, from its own directory (`../../gradlew build` in `plugins/debug/`, or with a wrapper of its own).

`frontcacheVersion` in each plugin's `gradle.properties` is the Frontcache release the API and the testkit are
taken from. A plugin runs on every node of the plugin-api major its manifest names (`plugin-api = 1`), not only
that release. Against an unreleased core: `./gradlew publishToMavenLocal` in the Frontcache checkout, set
`frontcacheVersion` to its version, then `./gradlew build -PfrontcacheMavenLocal`.

## Installing a plugin on a node

Put the jar in `FRONTCACHE_HOME/plugins/<id>/`, enable it by name in `conf/frontcache.properties`, and restart the
node:

```properties
front-cache.plugins = debug
front-cache.plugins.debug.sha256 = <./gradlew sha256>
```

Each plugin's README has the details. The guard-rule side - `ext:` conditions, `challenge`, `/fc-x/`, lists a plugin
keeps - is in Frontcache's
[guard-getting-started.md](https://github.com/eternita/frontcache/blob/main/docs/guard-getting-started.md) (§4.2, §4.3).

## License

Apache-2.0 - see [LICENSE](LICENSE).
