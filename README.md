# About

Bootstrap isn't (currently) designed around the newer system of Sass modules, making it difficult in some ways to use, especially in regards to customizing `_variables.scss` (which is already a little bit tricky to customize with its intricate system of dependencies).

This library makes it much easier to `@use "bootstrap.scss"` with a specific set of customizations, without making it impossible for users further down the line to continue customizing (normally, SASS will only allow a `.scss` file to be customized once, which doesn't change here&mdash;it's just worked around).

## Step 1. Customize Bootstrap's variables
````scss
// Make a rudimentary dark theme
// by setting $body-bg to $black 
// and $body-color to $white.
// 
// (This is a type of relationship between
// variables that's hard to customize
// in Bootstrap without any redundancy because
// both the variables we want to change
// and those variables' default values
// are in the same _variables.scss file.)

// Get the dependencies we need for $body-bg and $body-color
@use "bootstrap-sass-variables/white" as *;
@use "bootstrap-sass-variables/black" as *;

// Adjust these default colors before we actually load Bootstrap
@forward "bootstrap-sass-variables/body-bg" with ($body-bg: $black !default);
@forward "bootstrap-sass-variables/body-color" with ($body-color: $white !default);

// Note that consumers of this file can still customize 
// $white, $black, $body-bg and $body-color! Those customizations
// just need to be loaded before or with this file.

// (...continued below...)
````
## Step 2. Load Bootstrap itself
````scss
// (...continued from the same file above...)

// Load Bootstrap itself, configured to use our custom variables.
// (All this file does is @forward "bootstrap/scss/bootstrap", but with every single variable defined)
@use "bootstrap-sass-variables/bootstrap";


// NOTE: You can instead do the following, if load-css is more your style:
@use "bootstrap-sass-variables/variable-map" as *;            // Gives us $variables
@include meta.load-css("bootstrap/scss/bootstrap", $with: $variables);
// Just note that you won't get any of the mixins from Bootstrap this way.
````

