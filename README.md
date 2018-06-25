# Closure Tools Helper

This module contains closure tools compiled `jar`s and various helper function to make dependency management easier.

This module exports 4 functions:
```js
const closureTools = require('closure-tools-helper');

closureTools.compiler(...);
closureTools.templates(...);
closureTools.stylesheets(...);
closuretools.extractTemplateMsg(...);
```

## Closure Compiler api

The closure compiler api abstracts the module system of closure compiler to something based on entry points, much similar to rollup.

```js
(await closureTools.compiler(baseCompilerFlags, sourceGlob, entryPoints)).src()
    .pipe(...)
```
Note the requirements of `await`, due to building compiler flag being an async operation.
The plugin controls `--module` and `-js` compiler flags. All the other flags must be provided with `baseCompilerFlags` argument.

`sourceGlob` is a glob pattern that provides js sources to the compiler. All files referenced by entryPoints must be provided by the glob.

`entryPoints` is an array where each element follows the below interface.
```ts
interface ICompilerEntryPoint {
    /**
     * This is what is used in `goog.require()`.
     * `null` means there is no entry module, but it can have some extraSources.
     */
    id:string|null,
    /**
     * module name, used in specifying dependencies, and also used in output file name.
     */
    name:string,
    /**
     * Array of module names that this bundle depends on.
     */
    deps:string[],
    /**
     * Any files that are not reachable via `goog.require`s but still need to be provided
     * to the compiler.
     */
    extraSources?:string[]
}
```


## Closure Templates api

```js
const compiler = require('closure-tools-helper');

gulp.task('build-soy', () => {
    return compiler.templates([.../* Command line args passed to SoyToJsSrcCompiler.jar */])
        .src()
        .pipe(gulp.dest('build'));
});

gulp.task('extract-soy-messages', () => {
    return compiler.extractTemplateMsg([.../* Command line args passed to SoyMsgExtractor.jar */])
        .src();
});
```

Compiler apis returns a readable stream of the stdout. Users of the package should explicitly call `.src()` in order to turn the stream into flowing mode (same as `google-closure-compiler` gulp plugin).

### Runtime i18n

Closure templates by default supports compile-time i18n. Provide a second argument `TemplatesI18nOptions` to post-process js files generated by soy compiler to enable i18n. Its interface is what follows:
```ts
interface ITemplatesI18nOptions {
    /**
     * This is a string to replace `goog.getMsg` with.
     */
    googGetMsg:string
    /**
     * This is a string that will be appended right after the `goog.module(..)` expression.
     */
    header?:string
    /**
     * This must match files generated by the compiler jar.
     */
    inputGlob:string|string[]
    outputPath?:string // If provided, files will be written to `outputPath/fileName`.
}
```

If runtime i18n is enabled, it will additionally transform legacy-style `goog.provide` modules into [`goog.module`](https://github.com/google/closure-library/wiki/goog.module:-an-ES6-module-like-alternative-to-goog.provide), so that a consumer can import the compiled templates by using `const templates = goog.require('namespace.to.the.template')`, or `import templates from 'goog:namespace.to.the.template'` if the consumer is using [tsickle](https://github.com/angular/tsickle).

## SoyUtils

Import or reference `./third-party/soyutils.js` or `./third-party/soyutils_usegoog.js`.

## Closure Stylesheets api

```js
const compiler = require('closure-tools-helper');

gulp.task('build-gss', () => {
    return compiler.stylesheets([.../* Command line args passed to closure-stylesheets.jar */])
        .src()
})
```

Compiler apis returns a readable stream of the stdout. User of the package should explicitly call `.src()` to turn the stream into a flowing mode.

## Installation

```
yarn add https://github.com/seanl-adg/closure-tools-helper.git
yarn add https://github.com/seanl-adg/closure-tools-helper.git#<specific-tag>
```

## Build
```
tsc
```