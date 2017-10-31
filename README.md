# <img src="https://user-images.githubusercontent.com/2548061/30693332-c07f5844-9ed7-11e7-970d-17498f7c8e62.png" height="24" alt="Super R" /> SuperRepo

Repository-ish pattern for your data, that implements **best practices** for working with and storing data on the client-side. **Please read [my CSS-Tricks post first](https://css-tricks.com/importance-javascript-abstractions-working-remote-data/).**

> TL;DR: Get ready to make the way you work with and store remote data more maintainable & scalable! :dark_sunglasses: 

## :package: Install

This package can be installed with:

- [npm](https://www.npmjs.com/package/super-repo): `npm install --save super-repo`
- [bower](https://bower.io/search/?q=super-repo): `bower install --save super-repo`
- ... or simply download the [latest release](https://github.com/superKalo/repository/releases).


## :rocket: Load

- Static HTML:

    ```html
    <script src="node_modules/super-repo/lib/index.js"></script>
    ```

- Using ES6 Imports:

    ```javascript
    // If a transpiler is configured (like Traceur Compiler, Babel, Rollup or Webpack):
    import SuperRepo from 'super-repo';
    ```

- Using CommonJS Imports:
    ```javascript
    // If a module loader is configured (like RequireJS, Browserify or Neuter):
    const SuperRepo = require('super-repo');
    ```

## :bulb: Usage

Let's assume we have a weather API. It returns the temperature, feels-like, wind speed (m/s), pressure (hPa) and humidity (%). A common pattern, in order for the JSON response to be as slim as possible, attributes are compressed up to the first letter. So here’s what we receive from the server:

```json
{
    "t": 31,
    "w": 32,
    "p": 1011,
    "h": 38,
    "f": 32
}
```

Now let's define a function responsible for getting data from the server. It doesn't matter how, as long as your function returns a Promise.

- You can use jQuery's `$.ajax()` (as of v1.5, jQuery implements the Promise interface):

    ```javascript
    const requestWeatherData = () => $.ajax( {url:'weather.json'} );
    ```

- ... or FetchAPI:

    ```javascript
    const requestWeatherData = () => fetch('weather.json').then(r => r.json());
    ```

- ... or plain XMLHttpRequest or whatever you want. **As long as you return a Promise it will work!**

We're ready to define our `SuperRepo`sitory:

```javascript
/**
 * 1. Define where you want to store the data, in this example, in the LocalStorage.
 *
 * 2. Then - define a name of your data repository, it's used for the LocalStorage key.
 *    Should be unique!
 *
 * 3. Define when the data will get out of date.
 *
 * 4. Define your data model and set custom attribute names for each response items.
 *    In the example, server returns the params 't', 'w', 'p',
 *    we map them to 'temperature', 'windspeed', and 'pressure' instead.
 *    Remember why? See below.
 */
const WeatherRepository = new SuperRepo({
    storage: 'LOCAL_STORAGE',                   // [1]
    name: 'weather',                            // [2]
    outOfDateAfter: 5 * 60 * 1000, // 5 min     // [3]
    request: requestWeatherData,                // Function that returns a Promise
    dataModel: {                                // [4]
        temperature: 't',
        windspeed: 'w',
        pressure: 'p'
    }
});

/**
 * From here on, you can use the `.getData()` method to access your data.
 * It will first check if out data outdated (based on the `outOfDateAfter`).
 * If so - it will do a server request to get fresh data,
 * otherwise - it will get it from the cache (Local Storage).
 */
WeatherRepository.getData().then( data => {
    // Do something awesome.
    console.log(`It is ${data.temperature} degrees`);
});
```

Why **`[4]`** this is a good idea (best practice):

- Throughout your codebase via `WeatherRepository.getData()` you access meaningful and semantic attributes like `.temperature` and `.windspeed` instead of `t` and `s`.
- You expose only parameters you need and simply don't include the others.
- If the response attributes names change (or you need to wire-up another API with different response structure), you only need to tweak it here - in only 1 place of your codebase.

## :dark_sunglasses: Features

The library brings the following benefits:

- Performance:
  - Gets data from the server (if it’s missing or outdated on our side) or otherwise - gets it from the cache.
  - If `WeatherRepository.getData()` is called multiple times from different parts of our app, only 1 server request is triggered.
- Scalability:
  - Applies the data model to our rough data (see **`[4]`** above).
  - You can store the data in the Local Storage or in the Browser (local) Storage (if you’re building a browser extension) or in a local variable (if you don't want to store data across browser sessions). See the options for the `storage` setting.
  - You can initiate an automatic data sync with `WeatherRepository.initSyncer()`. This will initiate a setInterval, which will countdown to the point when the data is out of date (based on the `outOfDateAfter` value) and will trigger a server request to get fresh data. Sweet.

... and a few more. Read the documentation for advanced usage.

## :scream: Dependencies

None.

## :open_book: Documentation

### Options

#### `name` [required]
Type: `String`

Name of the Repository. Should be unique! It's used for Local Storage or Browser Local Storage item name.

#### `request` [required]
Type: `Function` that returns a `Promise`, that resolves to `Array` or `Object`, based on your server response :nerd_face: 

The request that does the actual API call. It must be a Promise. Use FetchAPI or jQuery's $.ajax() or plain XMLHttpRequest or whatever you want, but wrap it in a Promise (if it isn't).
    
- You can use jQuery's `$.ajax()` (as of v1.5, jQuery implements the Promise interface):

    ```javascript
    const requestWeatherData = () => $.ajax( {url:'weather.json'} );
    
    const WeatherRepository = new SuperRepo({
        /* ... */
        request: requestWeatherData
    });
    ```

- ... or FetchAPI:

    ```javascript
    const requestWeatherData = () => fetch('weather.json').then(r => r.json());
    
    const WeatherRepository = new SuperRepo({
        /* ... */
        request: requestWeatherData
    });
    ```

- ... or plain XMLHttpRequest or whatever you want. **As long as you return a Promise it will work!**

#### `storage` [optional]
Default: `'LOCAL_STORAGE'` | All options: `'LOCAL_STORAGE'`, `'BROWSER_STORAGE'` or `'LOCAL_VARIABLE'`

The preferred client-side storage.

- `'LOCAL_STORAGE'`: Stores data in the [Local Storage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage), if you're building a web app.
- `'BROWSER_STORAGE'`: Stores data in the [Browser (local) Storage](https://developer.mozilla.org/en-US/Add-ons/WebExtensions/API/), if you're building a browser extension.
- `'LOCAL_VARIABLE'`: Stores the data in a local `.data` variable, attached to the class instance, if you don’t want to store data across browser sessions.

#### `dataModel` [optional, but recommended]
Type: `Object` or `Array`

The mapping of the attribute names you'd like to use across your codebase with the attribute names coming from the server response.

- In case the server response is a single Object or an Array of Objects:

  - Syntax 1: In case the server returns a single Object, you can use the following syntax:

    ```javascript
    const WeatherRepository = new SuperRepo({
        /* ... */
        dataModel: {
            temperature: 't',
            windspeed: 'w',
            pressure: 'p'
        }
    });
    ```

  - Syntax 2: In case the server returns an Array of items, you can use the following syntax:

    ```javascript
    const WeatherRepository = new SuperRepo({
        /* ... */
        dataModel: [{
            temperature: 't',
            windspeed: 'w',
            pressure: 'p'
        }]
    });
    ```

- In case the server response is more complex or nested a bit more, use [`mapData`](https://github.com/superKalo/super-repo#mapdata-optional) to apply your data model manually. Here's an example.
  Let's say our Weather API response is:

    ```json
    {
        "current": {
            "main": {
                "t": 31,
                "w": 32,
                "h": 38
            },
            "additional": {
                "f": 32,
                "p": 1011
            }
        }
    }
    ```

    Here's how we handle this case:

    ```javascript
    const WeatherRepository = new SuperRepo({
        /* ... */
        mapData( data => {
            return {
                temperature: data.current.main.t,
                windspeed: data.current.main.w,
                pressure: data.current.additional.p
            }
        });
        // ... and simply don't pass `dataModel`
    });
    ```

#### `outOfDateAfter` [optional]
Default: `-1` | Type: `Number`, in milliseconds

Defines when the data will get out of date.

- Set to `0` if the data will always be up to date once cached.
- Set to `-1` if the data will always be outdated when fetched. If so, probably it will make the most sense to store it in a local variable, instead of in the local storage (the default). Like so:
    ```javascript
    const WeatherRepository = new SuperRepo({
        /* ... */
        storage: 'LOCAL_VARIABLE',
        outOfDateAfter: 0
    });
    ```

#### `mapData` [optional]
Type: `Function`

After the `dataModel` is applied, if you need any further manipulation data manipulation, you can hook on this method. For example, let's say the server sends the temperature in Celsius and you want to convert it to Fahrenheit:

```javascript
const WeatherRepository = new SuperRepo({
    /* ... */
    dataModel: {
        temperature: 't',
        windspeed: 'w',
        pressure: 'p'
    },
    mapData: data => {
        // Convert to Fahrenheit
        const temperature = (data.temperature * 1.8) + 32;

        // These two stays the same
        const { windspeed, pressure } = data;

        return { temperature, windspeed, pressure };
    }
});
```

### Methods

#### `.getData()`
Returns: `Promise`, that resolves to `Array` or `Object`, based on your server response :nerd_face: 

This is how you can access data. It triggers a server request if data is missing or invalidated or out of date. It returns the data from the cache (local storage, browser local storage or local variable) if the data is up to date.

```javascript
const WeatherRepository = new SuperRepo({ /* ... */ });

WeatherRepository.getData().then( data => {
    // Do something.
    console.log(`It is ${data.temperature} degrees`);
});
```

#### `.invalidateData()`
Returns: `Promise`,  that resolves to {Object} that has `prevData` {Object} and the `nextData` {Object}

Invalidates data by setting a flag. It **doesn't delete the data from the storage**. However, the very next time when the `.getData()` method is invoked, it will directly call the server to get fresh data.

```javascript
const WeatherRepository = new SuperRepo({ /* ... */ });

WeatherRepository.invalidateData().then( _response => {
    console.log('Previous data', _response.prevData);
    console.log('Next data', _response.nextData);
});
```

#### `.clearData()`
Returns: `Promise`, that resolves to `prevData` {Object} - the previous (just deleted) data

Deletes the data from the storage. Therefore, the very next time when the `.getData()` method is invoked, it will directly call the server to get fresh data.

```javascript
const WeatherRepository = new SuperRepo({ /* ... */ });

WeatherRepository.clearData().then( _prevData => {
    console.log('Previous (just deleted data) data', _prevData);
});
```

#### `.getDataUpToDateStatus()`
Returns: `Promise`, that resolves to an {Object} with `isDataUpToDate` {Boolean}, `lastFetched` {TimeStamp}, `isInvalid` {Boolean} and `localData` - the currently cached data.

```javascript
const WeatherRepository = new SuperRepo({ /* ... */ });

WeatherRepository.getDataUpToDateStatus().then( _res => {
    const { isDataUpToDate, lastFetched, isInvalid, localData } = _res;

    console.log('Is data up to date?', isDataUpToDate);
    console.log('When the data was last fetched?', lastFetched);
    console.log('Is data marked as invalid?', isInvalid);
    console.log('Currently cached data', localData);
});
```

#### `.initSyncer()`
Returns: `Void`

Initiates a countdown-er to the point when the data gets out of date (based on the `outOfDateAfter` value) and triggers a (network) request to get fresh data.

```javascript
const WeatherRepository = new SuperRepo({
    outOfDateAfter: 5 * 60 * 1000 // 5 min
    /* ... */
});

WeatherRepository.initSyncer();
```

There are 3 edge cases that prevent network (performance) overhead:

- if `outOfDateAfter` value is not set (default value is `-1`), it will trigger a sync on every 1 second.
- if `outOfDateAfter` value is `0`, it will trigger a sync on every 1 second.
- if `outOfDateAfter` value is less than `1000` (1 second), it will trigger a sync on every 1 second instead.

#### `.destroySyncer()`
Returns: `Void`

Destroys the setInterval, initiated by the `.initSyncer()` method.

```javascript
const WeatherRepository = new SuperRepo({
    outOfDateAfter: 5 * 60 * 1000 // 5 min
    /* ... */
});

WeatherRepository.initSyncer();

// La la la la la

// Do not sync anymore.
WeatherRepository.destroySyncer();
```


## :tv: Browser Support

SuperRepo is compiled using [Babel](https://babeljs.io/) to enable support for [ES5 browsers](http://caniuse.com/#feat=es5).

At present, we officially aim to support the last two versions of the following browsers:

- Chrome
- Firefox
- Safari
- Opera
- Edge

If you need to support Internet Explorer 11 or any older browser, you need to include a polyfill like [polyfill.io](https://polyfill.io/v2/docs/) or [babel-polyfill](https://babeljs.io/docs/usage/polyfill/) that emulates a full ES2015+ environment. Make sure you import the polyfill upfront.

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-polyfill/6.26.0/polyfill.min.js"></script>
<script src="/node_modules/super-repo/lib/index.js"></script>

<script>
    const WeatherRepository = new SuperRepo({ /* ... */ });
</script>
```

Even though SuperRepo can be used in older browsers, please note that our focus will always be on the modern-ish browsers listed above.


## :+1: Contributing
I'm open to ideas and suggestions! If you want to contribute or simply you've caught a bug - you can either open an issue or clone the repository, tweak the `src/index.js` file and fire a Pull Request.

To install the project, first make sure you have NodeJS and NPM installed. Preferably, the latest versions, but anything not extremely old should work too. Then, simply run `npm install`.

To generate the distribution files (in the `lib/` directory), do `npm run build`.

To run the tests and generate a test report (both, as output in the CLI and as HTML in the `coverage/` directory), do `npm run test`.

> One little gotcha: in order for the test report to show the actual code line numbers (from `src/index.js`), rather than the code lines of the compiled one (from `src/index.js`), the test command first generates a build with a sourcemap inlined (in `lib/index.js`), then runs the actual tests and finally - undoes the changes into `lib/index.js` (`git checkout lib/index.js`).
> 
> Therefore, do not wonder why in case you do any changes to the distribution file in `lib/index.js`, after running the tests - they disappear!

## :zap: Projects Powered by <img src="https://user-images.githubusercontent.com/2548061/30693332-c07f5844-9ed7-11e7-970d-17498f7c8e62.png" height="20" alt="Super R" /> SuperRepo
- [Irrationally Yours Chrome Extension](https://www.producthunt.com/posts/irrationally-yours-chrome-extension)
- [Crypto Tab](https://crypto-tab.com)
- ... want your project showcased here? Just shoot me an [e-mail](mailto://me@superkalo.com)!

## :oncoming_police_car: License
The code and the documentation are released under the [MIT License](https://github.com/superKalo/repository/blob/master/LICENSE).
