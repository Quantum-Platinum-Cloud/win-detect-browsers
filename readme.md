# win-detect-browsers

> Detects installed versions of Chrome, Chromium, Firefox, Phantomjs (local and global), Internet Explorer, Safari and Opera (Stable, Beta and Developer). Supports Windows XP and up.

[![Build status](https://img.shields.io/appveyor/ci/vweevers/win-detect-browsers.svg?style=flat-square)](https://ci.appveyor.com/project/vweevers/win-detect-browsers) [![Dependency Status](https://img.shields.io/david/vweevers/win-detect-browsers.svg?style=flat-square)](https://david-dm.org/vweevers/win-detect-browsers)

[Command line](#command-line) / [API](#api) / [License](#license)

## About

Basically, browser detection on Windows can't be done right. This is the *try-everything-and-fail-silently* approach. It accounts for architecture differences, normalizes environment variables, tries default locations, searches the registry (in the HKLM and HKCU hives), checks [Start Menu Internet Applications](http://msdn.microsoft.com/en-us/library/windows/desktop/dd203067(v=vs.85).aspx) and looks in `PATH`. Version numbers are read from the executable metadata.

`browser-launcher` by substack has [poor Windows support](https://github.com/substack/browser-launcher/issues/7), and it prompted me to create this module. It is now used in [browser-launcher2](https://github.com/benderjs/browser-launcher2), an active fork of `browser-launcher`.

## Upcoming changes

**2.0 is a faster, streaming version of `win-detect-browsers`. It has no breaking changes in the public API but the internal rewrites might break functionality on some systems (most notably XP and 64-bit Windows). If you have 30 seconds to spare, please try it out (with debug enabled) and send me a report. Thank you!**

    npm i vweevers/win-detect-browsers#2.0 -g
    set debug=win-detect-browsers
    win-detect-browsers

Notable changes:

- Speed improvement (2-4x), because it uses a single `cscript` process to query the registry (replaces `reg` queries) and a single `cscript` process to get the version numbers (replaces `wmic` queries).
- Exports a readable object stream, but 1.x callback style is still supported (at least until 3.0)
- Only emits executables (`*.exe`)
- No longer uses command-line version flags or version numbers found in the registry or elsewhere. This simplifies the process and makes the version numbers consistent and more detailed.

## Command line

Install globally and run:

    npm i win-detect-browsers -g
    win-detect-browsers

Example output on 32-bits Windows 7:

    Found 9 browsers in 355.518 ms
    
    chrome 44.0.2383.0
      @ C:\Program Files\Google\Chrome\Application\chrome.exe
    chromium 44.0.2390.0
      @ C:\Users\vweevers\AppData\Local\Chromium\Application\chrome.exe
    firefox 34.0.5.5443
      @ C:\Program Files\Mozilla Firefox\firefox.exe
    ie 11.0.9600.17728
      @ C:\Program Files\Internet Explorer\iexplore.exe
    opera 29.0.1795.47
      @ C:\Program Files\Opera\Launcher.exe
    opera 29.0.1795.41
      @ C:\Program Files\Opera beta\Launcher.exe
    opera 30.0.1835.5
      @ C:\Program Files\Opera developer\Launcher.exe
    phantomjs 1.9.8.0
      @ C:\Program Files\nodejs\node_modules\phantomjs\lib\phantom\phantomjs.exe
    safari 5.34.54.16
      @ C:\Program Files\Safari\Safari.exe

Enable debug with `SET DEBUG=win-detect-browsers`.

### examples

Detect Internet Explorer and Phantomjs, without version numbers:

    win-detect-browsers --no-version ie phantomjs

Return first found version of FF:

    win-detect-browsers --lucky firefox

## API

- Legacy callback mode: `detect([names, options,] cb)` calls `cb` with an array of results.
- Streaming mode: `detect([names, options])` returns a readable stream.

Where `names` is an array of browser names you want to find. If omitted, it will detect all browsers. Available `options` are:

- `boolean lucky` whether to end the search for a browser after the first result. Note that this result is not consistent, because search is asynchronous. Defaults to `false`, meaning: find all versions.
- `boolean version` whether to get version numbers, defaults to `true`.

## examples

Detect *[everything](http://youtu.be/k1yvvNvlXtg)*:

```js
var detect   = require('win-detect-browsers')
  , through2 = require('through2')

detect().pipe(through2.obj(function(b, _, next){
  var format = "\n%s %s\n  @ %s"
  console.log(format, b.name, b.version, b.path)
  next()
})

```

In callback mode:

```js
var detect = require('win-detect-browsers')

detect(function(browsers){
  var format = "\n%s %s\n  @ %s"
  browsers.forEach(function(b){
    console.log(format, b.name, b.version, b.path)
  })
})

```

Detect only Chrome and IE, without version numbers:

```js
detect(['chrome', 'ie'], {version: false})
```

## License and credits

[MIT](http://opensource.org/licenses/MIT) © [Vincent Weevers](http://vincentweevers.nl). Registry stream adapted from [regedit](https://www.npmjs.com/package/regedit) ([MIT](http://opensource.org/licenses/MIT)) © [ironSource](http://www.ironsrc.com/).
