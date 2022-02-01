## About

Bootstrap isn't currently designed around the newer system of Sass modules, making it difficult in some ways to use, especially in regards to customizing `_variables.scss` (which is already a little bit tricky to customize with its intricate system of dependencies).

This library makes it much easier to `@use "bootstrap"` with a specific set of customizations, without making it impossible for users further down the line to continue customizing (normally, SASS will only allow a `.scss` file to be customized once, which doesn't change here&mdash;it's just worked around).

### Step 1. Customize Bootstrap's variables
````scss
// Make a rudimentary dark theme
// (just setting the body BG and color)

// We reference the $black and $white variables,
// so we need to @use them here.
// If anyone who uses this file needs to customize
// $black and $white, they can still do that by
// using these files with those customizations
// before using this file.
@use "bootstrap-sass-variables/white" as *;
@use "bootstrap-sass-variables/black" as *;

// Adjust these default colors before we actually load Bootstrap
// These can also still be customized via `@use with` on this file.
@forward "bootstrap-sass-variables/body-bg" with ($body-bg: $black !default);
@forward "bootstrap-sass-variables/body-color" with ($body-color: $white !default);

// (...continued below...)
````
### Step 2. Load Bootstrap itself
````scss
// (...continued from the same file above...)

// Load Bootstrap itself, configured to use our custom variables.
// Pick ONE of the two;
// 1. the former loads the CSS and makes all mixins and other things available, and
// 2. the latter just loads the CSS, but lets you specify your own Bootstrap file.

// Option 1:
@forward "bootstrap-sass-variables/bootstrap";

// OR Option 2:
@use "bootstrap-sass-variables/variables" as *; // Gives us $variables
@include meta.load-css("node_modules/bootstrap/scss/bootstrap", $with: $variables);
````

----
## Default themes
A full dark theme of the default Bootstrap theme is provided as an example. You can use it like this:

````scss
// your-theme.scss

// $primary isn't modified by the dark theme, so you can change it normally here.
@forward "bootstrap-sass-variables/primary" with ($primary: fuchsia);

// Now load the rest of the dark theme variables. 
// $body-bg is modified by the dark theme, but it can
// still be further customized to be anything here:
@forward "bootstrap-sass-variables/theme-dark" with ($body-bg: yellow);
````

Or, for a little more control,
````scss
// your-theme.scss

// Modify variables used by variables-dark 
// (but not *modified* by variables-dark) here.
@forward "boostrap-sass-variables/black" with ($black: #010101);

// Actually modify the variables modified by variables-dark
// (This is where you can *modify* variables modified by variables-dark)
@forward "bootstrap-sass-variables/variables-dark";

// Modify all other variables that aren't related
// to the dark theme whatsoever here.
// (You can also put some of them above, but if they
// end up using some variable that the dark theme
// customizes, you'll get an error from Sass,
// because dependencies are fun)
@forward "bootstrap-sass-variables/enable-cssgrid" with ($enable-cssgrid: true);

// Actually load Bootstrap
@forward "bootstrap-sass-variables/bootstrap";

````

## Caveats
Watch your `@use` order! Any customization always needs to come **before any other use** of `@use` or `@forward` (don't worry, Sass will let you know if/where/how you get it wrong). Because customizations could themselves `@use` other variables (a lot rely on `$primary` or `$black`, for example), this web can become tangled quickly, and the more variables you put into one file the faster you'll find an unmanageable knot. This is why each variable is placed into its own file (see howto.md for their maintenance when Bootstrap adds/changes variables; it's not fully automated because hopefully Bootstrap gets better module support in version 6?).

## Files structure
````
(NAME := the name of any given Bootstrap variable in _variables.scss)

/dark/_<NAME>.scss
/dependencies/_<NAME>.scss
/_<NAME>.scss
/_variables.scss
/_variables-dark.scss
/bootstrap.scss
/theme-dark.scss
/theme-light.scss
````

* `theme-(light-dark).scss`: Loads Bootstrap with all CSS, mixins, and functions, using any variables you've already customized. The final step.
* `bootstrap.scss`: `@forward`s the actual Bootstrap Sass file with all your customized variable values, if any. Used by `theme-(light|dark).scss`. Use if you're making your own theme as the final step.
* `_variables.scss`: Provides a map named `$variables` containing the value of every Bootstrap variable. After `@use`ing this, you can no longer customize any variables. Used by `bootstrap.scss`. You would only need to use this if you're manually loading Bootstrap, and also specifically with `load-css` instead of `@use`.
* `_variables-dark.scss`: `@use`s all dark theme versions of the default Bootstrap variables, replacing some of the defaults that `_variables.scss` uses. Used by `theme-dark.scss`. Use this if you want to create a new theme based off of a dark Bootstrap theme.
* `_<boostrap variable name>.scss`: Customize a Bootstrap variable with these files.  E.G. `@use "var-name" with ($var: value);`. Used by `_variables.scss`. Use these to customize an individual Bootstrap variable.
* `dark/_<boostrap variable name>.scss`: A dark theme version of a given Bootstrap variable (only colors and related variables). Use these to customize the variables that `variables-dark.scss` and `theme-dark.scss` use.
* `/dependencies/_<boostrap variable name>.scss`: *Not intended to be used directly.* `@forward`s the dependencies any given variable has. These exist for maintenance purposes; for new Bootstrap versions the variable files are programatically overwritten but the dependency files are left alone.