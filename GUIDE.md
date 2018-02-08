## User Guide

This guide is intended to give an elementary description of ripgrep, and a
brief overview of its capabilities. This guide assumes that ripgrep is
[installed](README.md#installation)
and that readers have passing familiarity with using command line tools. This
also assumes a Unix-like system, although most commands are probably easily
translatable to any command line shell environment.


### Table of Contents

* [Basics](#basics)
* [Recursive search](#recursive-search)
* [Automatic filtering](#automatic-filtering)
* [Manual filtering](#manual-filtering)
* [Replacements](#replacements)
* [Configuration file](#configuration-file)


### Basics

ripgrep is a command line tool that searches your files for patterns that
you give it. ripgrep behaves as if reading each file line by line. If a line
matches the pattern provided to ripgrep, then that line will be printed. If a
line does not match the pattern, then the line is not printed.

The best way to see how this works is with an example. To show an example, we
need something to search. Let's try searching ripgrep's source code. First
grab a ripgrep source archive from
https://github.com/BurntSushi/ripgrep/archive/0.7.1.zip
and extract it:

```
$ curl -LO https://github.com/BurntSushi/ripgrep/archive/0.7.1.zip
$ unzip 0.7.1.zip
$ cd ripgrep-0.7.1
$ ls
benchsuite  grep       tests         Cargo.toml       LICENSE-MIT
ci          ignore     wincolor      CHANGELOG.md     README.md
complete    pkg        appveyor.yml  compile          snapcraft.yaml
doc         src        build.rs      COPYING          UNLICENSE
globset     termcolor  Cargo.lock    HomebrewFormula
```

Let's try our first search by looking for all occurrences of the word `fast`
in `README.md`:

```
$ rg fast README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast while
119:### Is it really faster than everything else?
124:Summarizing, `ripgrep` is fast because:
129:  optimizations to make searching very fast.
```

So what happened here? ripgrep read the contents of `README.md`, and for each
line that contained `fast`, ripgrep printed it to your terminal. ripgrep also
included the line number for each line by default. If your terminal supports
colors, then your output might actually look something like this screenshot:

[![A screenshot of a sample search ripgrep](https://burntsushi.net/stuff/ripgrep-guide-sample.png)](https://burntsushi.net/stuff/ripgrep-guide-sample.png)

In this example, we searched for something called a "literal" string. This
means that our pattern was just some normal text that we asked ripgrep to
find. But ripgrep supports the ability to specify patterns via [regular
expressions](https://en.wikipedia.org/wiki/Regular_expression). As an example,
what if we wanted to find all lines have a word that contains `fast` followed
by some number of other letters?

```
$ rg 'fast\w+' README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
119:### Is it really faster than everything else?
```

In this example, we used the pattern `fast\w+`. This pattern tells ripgrep to
look for any lines containing the letters `fast` followed by *one or more*
word-like characters. Namely, `\w` matches characters that compose words (like
`a` and `L` but unlike `.` and ` `). The `+` after the `\w` means, "match the
previous pattern one or more times." This means that the word `fast` won't
match because there are no word characters following the final `t`. But a word
like `faster` will. `faste` would also match!

Here's a different variation on this same theme:

```
$ rg 'fast\w*' README.md
75:  faster than both. (N.B. It is not, strictly speaking, a "drop-in" replacement
88:  color and full Unicode support. Unlike GNU grep, `ripgrep` stays fast while
119:### Is it really faster than everything else?
124:Summarizing, `ripgrep` is fast because:
129:  optimizations to make searching very fast.
```

In this case, we used `fast\w*` for our pattern instead of `fast\w+`. The `*`
means that it should match *zero* or more times. In this case, ripgrep will
print the same lines as the pattern `fast`, but if your terminal supports
colors, you'll notice that `faster` will be highlighted instead of just the
`fast` prefix.

It is beyond the scope of this guide to provide a full tutorial on regular
expressions, but ripgrep's specific syntax is documented here:
https://docs.rs/regex/0.2.5/regex/#syntax


### Recursive search

In the previous section, we showed how to use ripgrep to search a single file.
In this section, we'll show how to use ripgrep to search an entire directory
of files. In fact, *recursively* searching your current working directory is
the default mode of operation for ripgrep, which means doing this is very
simple.

Using our unzipped archive of ripgrep source code, here's how to find all
function definitions whose name is `write`:

```
$ rg 'fn write\('
src/printer.rs
469:    fn write(&mut self, buf: &[u8]) {

termcolor/src/lib.rs
227:    fn write(&mut self, b: &[u8]) -> io::Result<usize> {
250:    fn write(&mut self, b: &[u8]) -> io::Result<usize> {
428:    fn write(&mut self, b: &[u8]) -> io::Result<usize> { self.wtr.write(b) }
441:    fn write(&mut self, b: &[u8]) -> io::Result<usize> { self.wtr.write(b) }
454:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
511:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
848:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
915:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
949:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
1114:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
1348:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
1353:    fn write(&mut self, buf: &[u8]) -> io::Result<usize> {
```

(**Note:** We escape the `(` here because `(` has special significance inside
regular expressions. You could also use `rg -F 'fn write('` to achieve the
same thing, where `-F` interprets your pattern as a literal string instead of
a regular expression.)

In this example, we didn't specify a file at all. Instead, ripgrep defaulted
to searching your current directory in the absence of a path. In general,
`rg foo` is equivalent to `rg foo ./`.

This particular search showed us results in both the `src` and `termcolor`
directories. The `src` directory is the core ripgrep code where as `termcolor`
is a dependency of ripgrep (and is used by other tools). What if we only wanted
to search core ripgrep code? Well, that's easy, just specify the directory you
want:

```
$ rg 'fn write\(' src
src/printer.rs
469:    fn write(&mut self, buf: &[u8]) {
```

Here, ripgrep limited its search to the `src` directory. Another way of doing
this search would be to `cd` into the `src` directory and simply use `rg 'fn
write\('` again.


### Automatic filtering

After recursive search, ripgrep's most important feature is what it *doesn't*
search. By default, when you search a directory, ripgrep will ignore all of
the following:

1. Files and directories that match the rules in your `.gitignore` glob
   pattern.
2. Hidden files and directories.
3. Binary files. (ripgrep considers any file with a `NUL` byte to be binary.)
4. Symbolic links aren't followed.

All of these things can be toggled using various flags provided by ripgrep:

1. You can disable `.gitignore` handling with the `--no-ignore` flag.
2. Hidden files and directories can be searched with the `--hidden` flag.
3. Binary files can be searched via the `--text` (`-a` for short) flag.
   Be careful with this flag! Binary files may emit control characters to your
   terminal, which might cause strange behavior.
4. ripgrep can follow symlinks with the `--follow` (`-L` for short) flag.

As a special convenience, ripgrep also provides a flag called `--unrestricted`
(`-u` for short). Repeated uses of this flag will cause ripgrep to disable
more and more of its filtering. That is, `-u` will disable `.gitignore`
handling, `-uu` will search hidden files and directories and `-uuu` will search
binary files. This is useful when you're using ripgrep and you aren't sure
whether it's filtering is hiding results from you. Tacking on a couple `-u`
flags is a quick way to find out. (Use the `--debug` flag if you're still
perplexed, and if that doesn't help,
[file an issue](https://github.com/BurntSushi/ripgrep/issues/new).)

ripgrep's `.gitignore` handling actually goes a bit beyond just `.gitignore`
files. ripgrep will also respect repository specific rules found in
`$GIT_DIR/info/exclude`, as well as any global ignore rules in your
`core.excludesFile` (which is usually `$XDG_CONFIG_HOME/git/ignore` on
Unix-like systems).

Sometimes you want to search files that are in your `.gitignore`, so it is
possible to specify additional ignore rules or overrides in a `.ignore`
(application agnostic) or `.rgignore` (ripgrep specific) file.

For example, let's say you have a `.gitignore` file that looks like this:

```
log/
```

This generally means that any `log` directory won't be tracked by `git`.
However, perhaps it contains useful output that you'd like to include in your
searches, but you still don't want to track it in `git`. You can achieve this
by creating a `.ignore` file in the same directory as the `.gitignore` file
with the following contents:

```
!log/
```

ripgrep treats `.ignore` files with higher precedence than `.gitignore` files
(and treats `.rgignore` files with higher precdence than `.ignore` files).
This means ripgrep will see the `!log/` whitelist rule first and search that
directory.

Like `.gitignore`, a `.ignore` file can be placed in any directory. Its rules
will be processed with respect to the directory it resides in, just like
`.gitignore`.

For a more in depth description of how glob patterns in a `.gitignore` file
are interpreted, please see `man gitignore`.


### Manual filtering

In the previous section, we talked about ripgrep's filtering that it does by
default. It is "automatic" because it reacts to your environment. That is, it
uses already existing `.gitignore` files to produce more relevant search
results.

In addition to automatic filtering, ripgrep also provides more manual or ad hoc
filtering. This comes in two varieties: additional glob patterns specified in
your ripgrep commands and file type filtering.

#### Manual filtering: globs

When using ripgrep on the command line, you can give it glob patterns on the
fly to control what is searched.

In our ripgrep source code (see [Basics](#basics) for instructions on how to
get a source archive to search), let's say we wanted to see which things depend
on `clap`, our argument parser.

We could do this:

```
$ rg clap
[lots of results]
```

But this shows us many things, and we're only interested in where we wrote
`clap` as a dependency. Instead, we could limit ourselves to TOML files, which
is how dependencies are communicated to Rust's build tool, Cargo:

```
$ rg clap -g '*.toml'
Cargo.toml
35:clap = "2.26"
51:clap = "2.26"
```

The `-g '*.toml'` syntax says, "make sure every file searched matches this
glob pattern." Note that we put `'*.toml'` in single quotes to prevent our
shell from expanding the `*`.

If we wanted, we could tell ripgrep to search anything *but* `*.toml` files:

```
$ rg clap -g '!*.toml'
[lots of results]
```

This will give you a lot of results again as above, but they won't include
files ending with `.toml`. Note that the use of a `!` here to mean "negation"
is a bit non-standard, but it was chosen to be consistent with how globs in
`.gitignore` files are written. (Although, the meaning is reversed. In
`.gitignore` files, a `!` prefix means whitelist, and on the command line, a
`!` means blacklist.)

Globs are interpreted in exactly the same way as `.gitignore` patterns. That
is, later globs will override earlier globs. For example, the following command
will search only `*.toml` files:

```
$ rg clap -g '!*.toml' -g '*.toml'
```

Interestingly, reversing the order of the globs in this case will match
nothing, since the presence of at least one non-blacklist glob will institute a
requirement that every file searched must match at least one glob. In this
case, the blacklist glob takes precedence over all candidates and prevents any
file from being searched at all!

#### Manual filtering: file types


### Replacements


### Configuration file


## User Guide

The command-line usage of `ripgrep` doesn't differ much from other tools that
perform a similar function, so you probably already know how to use `ripgrep`.
The full details can be found in `rg --help`, but let's go on a whirlwind tour.

`ripgrep` detects when its printing to a terminal, and will automatically
colorize your output and show line numbers, just like The Silver Searcher.
Coloring works on Windows too! Colors can be controlled more granularly with
the `--color` flag.

One last thing before we get started: generally speaking, `ripgrep` assumes the
input it is reading to be UTF-8. However, if ripgrep notices a file is encoded as
UTF-16, then it will know how to search it. For other encodings, you'll need to
explicitly specify them with the `-E/--encoding` flag.

To recursively search the current directory, while respecting all `.gitignore`
files, ignore hidden files and directories and skip binary files:

```
$ rg foobar
```

The above command also respects all `.ignore` files, including in parent
directories. `.ignore` files can be used when `.gitignore` files are
insufficient. In all cases, `.ignore` patterns take precedence over
`.gitignore`.

To ignore all ignore files, use `-u`. To additionally search hidden files
and directories, use `-uu`. To additionally search binary files, use `-uuu`.
(In other words, "search everything, dammit!") In particular, `rg -uuu` is
similar to `grep -a -r`.

```
$ rg -uu foobar  # similar to `grep -r`
$ rg -uuu foobar  # similar to `grep -a -r`
```

(Tip: If your ignore files aren't being adhered to like you expect, run your
search with the `--debug` flag.)

Make the search case insensitive with `-i`, invert the search with `-v` or
show the 2 lines before and after every search result with `-C2`.

Force all matches to be surrounded by word boundaries with `-w`.

Search and replace (find first and last names and swap them):

```
$ rg '([A-Z][a-z]+)\s+([A-Z][a-z]+)' --replace '$2, $1'
```

Named groups are supported:

```
$ rg '(?P<first>[A-Z][a-z]+)\s+(?P<last>[A-Z][a-z]+)' --replace '$last, $first'
```

Up the ante with full Unicode support, by matching any uppercase Unicode letter
followed by any sequence of lowercase Unicode letters (good luck doing this
with other search tools!):

```
$ rg '(\p{Lu}\p{Ll}+)\s+(\p{Lu}\p{Ll}+)' --replace '$2, $1'
```

Search only files matching a particular glob:

```
$ rg foo -g 'README.*'
```

<!--*-->

Or exclude files matching a particular glob:

```
$ rg foo -g '!*.min.js'
```

Search and return paths matching a particular glob (i.e., `-g` flag in ag/ack):

```
$ rg -g 'doc*' --files
```

Search only HTML and CSS files:

```
$ rg -thtml -tcss foobar
```

Search everything except for Javascript files:

```
$ rg -Tjs foobar
```

To see a list of types supported, run `rg --type-list`. To add a new type, use
`--type-add`, which must be accompanied by a pattern for searching (`rg` won't
persist your type settings):

```
$ rg --type-add 'foo:*.{foo,foobar}' -tfoo bar
```

The type `foo` will now match any file ending with the `.foo` or `.foobar`
extensions.
