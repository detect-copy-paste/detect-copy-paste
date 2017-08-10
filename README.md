Detect copy-pasted and structurally similar JavaScript code. Requires Node.js
6.0+, and supports ES6, JSX as well as Flow. Note: the project has been mostly
rewritten for the 0.10 release and saw several breaking changes.

[![Build Status](https://travis-ci.org/detect-copy-paste/detect-copy-paste.svg?branch=master)](https://travis-ci.org/detect-copy-paste/detect-copy-paste)

* [Overview](#overview)
* [Installation](#installation)
* [Usage](#usage)
* [Integration](#integration)
* [Reporters](#reporters)

## Overview

We've all had to deal with code smell, and duplicate code is a common source.
While some instances are easy to spot, this type of searching is the perfect
use-case for a helpful CLI tool.

Existing solutions do exist for this purpose, but some struggle with code
that has wildly varying identifiers or literals, and others have lackluster
support for the JS ecosystem: ES6, JSX, Flow, ignoring module declarations
and imports, etc.

And copy-pasted code is but one type of code duplication. Common boilerplate
and repeated logic can be identified as well using detect-copy-paste, since it
doesn't operate directly on tokens - it uses the ASTs of the parsed code.

You have the freedom to specify a threshold determining the smallest subset of
nodes to analyze. This will identify code with a similar structure, based
on the AST node types, e.g. BlockStatement, VariableDeclaration,
ObjectExpression, etc. By default, it searches nodes with matching identifiers
and literals for copy-paste oriented detection, but this can be disabled.
For context, identifiers include the names of variables, methods, properties,
etc, while literals are strings, numbers, etc.

The tool accepts a list of paths to parse and prints any found matches. Any
directories among the paths are walked recursively, and only `.js` and `.jsx`
files are analyzed. You can explicitly pass file paths that include a different
extension as well. Any `node_modules` and `bower_components` dirs are also
ignored.

![screenshot](https://cloud.githubusercontent.com/assets/817212/24126139/bd151a34-0da2-11e7-94a8-9742279c8566.png)

## Installation

It can be installed via `npm` using:

``` bash
npm install -g detect-copy-paste
```

## Usage

```
Usage: detect-copy-paste [options] <paths ...>


Detect copy-pasted and structurally similar JavaScript code
Example use: detect-copy-paste -I -L -t 20 --ignore "test" ./path/to/src


Options:

  -h, --help                         output usage information
  -V, --version                      output the version number
  -t, --threshold <number>           number of nodes (default: 30)
  -m, --min-instances <number>       min instances for a match (default: 2)
  -c, --config                       path to config file (default: .dcprc)
  -r, --reporter [default|json|pmd]  specify the reporter to use
  -I, --no-identifiers               do not match identifiers
  -L, --no-literals                  do not match literals
  -C, --no-color                     disable colors
  --ignore <pattern>                 ignore paths matching a regex
  --truncate <number>                length to truncate lines (default: 100, off: 0)
  --debug                            print debug information
```

If a `.dcprc` file is located in the project directory, its values will
be used in place of the defaults listed above. For example:

``` javascript
{
  "threshold":     30,
  "identifiers":   true,
  "literals":      true,
  "color":         true,
  "minInstances":  2,
  "ignore":        "test|spec|mock",
  "reporter":      "json",
  "truncate":      100,
}
```

On first use with a project, you may want to run the tool with the following
options, while running explicitly on the lib/src directories, and not the
test/spec dir.

```
detect-copy-paste -t 50 --ignore "test" ./path/to/src
```

From there, feel free to try decreasing the threshold, ignoring identifiers
using the `-I` flag and ignoring literals with `-L`. A lower threshold may lead
you to discover new areas of interest for refactoring or cleanup.

## Integration

It's simple to run detect-copy-paste on your library source as part of a build
process. It will exit with an error code of 0 when no matches are found,
resulting in a passing step, and a positive error code corresponding to its
failure. For example, with Travis CI, you could add the following entries
to your `.travis.yml`:

``` yaml
before_script:
  - "npm install -g detect-copy-paste"

script:
  - "detect-copy-paste ./path/to/src"
```

Note that in the above example, we're using a threshold of 30 for detecting
structurally similar code. A higher threshold may be appropriate as well.

To have detect-copy-paste run with each job, but not block or fail the build, you can
use something like the following:

``` yaml
script:
  - "detect-copy-paste ./path/to/src || true"
```

## Reporters

Aside from the default reporter, both JSON and PMD CPD-style XML reporters are
available. Note that in the JSON example below, indentation and formatting
has been applied. Furthermore, the id property available in these reporters is
useful for parsing by automatic scripts to determine whether or not duplicate
code has changed between builds.

#### JSON

``` json
[{
  "id":"6ceb36d5891732db3835c4954d48d1b90368a475",
  "instances":[
    {
      "path":"spec/fixtures/intersection.js",
      "lines":[1,5],
      "code":"function intersectionA(array1, array2) {\n  array1.filter(function(n) {\n    return array2.indexOf(n) != -1;\n  });\n}"
    },
    {
      "path":"spec/fixtures/intersection.js",
      "lines":[7,11],
      "code":"function intersectionB(arrayA, arrayB) {\n  arrayA.filter(function(n) {\n    return arrayB.indexOf(n) != -1;\n  });\n}"
    }
  ]
}]
```

#### PMD CPD XML

``` xml
<?xml version="1.0" encoding="utf-8"?>
<pmd-cpd>
<duplication lines="10" id="6ceb36d5891732db3835c4954d48d1b90368a475">
<file path="/detect-copy-paste/spec/fixtures/intersection.js" line="1"/>
<file path="/detect-copy-paste/spec/fixtures/intersection.js" line="7"/>
<codefragment>
spec/fixtures/intersection.js:1,5
function intersectionA(array1, array2) {
  array1.filter(function(n) {
    return array2.indexOf(n) != -1;
  });
}

spec/fixtures/intersection.js:7,11
function intersectionB(arrayA, arrayB) {
  arrayA.filter(function(n) {
    return arrayB.indexOf(n) != -1;
  });
}
</codefragment>
</duplication>
</pmd-cpd>
```


## Thanks

Previously known as https://github.com/danielstjules/jsinspect
