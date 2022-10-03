This file contains instructions for recreating the hundreds of _[VARIABLE].scss files.

First, take Bootstrap's `_variables.scss` file and remove all blank lines, commends, and @include statements (e.g. with the regexes `^(//.*)?$\n` and `(((^| )//)|(@include)).*`). Also, fix the various map properties so that they fit on one line (replace `^((\$.+: \()|(\s+.*))$\n` with `$1` multiple times).

Then, find and replace all apostrophes to escape them for use in the terminal by replacing `'` with `\'` (`_variables.scss` only contains apostrophes within quoted strings, so this is safe).

You should now have a list of properties, one property per line.  Transform it into either shell commands or pieces of other files with the following regexes.  Start by searching for `^\$([a-z0-9\-]+):(?:\s)*(.*)( !default;)` and then:

1. To generate the shell command to create all the files in root, replace with this:

   `echo $'@use "./dependencies/_$1.scss" as *;\n$$$1: $2 !default;' > './_$1.scss' && touch './dependencies/_$1.scss' &&`
   
   (It obviously can't put the dependencies in the dependency files, it just ensures they exist.)


2. To create the list of `@use`/`@forward` rules in the files that use them, replace with these:

   `@use "./$1" as *;`
   
   `@forward "./$1";`

3. Replace with this to create the `with` part of the `@use`/`@forward` rule:

   `$$$1: $$$1 !default,`


Also yes, there's absolutely a non-hack way to do this that doesn't involve regex'd shell commands, but I have confidence that Bootstrap will get proper module support before enough new versions release that having re-done it "proper" would save time.
