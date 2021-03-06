# apollo-cache-persist

**NOTE: This repo is in 'pre-release' and not quite ready to production use
yet.**

Simple persistence for all Apollo Cache 2.0+ implementations, including
[`apollo-cache-inmemory`][0] and [`apollo-cache-hermes`][1].

Supports web and React Native. [See all storage providers.](#storage-providers)

[0]: https://github.com/apollographql/apollo-client/tree/master/packages/apollo-cache-inmemory
[1]: https://github.com/convoyinc/apollo-cache-hermes

## Basic Usage

To get started, simply pass your Apollo Cache and an
[underlying storage provider](#storage-providers) to `persistCache`.

By default, the contents of your Apollo Cache will be immediately restored
(asynchronously), and will be persisted upon every write to the cache (with a
short debounce interval).

### Examples

#### React Native

```js
import { AsyncStorage } from 'react-native';
import { InMemoryCache } from 'apollo-cache-inmemory';
import { persistCache } from 'apollo-cache-persist';

const cache = new InMemoryCache({...});

persistCache({
  cache,
  storage: AsyncStorage,
});

// Continue setting up Apollo as usual.
```

#### Web

```js
import { InMemoryCache } from 'apollo-cache-inmemory';
import { persistCache } from 'apollo-cache-persist';

const cache = new InMemoryCache({...});

persistCache({
  cache,
  storage: document.localStorage,
});

// Continue setting up Apollo as usual.
```

### Additional Options

`persistCache` and the constructor for `CachePersistor` accept the following
options:

```js
persistCache({
  /**
   * Required
   */

  cache, // Reference to your Apollo Cache.
  storage, // Reference to your storage provider.

  /**
   * Trigger options
   */

  trigger: 'write', // Persist upon every write to the store. Default.
  trigger: 'background', // Persist when your app moves to the background. React Native only.

  // Debounce interval (in ms) between persists.
  // Defaults to 1000 for 'write' and 0 for 'background'.
  debounce: 1000,

  /**
   * Storage options
   */

  // Key to use with the storage provider.
  key: 'apollo-cache-persist',

  // Whether to JSON serialize before/after persisting.
  serialize: true,

  /**
   * Miscellaneous
   */

  // Purges storage & pauses persistence if trigger causes cache size to pass a threshold (in bytes). App will start up cold.
  maxSize?: number,

  // Enable console logging.
  debug: false,
});
```

## Advanced Usage

Instead of using `persistCache`, you can instantiate a `CachePersistor`, which
will give you fine-grained control of persistence.

`CachePersistor` accepts the same options as `persistCache` and returns an
object with the following methods:

```js
const persistor = new CachePersistor({...});

persistor.restore();   // Immediately restore the cache. Returns a Promise.
persistor.persist();   // Immediately persist the cache. Returns a Promise.
persistor.purge();     // Immediately purge the stored cache. Returns a Promise.

persistor.pause();     // Pause persistence. Triggers are ignored while paused.
persistor.resume();    // Resume persistence.
persistor.remove();    // Remove the persistence trigger. Manual persistence required after calling this.

// Obtain the most recent 30 persistor loglines.
// `print: true` will print them to the console; `false` will return an array.

persistor.getLogs(print);

// Obtain the current persisted cache size in bytes. Returns a Promise.
// Resolves to 0 for empty and `null` when `serialize: true` is in use.

persistor.getSize();
```

## Storage Providers

The following storage providers work 'out of the box', with no additional
dependencies:

* `AsyncStorage` on React Native
* `document.localStorage` on web
* `document.sessionStorage` on web
* [`localForage`](https://github.com/localForage/localForage) on web

`apollo-cache-persist` uses the same storage provider API as
[`redux-persist`](https://github.com/rt2zz/redux-persist), so you can also make
use of the providers [listed here](#storage-providers), including:

* [`redux-persist-node-storage`](https://github.com/pellejacobs/redux-persist-node-storage)
* [`redux-persist-fs-storage`](https://github.com/leethree/redux-persist-fs-storage)
* [`redux-persist-cookie-storage`](https://github.com/abersager/redux-persist-cookie-storage)
