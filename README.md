[![Build status](https://badge.buildkite.com/accb37a80d88e0ffda97f55451d05eea2004ed8bbb80a27958.svg)](https://buildkite.com/bazel/rules-sass-postsubmit)

# Sass Rules for Bazel

## Rules
* [sass_binary](#sass_binary)
* [sass_library](#sass_library)
* [multi_sass_binary](#multi_sass_binary)

## Overview
These build rules are used for building [Sass][sass] projects with Bazel.

[sass]: http://www.sass-lang.com

## Setup
To use the Sass rules, add the following to your
`WORKSPACE` file to add the external repositories for Sass, making sure to use the latest
published versions:

```py
http_archive(
    name = "io_bazel_rules_sass",
    # Make sure to check for the latest version when you install
    url = "https://github.com/bazelbuild/rules_sass/archive/1.15.2.zip",
    strip_prefix = "rules_sass-1.15.2",
    sha256 = "96cedd370d8b87759c8b4a94e6e1c3bef7c17762770215c65864d9fba40f07cf",
)

# Fetch required transitive dependencies. This is an optional step because you
# can always fetch the required NodeJS transitive dependency on your own.
load("@io_bazel_rules_sass//:package.bzl", "rules_sass_dependencies")
rules_sass_dependencies()

# Setup repositories which are needed for the Sass rules.
load("@io_bazel_rules_sass//:defs.bzl", "sass_repositories")
sass_repositories()
```

## Basic Example

Suppose you have the following directory structure for a simple Sass project:

```
[workspace]/
    WORKSPACE
    hello_world/
        BUILD
        main.scss
    shared/
        BUILD
        _fonts.scss
        _colors.scss
```

`shared/_fonts.scss`

```scss
$default-font-stack: Cambria, "Hoefler Text", serif;
$modern-font-stack: Constantia, "Lucida Bright", serif;
```

`shared/_colors.scss`

```scss
$example-blue: #0000ff;
$example-red: #ff0000;
```

`shared/BUILD`

```python
package(default_visibility = ["//visibility:public"])

load("@io_bazel_rules_sass//:defs.bzl", "sass_library")

sass_library(
    name = "colors",
    srcs = ["_colors.scss"],
)

sass_library(
    name = "fonts",
    srcs = ["_fonts.scss"],
)
```

`hello_world/main.scss`:

```scss
@import "shared/fonts";
@import "shared/colors";

html {
  body {
    font-family: $default-font-stack;
    h1 {
      font-family: $modern-font-stack;
      color: $example-red;
    }
  }
}
```

`hello_world/BUILD:`

```py
package(default_visibility = ["//visibility:public"])

load("@io_bazel_rules_sass//:defs.bzl", "sass_binary")

sass_binary(
    name = "hello_world",
    src = "main.scss",
    deps = [
         "//shared:colors",
         "//shared:fonts",
    ],
)
```

Build the binary:

```
$ bazel build //hello_world
INFO: Found 1 target...
Target //hello_world:hello_world up-to-date:
  bazel-bin/hello_world/hello_world.css
  bazel-bin/hello_world/hello_world.css.map
INFO: Elapsed time: 1.911s, Critical Path: 0.01s
```

## Build Rule Reference

<a name="reference-sass_binary"></a>
### sass_binary

```py
sass_binary(name, src, deps=[], include_paths=[], output_dir=".", output_name=<src_filename.css>, output_style="compressed", sourcemap=True)
```

`sass_binary` compiles a single CSS output from a single Sass entry-point file. The entry-point file
may have dependencies (`sass_library` rules, see below).


#### Implicit output targets
| Label               | Description                                                               |
|---------------------|---------------------------------------------------------------------------|
| **output_name**     | The generated CSS output                                                  |
| **output_name**.map | The [source map][] that can be used to debug the Sass source in-browser   |

[source map]: https://developers.google.com/web/tools/chrome-devtools/javascript/source-maps


| Attribute       | Description                                                                   |
|-----------------|-------------------------------------------------------------------------------|
| `name`          | Unique name for this rule (required)                                          |
| `src`           | Sass compilation entry-point (required).                                      |
| `deps`          | List of dependencies for the `src`. Each dependency is a `sass_library`       |
| `include_paths` | Additional directories to search when resolving imports                       |
| `output_dir`    | Output directory, relative to this package                                    |
| `output_name`   | Output file name, including .css extension. Defaults to `<src_name>.css`      |
| `output_style`  | [Output style][] for the generated CSS.                                       |
| `sourcemap`     | Whether to generate sourcemaps for the generated CSS. Defaults to True.       |

[Output style]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#output_style

### sass_library

```py
sass_library(name, src, deps=[])
```

Defines a collection of Sass files that can be depended on by a `sass_binary`. Does not generate
any outputs.

| Attribute | Description                                                                         |
|-----------|-------------------------------------------------------------------------------------|
| `name`    | Unique name for this rule (required)                                                |
| `srcs`    | Sass files included in this library. Each file should start with an underscore      |
| `deps`    | Dependencies for the `src`. Each dependency is a `sass_library`                     |

### multi_sass_binary

```py
multi_sass_binary(name, srcs=[], output_style="compressed", sourcemap=True)
```

`multi_sass_binary` compiles a list of Sass files and outputs the corresponding
CSS files and optional sourcemaps. Output is omitted for filenames that start
with underscore "_".


:warning: **WARNING**: This rule does a global compilation, and thus any change in the sources
will trigger a build for **all** files. It is inefficient. Always prefer
`sass_binary` and provide strict dependencies for most efficient compilation.
This rule is also not used internally at Google.


#### Output targets

The following pair of files is generated for _each_ file in `srcs`.

| Label              | Description                                                                  |
|--------------------|------------------------------------------------------------------------------|
| <filename>.css     | The generated CSS output                                                     |
| <filename>.css.map | The [source map][] that can be used to debug the Sass source in-browser      |

[source map]: https://developers.google.com/web/tools/chrome-devtools/javascript/source-maps


| Attribute       | Description                                                                   |
|-----------------|-------------------------------------------------------------------------------|
| `name`          | Unique name for this rule (required)                                          |
| `srcs`          | A list of Sass files (required).                                              |
| `output_style`  | [Output style][] for the generated CSS.                                       |
| `sourcemap`     | Whether to generate sourcemaps for the generated CSS. Defaults to True.       |

[Output style]: http://sass-lang.com/documentation/file.SASS_REFERENCE.html#output_style
