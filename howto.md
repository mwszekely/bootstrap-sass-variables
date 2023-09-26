This file contains instructions for recreating the thousands of _[VARIABLE].scss files whenever Bootstrap updates its variables.

First, take Bootstrap's `_variables.scss` and `variables-dark.scss` files and remove all blank lines, comments, and @include statements (e.g. with the regexes `((((^| )//)|(@include)).*)|(^(//.*)?$\n)` repeatedly for empty lines). Also, fix the various map properties so that they fit on one line (replace `\n((?:\))|(?:\s+.*(?:,|(?:\n\)))))` with `$1` repeatedly), and fix `form-validation-states` manually if necessary. Also replace extra spaces (`\s+` with ` `, `\( ` with `(`, and ` \)` with `)`).

Then, find and replace all apostrophes to escape them for use in the terminal by replacing `'` with `\'` (both `_variables.scss` files only contain apostrophes within quoted strings, so this is safe).

You should now have a list of properties, one property per line.  Transform it into other things (either shell commands or pieces of other files) with the following regexes.  Start by searching for `^\$([a-z0-9\-]+):(?:\s)*(.*)( !default;)` and then:

1. To generate the shell command to create all the files in root, replace with this:

   `echo $'@use "./dependencies/_$1.scss" as *;\n$$$1: $2 !default;' > './_$1.scss' && touch './dependencies/_$1.scss' &&`

   and

   `echo $'@use "./dependencies/_$1.scss" as *;\n$$$1: $2 !default;' > './dark/_$1.scss' && touch './dark/dependencies/_$1.scss' &&` for dark mode
   
   (It obviously can't put the dependencies in the dependency files, it just ensures they exist. You'll need to add whatever contents will satisfy the build errors)


2. To create the list of `@use` rules in the files that use them, replace with these:

   `@use "bootstrap-sass-variables/_$1.scss" as *;` and `@use "bootstrap-sass-variables/dark/_$1.scss" as *;`

3. Replace with this to create the `with` part of the `@forward` rule:

   `$$$1: $$$1 !default,`

4. And for the map file,

   `"$1": $$$1,`


Also yes, there's absolutely a non-hack way to do this that doesn't involve regex'd shell commands, but I have confidence that Bootstrap will get proper module support before enough new versions release that having re-done it "proper" would save time.
