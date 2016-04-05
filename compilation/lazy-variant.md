- Introduction
- Changes in respect to the previous version
- Adaptation of .js librariess
- Controllers and Components
- Libraries and Executables
- Compiling Libraries & Executables
- Compiling The Project

## Introduction
After doing research I found out that we can choose between two approaches of Glyphr Studio compilation, those are: typed and lazy. The two approaches differ only in the way third-party libraries are adoped. 

In the case of the lazy variant we use Babel and straightforwardly reference the third-party dependecies (.js files) in index-dev.html. This option is quick but doesn't support IDE type hinting out of the box. However, we can use JSDoc to make Webstorm provide some kind of type hinting for a particular part of the code. The libraries we're going to use are not likely to feature JSDoc comment blocks out of the box. 

Using the typed variant we need to perform an extra step of writing TypeScript definition files for each library in exchange for type hinting within the IDE which is supported by Webstorm out of the box. Defining a .js library takes few hours of work although for popular libraries definitions already exist. When this step is omitted then the IDE will report errors (false positives) and there will be no hinting support for those libraries; which destroys the purpose of using TypeScript at all.

> note 1: TypeScript doesn't support including .js files in .ts (TypeScript) files, for obvious reasons.

When we decide upon the typed option and there will be no choice than to write ourselves definition for each library we're going to use, we'll also have to keep that definition up-to-date.

In futher parts of this post we'll discuss the lazy variant, because it will play nicely with the IDE and allow us to quickly adapt third-party code.

Changes in respect to the previous version
Both the libraries and exectuables are compiled using Webpack to prevent stale dependencies in the script-tags. Webpack won't include a libarary when no executable `import`s it.

Executables are restricted to be located under a single directory, separately from libraries. This will ensure that although executables may `import` libraries, executables can still be automatically compiled into a single file, separately from the libraries just by specifying the directory they're located in; whenever such a need arises.

## Adaptation of .js libraries
The library is going to be registered with npm and included using the `import` tag. When compiling Glyphr Studio to the dist version a Webpack will traverse through the dependencies and include them altogether with the executables into a single bundle.js file.

## Controllers and Components
The implementation will be based on Ractive.js although, we need controllers and components to be able to structure our application.

A component is an instance of Ractive with specified template (html) and behaviour (event handling). Each component is located inside the component directory and has its own file. However, to separate our templates from the behaviour of the component we need another custom compiler. This complier will traverse through the component directory, look for  '@include path/to/view.html' strings and replace them with the contents of the referenced html-file, surrounded by multiline-inverted-primes (`). Therefore, it must also escape the "`"-symbols within these contents.

A controller is an instance of the Controller class. Controller is called by the Router and makes sure that a component(s) finds its place in the DOM, and that it receives all the information it needs to function – a controller takes care of a view.

> note 2: Routing is not disscused in this post, however it is a part of the specification.

> note 3: A view is what user sees under a specific URL or hash.

Therefore, controller's implementation must be minimal; this can be acquired by creating libraries which will do the computations for the controller. These libraries might be ours or third-party and can be shared across many controllers. In this sense a component might be considered a library.

## Libraries and Executables
A library is a piece of code, an implementation, altogether with all of its dependencies which is used in the implementation of a controller. Libraries can be compiled in an arbitrary order as they don't have any dependencies (as they are self-contained). A library must not depend on an executable.

An executable is a piece of code which can run only when all of the libraries are already included, executables are files located under the glyphr-studio directory. An executable may depend on libraries and reference them using the `import` tags. Furthermore, it may depend on another executables and reference them using the `import` tags. All executables must be located under a single directory separately from any library. 

> note 4: Configuartion files are not disscused in this post, however they are a part of the specification.

## Compiling Executables & Libraries
Compilation of executables and libraries is performed with Webpack which will enter  main.js and concat the executables and the corresponding libraries, by resolving the dependencies, into a single bundle.js file.

## Compiling The Project
Compilation of the project is performed with Grunt which will include bundle.js into a single html-file.

The compilation process must result in one single html file.
