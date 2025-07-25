<img src="./quill_tp.svg" width="200px" align="right">

### The Quill CLI
The command line interface for the Quill compiler.

### Installation

First, make sure you have C compiler installed on your system (C11 or later, no MSVC).   
Also make sure that you have [Git](https://git-scm.com/downloads) installed.  
Then download the latest release of the CLI as the `quill.c` file in the releases.   
In the directory containing your `quill.c` then simply run:
```
cc quill.c -lm -O3 -o quill
```
(if `cc` is not a recognized command, replace it with the name of your C compiler, e.g. `gcc` or `clang`)   
This produces a `quill` binary that you can put anywhere (make sure to add it to the path)!   

If the above command DID NOT work with `cc`, make sure to also tell which C compiler Quill should use by specifying it with the `QUILL_CC` environment variable (e.g. `QUILL_CC=gcc`).

For Windows I suggest using [w64devkit](https://github.com/skeeto/w64devkit) due to its comparatively small footprint.

### Usage

```
quill new (TYPE) (NAME)
```
(e.g. `quill new application example`)   
Creates a new directory `NAME` containinig a new Quill package of the given type.
`NAME` should only contain `a-Z`, `0-9` and `_` and not start with a numeral.
`TYPE` may be one of the following:
- `application` - A normal executable application.
- `library` - A library made of source code directly included when included.
- `dyn_library` - A library that should be compiled separately to a standalone dynamically linked library when included (which may be loaded using [the `DynLib` type from the `os` package](https://github.com/quill-project/os))
- `plugin` - A compiler plugin containing macros that should be compiled and loaded by the compiler when included.
```
quill init (TYPE)
```
(e.g. `quill init library`)   
Initializes the current directory as a new Quill package of the given type, using the name of the current directory as the package name.
`TYPE` may be one of the following:
- `application` - A normal executable application.
- `library` - A library made of source code directly included when included.
- `dyn_library` - A library that should be compiled separately to a standalone dynamically linked library when included (which may be loaded using [the `DynLib` type from the `os` package](https://github.com/quill-project/os))
- `plugin` - A compiler plugin containing macros that should be loaded by the compiler when included.

```
quill build ('debug'/'release')
```
(e.g. `quill build release`)   
Downloads all package dependencies and compiles the package in the current directory for the backend specified by the package. The output is placed into the `.quill`.   
When compiling for the `c` backend, `release` prioritizes the execution speed of the resulting binary over compilation time.

```
quill clean
```
Deletes the `.quill` directory in the current directory (which contains the build output and downloaded dependencies).

### Environment Variables

- `QUILL_GIT` - The git implementation to use (`git` by default)
- `QUILL_CC` - The C compiler to use (`cc` by default)
- `QUILL_CC_FLAGS` - Additional flags to pass to the C compiler
- `QUILL_NPROC` - The number of cores the Quill compiler should use for compilation (`N - 2` by default, where `N` is the number of cores available)

### Package Configuration

The package configuration is a JSON file called `quill.pkg.json` at the root of the package directory containing an object with the following properties:
- `"name"` - The name of the package. Should only consist of `a-z`, `A-Z`, `0-9`, `-` and `_`.
- `"authors"` - A list of authors.
- `"decription"` - A description of the package.
- `"type"` - The type of the package - `"application"`, `"library"`, `"dyn_library"` or `"plugin"`.
- `"main"` (if `"type": "application"`) - The full path of the main function.
- `"backend"` - The backend to use - `"js"`, `"c"` or `"any"`.
- `"dependencies"` - A list of dependency packages - each entry may either be a relative path to a package on disk or a git repository URL.
- `"macros"` (if `"type": "plugin"`) - A map of all functions that should be exported as macros to the compiler (where the key is the export name and the value is the full path inside the package, e.g. `"derive": "std_derive::derive"`). The signatures of the referenced functions should match [`Fun(List[macro::Token], macro::Compiler) -> List[macro::Token]`](https://github.com/quill-project/macro).