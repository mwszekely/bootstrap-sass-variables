(Could this be further automated? Sure, but I have confidence that Bootstrap will get proper module support before enough new versions release that that would save time)


First, take the `_variables.scss` file and remove all blank and
comment/only lines. Also, fix the various map properties so that
they fit on one line.

Then, find and replace all apostrophes: (`_variables.scss` only contains apostrophes within quoted strings)

`'`

`\'`

Finally, use a set of regex commands to transform list list of properties
into either shell commands or pieces of other files:

1. Search `_variables.scss` with the following regex. Then
replace every line with the string at each step, undo, and repeat.

   `^\$([a-z0-9\-]+):(?:\s)*(.*)`

2. Replace with this to create the command to create the files:

   `echo $'@use "./dependencies/_$1.scss" as *;\n$$$1: $2' > './_$1.scss' && touch './dependencies/_$1.scss' &&`
   
   (It obviously can't put the dependencies in the dependency files, 
it just ensures they exist.)

3. Replace with this to create the list of `@use`/`@forward` rules:

   `@use "./$1" as *;`
   
   `@forward "./$1";`

4. Replace with this to create the `with` part of the `@use`/`@forward` rule:

   `$$$1: $$$1 !default,`

5. There's absolutely a non-hack way to do this but, y'know
