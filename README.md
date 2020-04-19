<div align="center">
	<img width="300" height="300" src="https://raw.githubusercontent.com/mcaskill/sass-mq/master/icon.png" alt="Awesome">
</div>

# Media Queries with superpowers [![Build Status](https://api.travis-ci.org/mcaskill/sass-mq.svg?branch=master)](https://travis-ci.org/sass-mq/sass-mq)

**`mq()`+** is a [Sass](http://sass-lang.com/ "Sass - Syntactically Awesome Stylesheets") library that helps you compose media queries in an elegant way.

_Sass MQ was crafted in-house at the Guardian and is improved upon by many contributors._

This fork provides various, often much-requested, enhancements over the original.
_If you already use [@kaelig](https://github.com/kaelig)'s mixin, you know how to use MQ+._

> Note: This fork is always one minor version ahead of the primary package.

## Features

- moved core operations from the _mixin_ to a _function_
- this separation allows media queries to be chained through the `$and` and new `$or` parameters
- added a `$media-feature` parameter to allow media queries to target a different feature (device dimensions, aspect ratios, color, and resolution)

A variant of the mixin using [Jacket](https://github.com/at-import/jacket),
for rasterization, can be found here: [`mj()`](https://gist.github.com/mcaskill/89944b7ef12f37c85c05).

Here is a very basic example:

```scss
$mq-breakpoints: (
    mobile:  320px,
    tablet:  740px,
    desktop: 980px,
    wide:    1300px
);

@import 'mq';

.foo {
    @include mq($from: mobile, $until: tablet) {
        background: red;
    }
    @include mq($from: tablet, $or: mq($until: mobile)) {
        background: green;
    }
    @include mq($from: 600px, $media-feature: height) {
        font-size: 1.25em;
    }
}
```

Compiles to:

```css
@media (min-width: 20em) and (max-width: 46.24em) {
  .foo {
    background: red;
  }
}
@media (min-width: 46.25em), (max-width: 19.99em) {
  .foo {
    background: green;
  }
}
@media (min-height: 37.5em) {
  .foo {
    font-size: 125em;
  }
}
```

_Sass MQ was crafted in-house at the Guardian. Today, many more companies and developers are using it in their projects: [see who uses Sass MQ](#who-uses-sass-mq)._

## How to use it

_The original `mq()` can be played with on [SassMeister](http://sassmeister.com/): `@import 'mq';`._

1. Install:
    - with [Bower](http://bower.io/ "Bower: A package manager for the web"): `bower install mcaskill-sass-mq --save`
    - with [npm](https://www.npmjs.com/): `npm install @mcaskill/sass-mq --save` _supports [eyeglass](https://github.com/sass-eyeglass/eyeglass)_
    - with [yarn](https://www.yarnpkg.com/): `yarn add @mcaskill/sass-mq` _supports [eyeglass](https://github.com/sass-eyeglass/eyeglass)_

    OR [Download _mq.scss](https://raw.github.com/mcaskill/sass-mq/master/_mq.scss) into your Sass project.

2. Import the partial in your Sass files and override default settings
   with your own preferences before the file is imported:
    ```scss
    // To enable support for browsers that do not support @media queries,
    // (IE <= 8, Firefox <= 3, Opera <= 9) set $mq-responsive to false
    // Create a separate stylesheet served exclusively to these browsers,
    // meaning @media queries will be rasterized, relying on the cascade itself
    $mq-responsive: true;

    // Name your breakpoints in a way that creates a ubiquitous language
    // across team members. It will improve communication between
    // stakeholders, designers, developers, and testers.
    $mq-breakpoints: (
        mobile:  320px,
        tablet:  740px,
        desktop: 980px,
        wide:    1300px,

        // Tweakpoints
        desktopAd: 810px,
        mobileLandscape: 480px
    );

    // Define the breakpoint from the $mq-breakpoints list that should
    // be used as the target width when outputting a static stylesheet
    // (when $mq-responsive is set to 'false').
    $mq-static-breakpoint: desktop;

    // If you want to display the currently active breakpoint in the top
    // right corner of your site during development, add the breakpoints
    // to this list, ordered by width. For example: (mobile, tablet, desktop).
    $mq-show-breakpoints: (mobile, mobileLandscape, tablet, desktop, wide);

    // If _mq.scss is in your project:
    @import 'path/to/mq';
    // With eyeglass:
    @import '@mcaskill/sass-mq';
    // With webpack (and boilerplates such as create-react-app)
    @import '~@mcaskill/sass-mq';
    ```
3. Play around with `mq()` (see below)

### Responsive mode ON (default)

`mq()` takes up to three optional parameters:

- `$from`: _inclusive_ `min-*` boundary
- `$until`: _exclusive_ `max-*` boundary
- `$and`: additional custom directives
- `$or`: alternate custom directives
- `$media-feature`: either `width` or `height` of the output device's
  rendering surface

Note that `$until` as a keyword is a hard limit i.e. it's breakpoint - 1.

```scss
.responsive {
    // Apply styling to mobile and upwards
    @include mq($from: mobile) {
        color: red;
    }
    // Apply styling up to devices smaller than tablets (exclude tablets)
    @include mq($until: tablet) {
        color: blue;
    }
    // Same thing, in landscape orientation
    @include mq($until: tablet, $and: '(orientation: landscape)') {
        color: hotpink;
    }
    // Apply styling to tablets up to desktop (exclude desktop)
    @include mq(tablet, desktop) {
        color: green;
    }
}
```

### Responsive mode OFF

To enable support for browsers that do not support `@media` queries,
(IE <= 8, Firefox <= 3, Opera <= 9) set `$mq-responsive: false`.

Tip: create a separate stylesheet served exclusively to these browsers,
for example with conditional comments.

When `@media` queries are rasterized, browsers rely on the cascade
itself. Learn more about this technique on [Jake’s blog](http://jakearchibald.github.io/sass-ie/ "IE-friendly mobile-first CSS with Sass 3.2").

To avoid rasterizing styles intended for displays larger than what those
older browsers typically run on, set `$mq-static-breakpoint` to match
a breakpoint from the `$mq-breakpoints` list. The default is
`desktop`.

The static output will only include `@media` queries that start at or
span this breakpoint and which have no custom `$and` directives:

```scss
$mq-responsive:        false;
$mq-static-breakpoint: desktop;

.static {
    // Queries that span or start at desktop are compiled:
    @include mq($from: mobile) {
        color: lawngreen;
    }
    @include mq(tablet, wide) {
        color: seagreen;
    }
    @include mq($from: desktop) {
        color: forestgreen;
    }

    // But these queries won’t be compiled:
    @include mq($until: tablet) {
        color: indianred;
    }
    @include mq($until: tablet, $and: '(orientation: landscape)') {
        color: crimson;
    }
    @include mq(mobile, desktop) {
        color: firebrick;
    }
}
```

### Verbose and shortand notations

Sometimes you’ll want to be extra verbose (for example, if you’re developing a
library based on top of sass-mq), however for readability in a codebase,
the shorthand notation is recommended.

All of these examples output the exact same thing, and are here for
reference so you can use the notation that best matches your needs:

```scss
// Verbose
@include mq(
    $from: false,
    $until: desktop,
    $and: false,
    $or: false
    $media-feature: width,
    $media-type: $mq-media-type // defaults to 'all'
) {
    .foo {}
}

// Omitting argument names
@include mq(
    false,
    desktop,
    false,
    false,
    width,
    $mq-media-type
) {
    .foo {}
}

// Omitting tailing arguments
@include mq(false, desktop) {
    .foo {}
}

// Recommended
@include mq($until: desktop) {
    .foo {}
}
```

[See the detailed API documentation](http://sass-mq.github.io/sass-mq/#undefined-mixin-mq)

### Adding custom breakpoints

```scss
@include mq-add-breakpoint(tvscreen, 1920px);

.hide-on-tv {
    @include mq(tvscreen) {
        display: none;
    }
}
```

### Changing media type

If you want to specify a media type, for example to output styles
for screens only, set `$mq-media-type`. Alternatively, `mq()` takes an optional `$media-type` parameter for specific types:

#### SCSS

```scss
$mq-media-type: screen;

.screen-only-element {
    @include mq(mobile) {
        width: 300px;
    }

    @include mq($media-type: print) {
        display: none;
    }
}
```

#### CSS output

```css
@media screen and (max-width: 19.99em) {
    .screen-only-element {
        width: 300px;
    }
}

@media print {
    .screen-only-element {
        display: none;
    }
}
```

### Seeing the currently active breakpoint

While developing, it can be nice to always know which breakpoint is
active. To achieve this, set the `$mq-show-breakpoints` variable to
be a list of the breakpoints you want to debug, ordered by width.
The name of the active breakpoint and its pixel and em values will
then be shown in the top right corner of the viewport.

```scss
// Adapt the list to include breakpoint names from your project
$mq-show-breakpoints: (phone, phablet, tablet);
```

![$mq-show-breakpoints](https://raw.githubusercontent.com/sass-mq/sass-mq/master/show-breakpoints.gif)

## Running tests

```sh
npm test
```

## Generating the documentation

Sass MQ is documented using [SassDoc](http://sassdoc.com/).

Generate the documentation locally:

```sh
sassdoc .
```

Generate & deploy the documentation to <https://mcaskill.github.io/sass-mq/>:

```sh
npm run sassdoc
```

## Inspired By…

- <https://github.com/alphagov/govuk_frontend_toolkit/blob/master/stylesheets/_conditionals.scss>
- <https://github.com/bits-sass/helpers-responsive/blob/master/_responsive.scss>
- <https://gist.github.com/magsout/5978325>

## On Mobile-first CSS With Legacy Browser Support

- <http://jakearchibald.github.io/sass-ie/>
- <http://nicolasgallagher.com/mobile-first-css-sass-and-ie/>
- <http://cognition.happycog.com/article/fall-back-to-the-cascade>
- <http://www.theguardian.com/info/developer-blog/2013/oct/14/mobile-first-responsive-ie8>
