![generator-angular-2-library help guide]

## How to import non Angular libraries into your own project

To begin, first follow the general installation guide in the [README.md](https://github.com/kktam/generator-angular2-library) of generator-angular2-library.

### General installation of new generator-angular-2-library based project

Simply run the generator and install the dependencies to start a new project.

```
yo angular2-library
npm i
npm run build
```

### Structure of the project 

An important note to make about projects created by generator-angular-2-library is that the project contains 2 package.json files. The first one is located at ROOT\package.json and the second one at ROOT\src\package.json. The first package file is responsible to combine the project, like any other Angular project for build, run and distribution etc. So it has all the regular things a standard package.json would have, such as scripts, dependencies and devDependencies.

The second package file, however, is only responsible for describing the packaging steps used for createion of UMD builds, as required by npm repository. For more information about npm's requirement in package.json, please goto [npm package.json explained](https://docs.npmjs.com/cli/build)

### Import non Angular based 3rd party library

We will use FullCalendar.io as an example.

To install FullCalendar.io use yarn or npm to install the npm package

```
npm install fullcalendar
```

This will install the fullcalendar name into the dependencies section of ROOT\package.json. This is important as the library package need to have all its dependencies at any given time in order to compile and run the projects.

After the first install the ROOT\package.json will look like this

```
  "dependencies": {
    "fullcalendar": "^3.4.0"
  }
```

Next open the ROOT\src\package.json

The contents of ROOT\src\package.json are not managed either by npm, or by the generator itself. Therefore, the user must edit the contents of the ROOT\src\package.json manually, and **ensure** the peerDependencies section of the file matches all the contents of the dependencies section of ROOT\package.json.  

A common error is that, during the course of re-usuable component libray development, it is easy to forget to update the ROOT\src\package.json and it is missing a couple of libraries that was in the ROOT\package.json file. As a result, when the re-usable component project is imported into the host application, the running application will throw the following exception.

```
SCRIPT5022: Exception thrown and not caught
File: polyfills.bundle.js, Line: 859, Column: 36
```

In order to run libraries created from other technologies such as JQuery or just using pure Javascript, ES2015 support may be required, especially when those libraries were not ES6 compliant. In order to enable ES2015 support, simply open tsconfig.json, and edit following

```
    "lib": [ 
      "es2015", 
      "es2015.iterable", 
      "dom" 
    ],
```

There is also a default setting that needed to be changed, to support ES2015 by the AOT compiler.
The change is the set annotateForClosureCompiler flag to false for ES2015 dependencies. If the resuable component project does not have any ES5 dependencies, this step is not required.

To change, open the file at ROOT\src\tsconfig.es5.json and edit the following

```
  "angularCompilerOptions": {
    "annotateForClosureCompiler": false,
    ...
  },
```

If the setting is not done correctly, then compiling the library will result in error. This is documented in Angular's [AOT issue](https://github.com/angular/angular/issues/16084)

### Importing CSS provided by 3rd Party library

Most 3rd party library provides a default suite of CSS files to provide a default theme for use with the library. It is useful to have the re-usable component project bundle the necessary CSS files, instead of having the target users of the re-usable component project, having to download or import the CSS files form a hosted CDN, manually import them again from npm, etc.

### Things to add in the Host applications, to support CSS from the Re-usable component library

If you are using angular-cli to create hosted Angular projects, then you will need to create a new styles.css file under src, and import the merged css styles from the imported re-usable component library, like the following

src/styles.css

```
+@import "~fullcalendar-ag4/styles.css";
```

Please note the ~ symbol allows you to specify to the Angular transpiler to search from the nearest node_modules folder. In this case, you do not need to specify relative path, or include "node_modules" in the path.

If everything goes well, you will see your new project with the renewed CSS loaded on the component. 
If you every see the following error from the browser debugger while running the host application, and that would mean the component library CSS is not setup or loaded properly.

```
Uncaught Error: Unexpected value '[object Object]' imported by the module 'AppModule'. Please add a @NgModule annotation.
    at syntaxError (compiler.es5.js:1689)
    at compiler.es5.js:15373
    at Array.forEach (<anonymous>)
    at CompileMetadataResolver.webpackJsonp.../../../compiler/@angular/compiler.es5.js.CompileMetadataResolver.getNgModuleMetadata (compiler.es5.js:15356)
    at JitCompiler.webpackJsonp.../../../compiler/@angular/compiler.es5.js.JitCompiler._loadModules (compiler.es5.js:26679)
    at JitCompiler.webpackJsonp.../../../compiler/@angular/compiler.es5.js.JitCompiler._compileModuleAndComponents (compiler.es5.js:26652)
    at JitCompiler.webpackJsonp.../../../compiler/@angular/compiler.es5.js.JitCompiler.compileModuleAsync (compiler.es5.js:26581)
    at PlatformRef_.webpackJsonp.../../../core/@angular/core.es5.js.PlatformRef_._bootstrapModuleWithZone (core.es5.js:4595)
    at PlatformRef_.webpackJsonp.../../../core/@angular/core.es5.js.PlatformRef_.bootstrapModule (core.es5.js:4581)
    at Object.../../../../../src/main.ts (main.ts:11)
```



