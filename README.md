 
Bootstrap isn't currently designed around the newer system of SASS modules, making it difficult in some ways to use, especially in regards to customizing `_variables.scss` (which is already a little bit tricky to customize with its intricate system of dependencies).

This library makes it much easier to `@use "bootstrap"` with a specific set of customizations, without making it impossible for further customizations down the line (normally, SASS will only allow a `.scss` file to be customized once, which doesn't change here--it's just worked around).

````scss
// _variables-dark.scss

// Make a rudimentary dark theme
// (just setting the body BG and color)

// We need these two variables, so we load them normally.
// Anyone who uses this file can still customize these
// by @use-ing them before this @use-ing this file.
@use "bootstrap-sass-variables/gray-100" as *;
@use "bootstrap-sass-variables/gray-900" as *;

// Adjust these default colors before we actually load Bootstrap
// These can also still be customized via `@use with` on this file.
@forward "bootstrap-sass-variables/body-bg" with ($body-bg: $gray-900 !default);
@forward "bootstrap-sass-variables/body-color" with ($body-color: $gray-100 !default);
````

````scss
// theme-dark.scss

// Configure the variables
// (@use or @forward are both fine here)
@forward "./variables-dark";

// Load a map called $variables containing them all:
@use "bootstrap-sass-variables/variables" as *;

// Load Bootstrap itself, configured to use our custom variables:
@include meta.load-css("node_modules/bootstrap/scss/bootstrap", $with: $variables);

// -----------------------

// Or, maybe customize another variable first before using it:
@use "bootstrap-sass-variables/primary" with ($primary: fuchsia);
@use "./variables-dark" with ($body-bg: black);         // Change from dark gray to black
@use "bootstrap-sass-variables/variables" as *;
@include meta.load-css("node_modules/bootstrap/scss/bootstrap", $with: $variables);
````

----

A full dark theme of the default Bootstrap theme is provided as an example. You can use it like this:

````scss
// your-theme.scss

// $primary isn't modified by the dark theme, so you can change it normally here.
@use "bootstrap-sass-variables/primary" with ($primary: fuchsia);

// Now load the rest of the dark theme variables. 
// You can still customize any of the defaults.
@use "bootstrap-sass-variables/variables-dark" with ($body-bg: black);

// And the rest:
@use "bootstrap-sass-variables/variables" as *;
@include meta.load-css("node_modules/bootstrap/scss/bootstrap", $with: $variables);
````

If you just need a completed example to start with, `theme-light.scss` and `theme-dark.scss` are also provided, which just load Bootstrap with the default variables or dark mode variables, respectively:

````scss
// Load dark mode variables and Bootstrap's CSS
@use "bootstrap-sass-variables/theme-dark";

// --------

// Or customize first:
@use "bootstrap-sass-variables/primary" with ($primary: fuchsia);
@use "bootstrap-sass-variables/variables-dark" with ($body-bg: black);
@use "bootstrap-sass-variables/theme-dark";
````

----
Each variable gets its own file because, given the complex set of dependencies all the Bootstrap variables have on each other, there doesn't appear to be a good way to customize a set of variables more finely than at the level of an entire file beyond maybe the basic colors (any time you group variables together, you need to re-define all their defaults because a file can't be `@use`d twice, for example, and creating a map and then e.g. modifying its `$primary` value wouldn't retroactively update all the buttons backgrounds, say).

Each variable file is created with a small set of regexes used on the original `_variables.scss`. See howto.md for the exact method; it's not fully automated because hopefully Bootstrap gets better module support in version 6?