# Caveats
Watch your `@use` order! Any customization always needs to come **before any other use** of `@use` or `@forward` (don't worry, Sass will let you know if/where/how you get it wrong). Because customizations could themselves `@use` other variables (a lot rely on `$primary` or `$black`, for example), this web can become tangled quickly, and the more variables you put into one file the faster you'll find an unmanageable knot. This is why this library places each variable into its own file (see `howto.md` for their maintenance when Bootstrap adds/changes variables; it's not fully automated because hopefully Bootstrap gets better module support in version 6?).

# Directory/files structure
````
(NAME := the name of any given Bootstrap variable in _variables.scss)

/bootstrap.scss             # @uses Bootstrap with defaults set to customized values
/_<NAME>.scss               # Default variable values
/dependencies/_<NAME>.scss  # Variable dependencies are in separate files for easier maintenance
/dark/_<NAME>.scss          # Default variable values (in dark mode)
/dark/dependencies/...      # Dark mode variables also have their own depenencies
/variables-map.scss         # Provides a Sass map of all variables
````

* `/bootstrap.scss`: `@forward`s the actual Bootstrap Sass file with all your customized variable values, if any. This is generally the final step after you've customized all your variables, or loaded files that do that for you.
* `/_<bootstrap variable name>.scss`: Customize a Bootstrap property by `@use`-ing a file with the same name and customizing that property.  E.G. `@use "prop-name" with ($prop-name: #fff);`. `theme-dark.scss` pre-`@use`s some of these to set them to "dark" values.
* `/dark/_<bootstrap variable name>.scss`: A dark theme version of a given Bootstrap variable (only changes colors and related variables). If you'd like to easily customize the dark theme, you can customize these variables just like the root variables.
* `/dependencies/_<bootstrap variable name>.scss`: *Not intended to be used directly.* `@forward`s the dependencies any given variable has. These exist for maintenance purposes; for new Bootstrap versions the variable files are programatically overwritten but the dependency files are left alone.
* `/variables-map.scss`: Provides a map named `$variables` containing the value of every Bootstrap variable for use in `load-css`. **You would only need to use this if you're manually loading Bootstrap, and also specifically with `load-css` instead of `@use`.**

# FAQ

## No Custom CSS Properties/Custom CSS Variables?

Nope, partially because Bootstrap is already rapidly progressing in this areay, but also because CSS variables don't play well with a lot of Bootstrap's usage of color math mixing functions. By the time `color-mix` is supported by browsers, Bootstrap will probably have this problem solved anyway (or at least, again, better than the currently-required 4 properties per color).

## Why are there so many files, like c'mon now

In Sass, the smallest "unit" of variables (as a collection) is the file. It's not possible to override variables in any grouping smaller than the file that contains it while preserving mixins and such for use by the consumer.

So, just for the basics, it's not possible to load a file twice except under very specific circumstances:
````scss
// Example #1:
@use "bootstrap";
@use "bootstrap" with ($black: white);  // Error, "bootstrap"'s already been loaded

// Example #2:
@use "bootstrap" with ($white: black);
@use "bootstrap" with ($black: white);  // Error, "bootstrap"'s already been loaded

// Example #3:
@use "bootstrap" with ($white: black);
@use "bootstrap";                       // Okay, this one's fine, (though I'm sure the compiler is rolling its eyes at the redundancy)
````
Which, yeah, this absolutely makes sense, because after we've loaded Bootstrap once, it's too late to go back and get a "do-over" with our second set of overrides. The CSS has already been generated, the mixins already captured, etc.

But..., uh, especially in Bootstrap, this is kind of a major problem for variables that rely on the value of previously-defined variables. There's no way to just `@use` a single variable out of a file:
````scss
// Make body-bg a very light Boostrap blue:
@use "bootstrap" as BS;
@use "bootstrap" with ($body-bg: BS.$blue-900);    // Well dang
````

We could work around this by splitting some variables off into separate files:
````scss
// Pretend Bootstrap silos off all color variables to `_colors.scss`
// and leaves everything else in `_variables.scss`.
@use "bootstrap/colors" as BS with ($blue: #0000ff);
@use "bootstrap" with ($body-bg: BS.$blue-900);     // Works OK now!
````

But ultimately the smallest "unit" of variables is the file. Any grouping of variables we choose to put into one file is arbitrary and will cause issues for someone eventually:
````scss
// Again, pretend "bootstrap/colors" actually exists as a variable file
// and change purple to be dark blue
@use "bootstrap/colors" as BS with ($blue: #0000ff);
@use "bootstrap/colors" with ($purple: BS.$blue-300);   // Not gonna work...
@use "bootstrap";
````

From this example, we could further split `colors.scss` into `blues.scss`, `purples.scss` etc, but `$blue-500` depends on `$blue`, and what if we need to change *them* independently too?  So we need to split `blues.scss` into `blue-primary.scss` and `blues-secondary.scss`, but what if someone needs `blue-900` to depend on `blue-500`? And we just keep going deeper and deeper until we hit bedrock.


Sass does, thankfully, come with an escape hatch to allow variables to be customized more finely: `load-css` allows you to pass in an arbitrary map of overrides to apply, and this map can be created, re-ordered, and mutated in basically any way we want. Unfortunately, `load-css` *does not* expose the mixins, functions or variables used, and Bootstrap's mixins in particular are vitally important when creating or modifying Bootstrap themes. If we don't preserve the availability of mixins expected when just `@use`-ing Bootstrap, then it's not much of a solution, at least here for our purposes.


Ultimately, when making a big ol' pile of variables to customize, it seems there are 3 options:

1. Make 1 big `_variables.scss` file, and include all your overrides at once. Customizing one variable will often mean customizing, like, 50% of them as implicit dependencies. `load-css` can mitigate this, but you sacrifice your ability to use the mixins customized by your overrides. You'll only have the variables, and only the ones *you* defined.
2. You can arbitrarily group alike variables together (e.g. put all the basic colors in `primary-colors.scss`), which kicks the can down the road a fair bit if you organize well. Eventually you'll still run into the problem in option #1 on a smaller scale, though, and where you do draw those lines is really difficult to find a one-size-fits-all solution unless you:
3. Take option #2 to its logical conclusion and every variable just goes in its own file with no grouping. While errors can still occur if the import order is off, these errors can be fixed by simply re-ordering some statements around. Critically, you can *always have any combination of dependencies* that you like without ever needing to redundantly redefine anything and without worring about "impossible combinations".
