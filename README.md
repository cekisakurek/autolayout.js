![Logo](logo.png)

[![Build Status](https://travis-ci.org/IjzerenHein/autolayout.js.svg?branch=master)](https://travis-ci.org/IjzerenHein/autolayout.js)
[![view on npm](http://img.shields.io/npm/v/autolayout.svg)](https://www.npmjs.org/package/autolayout)

AutoLayout.js implements Apple's [Auto Layout](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html) and [Visual Format Language](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html) in Javascript. Auto layout is a system which lets you perform lay out using mathematical relationships (constraints). It uses the awesome [Cassowary.js](https://github.com/slightlyoff/cassowary.js) library to do the actual constraint resolving and implements Apple's constraint system and Visual Format Language (vfl) on top of that.

[![Example - click me](example.png)](https://rawgit.com/IjzerenHein/visualformat-editor/master/dist/index.html?vfl=example)
*Visual Format Language Example (click image to edit)*


## Index
- [Getting started](#getting-started)
  - [Installation](#installation)
  - [Using the API](#using-the-api)
  - [API Documentation](#api-documentation)
  - [Examples](#examples)
- [Extended Visual Format Language (EVFL)](#extended-visual-format-language-evfl)
- [Additional resources](#additional-resources)
- [Benchmark](https://rawgit.com/IjzerenHein/autolayout.js/master/bench/index.html)
- [Tests](https://rawgit.com/IjzerenHein/autolayout.js/master/test/index.html)
- [ToDo list](#todo-list)
- [Contribute](#contribute)

## Getting started

AutoLayout.js is an abstract library for integrating Auto Layout and VFL into other javascript technologies. It provides a simple API and programming model that you can use to build your own auto layout and VFL solution. A simple example of this is, is using `position: absolute;` to [lay out DOM elements](https://rawgit.com/IjzerenHein/autolayout.js/master/examples/DOM/index.html). A more elobarate example of is the [Visual Format Editor](https://github.com/IjzerenHein/visualformat-editor), which is built using [famo.us](http://famous.org) and [famous-flex](https://github.com/IjzerenHein/famous-flex). AutoLayout.js is written in ES6 and contains transpiled distribution files.

### Installation

    npm install autolayout

AutoLayout.js has a dependency on [Cassowary.js](https://github.com/slightlyoff/cassowary.js). When you are using the `dist/autolayout{.min}.js` file, make sure Cassowary.js is loaded as well.

```html
<head>
  <script type="text/javascript" src="node_modules/autolayout/node_modules/cassowary/bin/c.js"></script>
  <script type="text/javascript" src="node_modules/autolayout/dist/autolayout.js"></script>
</head>
```

```javascript
var AutoLayout = window.AutoLayout;
```

When using a bundler like webpack or browserify, use:

```javascript
var AutoLayout = require('autolayout.js');
```
*(do make sure plugins for transpiling .es6 files are installed!)*

### Using the API

To parse VFL into constraints, use:

```javascript
try {
  // The VFL can be either a string or an array of strings.
  // strings may also contain '\n' which indicates that a new line of VFL will begin.
  var constraints = AutoLayout.VisualFormat.parse([
    '|-[child(==child2)]-[child2]-|',
    'V:|[child(==child2)]|',
  ]);
} catch (err) {
    console.log('parse error: ' + err.toString());
}
```

A View is the main entity onto which constraints are added. It uses the cassowary SimplexSolver to add
relations and variables. You can set the size of the view and other properties such as spacing. When constraints are added it automatically creates so called "sub-views" for every unique name that is encountered in the constraints. The evaluated size and position of these sub-views can be accessed through the `.subViews` property.

```javascript
// Create a view with a set of constraints
var view = new AutoLayout.View({
    constraints: constraints, // initial constraints (optional)
    width: 100,               // initial width (optional)
    height: 200,              // initial height (optional)
    spacing: 10               // spacing size to use (optional, default: 8)
});

// get the size and position of the sub-views
for (var key in view.subViews) {
    console.log(key + ': ' + view.subViews[key]);
    // e.g. {
    //   name: 'child1',
    //   left: 20,
    //   top: 10,
    //   width: 70,
    //   height: 80
    // }
}
```

By changing the size, the layout is re-evaluated and the subView's are updated:

``` javascript
view.setSize(300, 600);

// get the new size & position of the sub-views
for (var key in view.subViews) {
    console.log(key + ': ' + view.subViews[key]);
}
```

Instead of using VFL, you can also add constraints directly.
The properties are identical to those of [NSLayoutConstraint](https://developer.apple.com/library/ios/documentation/AppKit/Reference/NSLayoutConstraint_Class).

```
view.addConstraint({
    view1: 'child3',
    attr1: 'width',    // see AutoLayout.Attribute
    relation: 'equ',   // see AutoLayout.Relation
    view2: 'child4',
    attr2: 'width',    // see AutoLayout.Attribute
    constant: 10,
    multiplier: 1
});
```

### API Documentation

[The API reference documentation can be found here.](docs/AutoLayout.md)


### Examples

- [DOM Example](https://rawgit.com/IjzerenHein/autolayout.js/master/examples/DOM/index.html) [(source)](examples/DOM)
- [Visual Format Editor](https://github.com/IjzerenHein/visualformat-editor)


## Extended Visual Format Language (EVFL)

Apple's Visual Format Language prefers good notation over completeness of expressibility. Because of this some useful constraints cannot be expressed by "Standard" VFL. AutoLayout.js defines an extended syntax (superset of VFL) which you opt-in to use. To enable the extended syntax, set option `extended` to `true` when parsing the visual format:

```javascript
var evfl = '|-[view1(==50%)]';
var constraints = AutoLayout.VisualFormat.parse(evfl, {extended: true});
```

### Language features

- [Proportional size](#proportional-size) (`|-[view1(==50%)]`)
- [Operators](#operators) (`|-[view1(==view2/2-10)]-[view2]-|`)
- [Attributes](#attributes) (`V:|[view2(view1.width)]`)
- [Z-ordering](#z-ordering) (`Z:|-[view1][view2]`)
- [Equal size spacers/centering](#equal-size-spacers-centering)(`|~[center(100)]~|`)
- [Disconnections](#disconnections) (`|[view1(200)]->[view2(100)]|`)
- [View stacks](#view-stacks) (`TODO`)
- [Comments](#comments) (`[view1(view1.height/3)] // enfore aspect ratio 1/3`)

### Proportional size

To make the size proportional to the **size of the parent**, you can use the following % syntax:

    |-[view1(==50%)]    // view1 is 50% the width of the parent (regardless of any spacing)
    [view1(>=50%)]      // view1 should always be more than 50% the width of the parent

### Operators

Operators can be used to create linear equations of the form:
`view1.attr1 <relation> view2.attr2 * multiplier + constant`.

Syntax:

    (view[.{attribute}]['*'|'/'{value}]['+'|'-'{value}])

To for instance, make the width or height proportional to **another view**, use:

    |-[view1(==view2/2)]-[view2]-|  // view1 is half the width of view2
    |-[view1(==view2*4-100)]-[view2]-|  // view1 is four times the width minus 100 of view2

### Attributes

In some cases it is useful to for instance make the **width equal to the height**. To do this you can
use the `.{attribute}` syntax, like this:

    |-[view1]-|
    V:|-[view1(view1.width)]

You can also combine with operators to for instance enforce a certain **aspect ratio**:

    V:|-[view1(view1.width/3)]

Supported attributes:

    .width
    .height
    .left
    .top
    .right
    .bottom
    .centerX
    .centerY

### Z-Ordering

When sub-views overlap it can be useful to specify the z-ordering for the sub-views:

    Z:|[child1][child2]  // child2 is placed in front of child1

By default, all sub-views have a z-index of `0`. When placed in front of each other, the z-index
will be `1` higher than the sub-view it was placed in front of. The z-index of the sub-view can
be accessed through the `zIndex` property:

    console.log('zIndex: ' + view.subViews.child2.zIndex);

### Equal size spacers / centering

Sometimes you just want to center a view. To do this use the `~` connector:

    |~[view1(100)]~|        // view1 has width of 100 and is centered
    V:|~(10%)~[view2]~|     // top & bottom spacers have height of 10%

All `~` connectors inside a single line of EVFL are constrained to have the same size.
You can also use more than 2 connectors to proportionally align views:

    |~[child1(10)]~[child2(20)]~[child3(30)]~|

### Disconnections (right/bottom alignment)

By default, views are interconnected when defined after each other (e.g. `[view1][view2][view3]`). In some cases
it is useful to not interconnect the views, in order to align content to the right or bottom. The following
example shows a disconnection causing the content after the disconnect to align to the right-edge:

```
 // left1..2 are left-aligned, right1..2 are right aligned
      |[left1(100)][left2(300)]->[right1(100)][right2(100)]|
      ^^                       ^^                         ^^
   left1 is                 left2 and                  right2 is
 connected to               right1 are                connected to
  super-view               not connected               super-view
```

### View stacks

TODO

### Comments

Single line comments can be used to explain the VFL or to prevent its execution:

    // Enfore aspect ratios
    [view1(view1.height/3)] // enfore aspect ratio 1/3
    // [view2(view2.height/3)] <-- uncomment to enable

## Additional resources
- [Apple's Auto Layout](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/Introduction/Introduction.html)
- [Visual Format Language](https://developer.apple.com/library/ios/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html)
- [Cassowary.js](https://github.com/slightlyoff/cassowary.js)
- [Overconstrained](http://overconstrained.io)
- [Visual Format Editor](https://github.com/IjzerenHein/visualformat-editor)
- [famous-flex](https://github.com/IjzerenHein/famous-flex)


## ToDo list

AutoLayout.js is currently a work in progress. Once feature complete, this todo list will be removed
and replaced by a roadmap.

**Overal:**
- [X] Toolchain (ES6, external cassowary.js, distributable output, testing, doc generation, travis CI)
- [X] Instructions
- [X] Documentation
- [X] Some examples
- [ ] DOM layouting primitives

**Features:**
- [X] Namespace & classes (AutoLayout, VisualFormat, View, Relation, Attribute)
- [X] Visual format
  - [X] Vfl Parser (thanks to the awesome angular-autolayout team!)
  - [X] Size constraints
  - [X] Greater than, less than relationships.
- [X] Equality relationships
  - [X] Base functionality
  - [X] Multiplier support
- [X] In-equality relationships
- [X] Spacing.
- [X] Priorities.
- [X] Fitting size.
- [X] Intrinsic content size.
- [ ] Checking for ambigous layout.
- [ ] Content hugging?
- [ ] Compression resistance?
- [ ] Remove constraints?
- [ ] Generate visual sub-view output from `View` (ASCII-art'ish)
- [ ] Get constraint definitions from `View`
- [ ] LTR (left to right reading) (Attribute.LEADING & Attribute.TRAILING)
- [ ] Baseline support?
- [ ] Margins? (View & Attributes)

**Extended format features:**
- [X] Percentage support (e.g. |-[child(50%)]-[child2]-])
- [X] Multiplier/divider support (e.g. [child(child2/2)])
- [X] Sub-properties access (e.g. [child(==child.height)])
- [X] Addition/substraction support (e.g. [child(child2-100)])
- [X] Single line comments (// bla)
- [X] Z-ordering (z-index)
- [X] Equal size spacers / centering (e.g. |~[centered(100)~])
- [X] Disonnections
- [ ] View stacks (inspsired by UIStackView) (this make stuff a lot easier)
- [ ] Multi line comments (/* bla */)

**Parked Features:**
- [ ] Variables support (e.g. |-(leftMargin)-[child]]).


## Contribute

If you like this project and want to support it, show some love
and give it a star.

If you want to participate in development, drop me a line or just issue a pull request.
Also have a look at [CONTRIBUTING](./CONTRIBUTING.md).


## Contact
-   @IjzerenHein
-   hrutjes@gmail.com

© 2015 Hein Rutjes
