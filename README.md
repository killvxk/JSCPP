# JSCPP

This is a simple C++ interpreter written in JavaScript.

Try it out [on github.io](https://felixhao28.github.io/JSCPP/)!

[![Travis Build Status][build-badge]][build]

## Purpose of the project

As far as I know, every public online C++ excuting environment requires backend servers to compile and run the produced executable. A portable and lightweight interpreter that can be run in browsers can be a fine substitute for those who do not intend to pay for such services.

I also want to make a strict interpreter. The reason being C++ has too many undefined and platform-dependent behaviors and popular C++ compilers tend to be an "over-caring mother" who tries to ignore or even justify the undocumented usages. The abuse of them should be avoided as much as possible IMO. For example, I do not want my students to take it as guaranteed that `sizeof int` produces `4`, because on Arduino Uno, an `int` is a 2-byte value.

Currently, it is mainly for educational uses for a MOOC course I am running (and fun).

## Prerequisites

* NodeJS with -harmony flag or
* A modern browser that [supports generators](https://kangax.github.io/compat-table/es6/#generators) (Firefox, Chrome)

## How to use

Installation

```
npm install JSCPP
```

or (to use lastest cutting-edge version or to contribute)

```
git clone https://github.com/felixhao28/JSCPP.git
cd JSCPP
npm install .
```

### With NodeJS

```js
var JSCPP = require("JSCPP");
var code =    "#include <iostream>"    + "\n"
            + "using namespace std;"   + "\n"
            + "int main() {"           + "\n"
            + "    int a;"             + "\n"
            + "    cin >> a;"          + "\n"
            + "    cout << a << endl;" + "\n"
            + "    return 0;"          + "\n"
            + "}"                      + "\n"
;
var input = "4321";
var exitcode = JSCPP.run(code, input);
console.info("program exited with code " + exitcode);
```

See _demo/example.coffee_ for example.

Use __debugger__

As of 2.0.0, there is a simple but functional real debugger available.

A list of debugger API:

- methods
	+ debugger.next(): one step further
	+ debugger.continue(): continue until breakpoint
	+ debugger.nextNode(): the AST node to be executed
		* sLine
		* sColumn
		* sOffset
		* eLine
		* eColumn
		* eOffset
	+ debugger.nextLine()
	+ debugger.type(typeName)
	+ debugger.variable()
	+ debugger.variable(variableName)
- properties
	+ src: preprocessed source
	+ prevNode: previous AST node
	+ done
	+ conditions
	+ stopConditions
	+ rt: the internal runtime instance
	+ gen: the internal generator

```js
var JSCPP = require("JSCPP")
var mydebugger = JSCPP.run(code, input);
// continue to the next interpreting operation
var done = mydebugger.next();
// if you have an active breakpoint condition, you can just continue
var done = mydebugger.continue();
// by default, debugger pauses at every new line, but you can change it
mydebugger.stopConditions = {
    isStatement: true
    positionChanged: true
    lineChanged: false
};
// so that debugger only stops at a statement of a new position
// or you can add your own condition, i.e. stops at line 10
mydebugger.conditions["line10"] = function (prevNode, nextNode) {
	if (nextNode.sLine=== 10)
		// disable itself so that it only triggers once on line 10
		mydebugger.stopConditions["line10"] = false
		return true;
	else
		return false;
};
// then enable it
mydebugger.stopConditions["line10"] = true
// we need to explicitly use "false" because exit code can be 0
if (done !== false) {
	console.log("program exited with code " + done);
}
// the AST node to be executed next
var s = mydebugger.nextNode();
// sometimes a breakpoint can be set without a statement to be executed next,
// i.e. entering a function call.
while ((s = mydebugger.nextNode()) == null) {
	mydebugger.next();
}
// the content of the statement to be executed next
var nextLine = mydebugger.nextLine();
// it is essentially same as
nextLine = mydebugger.src.slice(s.sOffset, s.eOffset).trim()

console.log("from " + s.sLine + ":" + s.sColumn + "(" + s.sOffset + ")");
console.log("to " + s.eLine + ":" + s.eColumn + "(" + s.eOffset + ")");
console.log("==> " + nextLine);
// examine the internal registry for a type
mydebugger.type("int");
// examine the value of variable "a"
mydebugger.variable("a");
// or list all local variables
mydebugger.variable();
```

A full interactive example is available in *demo/debug.coffee*. Use `node -harmony demo/debug A+B -debug` to debug "A+B" test.

### With a modern browser

There should be a newest version of _JSCPP.js_ in _dist_ ready for you. If not, use `grunt build` to generate one.

Then you can add it to your html. The exported global name for this package is "JSCPP".

```html
<script src="JSCPP.js"></script>
<script type="text/javascript">
	function run(code, input){
		var config = {
			stdio: {
				write: function(s) {
					output.value += s;
				}
			}
		}
		return JSCPP.run(code, input, config);
	}
</script>
```

If you do not provide a customized `write` method for `stdio` configuration, console output will not be correctly shown. See _index.html_ in **gh_pages** branch for an example.

### Run tests

```
grunt test
```

## Q&A

### Which features are implemented?

* (Most) operators
* Primitive types
* Variables
* Arrays
    - ~~Multidimensional array with initializers.~~
* Pointers
* If...else control flow
* Switch...case control flow
    - ~~Declarations inside switch block.~~
* For loop
* While loop
* Do...while loop
* Functions
* Variable scopes
* Preprocessor directives
	- Macro
	- Include

### Which notable features are not implemented yet?

* Goto statements
* Object-oriented features
* Namespaces
* Multiple files support

### How is the performance?

If you want to run C++ programs effciently, compile your C++ code to [LLVM-bitcode](https://en.wikipedia.org/wiki/LLVM) and then use [Emscripten](https://github.com/kripken/emscripten).

### Which libraries are supported?

See current progress in [_includes_](https://github.com/felixhao28/JSCPP/blob/master/includes) folder.

* iostream (only cin and cout and endl)
* cmath
* cctype
* cstring
* cstdio (partial)
* cstdlib (partial)

### Bug report? Feedback?

Post it on [Issues](https://github.com/felixhao28/JSCPP/issues).

## Changelog

* v1.0.0 (2015.3.31)
	- Formal release of this project.
* v1.0.1 (3.31)
	- This release is a mistake.
* v1.0.2 (3.31)
	- New examples.
	- Update README.
	- Renamed runtime and interpreter to start with upper case.
	- (dev-side) Grunt.
* v1.0.3 (4.1)
	- (dev-side) Fix dev-dependency on coffee-script.
	- (dev-side) Grunt watches.
	- (dev-side) Port to coffeescript
	- (dev-side) Refactoring
	- (dev-side) Reworked testing, now all tests are defined in `test.json`
	- Fixed a bug related to a.push(b).concat(c) in syntax parser (#1).
	- Added new tests
* v1.1.0 (4.2)
	- Fixed array initialization with 0 issue
	- Added support for reading string with cin
	- Member function should not be registered globally
	- Added new tests
	- Basic debugging support
* v1.1.1 (4.3)
	- Split debugger from example
	- (dev-side) Grunt only watches newer
	- Fix debug prev command
* v2.0.0 (4.11)
	- New
		+ **Real debugger!**
	- Change
		+ API: Now `JSCPP.run` is all you need
		+ Runtime: The project uses es6, please use V8 with harmony flag
		+ Deprecated: Removed old legacy profiling-replay debugging
		+ Misc: Many other small changes
	- Dev
		+ Major refactoring on interpreter using es6
* dev
	- Fix
		+ Debugger variable scope issue
		+ Readme example
		+ An issue on Chrome Canary

[build-badge]: https://travis-ci.org/felixhao28/JSCPP.svg
[build]: https://travis-ci.org/felixhao28/JSCPP

