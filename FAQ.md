## FAQ

* [Does ripgrep support configuration files?](#config)
* [What's changed in ripgrep recently?](#changelog)
* [When is the next release?](#release)
* [Does ripgrep have a man page?](#manpage)
* [Does ripgrep have support for shell auto-completion?](#complete)
* [How can I get results in a consistent order?](#order)
* [How do I search over multiple lines?](#multiline)
* [How do I use lookaround and/or backreferences?](#fancy)
* [How do I stop ripgrep from messing up colors when I kill it?](#stop-ripgrep)
* [How do I make the output look like The Silver Searcher's output?](#silver-searcher-output)
* [When I run `rg`, why does it execute some other command?](#rg-other-cmd)
* [How do I create an alias for ripgrep on Windows?](#rg-alias-windows)
* [How do I create a PowerShell profile?](#powershell-profile)
* [How do I pipe non-ASCII content to ripgrep on Windows?](#pipe-non-ascii-windows)


<h3 name="config">
Does ripgrep support configuration files?
</h3>

Yes. Currently, the only way to use a configuration file with ripgrep is to set
the `RIPGREP_CONFIG_PATH` environment variable to your config file's path.
ripgrep will not otherwise automatically read config files from your `HOME`
directory.

The format of the configuration file is an "rc" style and is very
simple. It is defined by two rules:

1. Every line is a shell argument, after trimming ASCII whitespace.
2. Lines starting with `#` (optionally preceded by any amount of
   ASCII whitespace) are ignored.

ripgrep will look for a single configuration file if and only if the
`RIPGREP_CONFIG_PATH` environment variable is set and is non-empty. ripgrep
will parse shell arguments from this file on startup and will behave as if
the arguments in this file were prepended to any explicit arguments given to
ripgrep on the command line.

For example, if your ripgreprc file contained a single line:

```
--smart-case
```

then the following command

```
RIPGREP_CONFIG_PATH=wherever/.ripgreprc rg foo
```

would behave identically to the following command

```
rg --smart-case foo
```

ripgrep also provides a flag, `--no-config`, that when present will suppress
any and all support for configuration. This includes any future support for
auto-loading configuration files (if that ever happens) from pre-determined
paths.

Conflicts between configuration files and explicit arguments are handled
exactly like conflicts in the same command line invocation. That is, this
command:

```
RIPGREP_CONFIG_PATH=wherever/.ripgreprc rg foo --case-sensitive
```

is exactly equivalent to

```
rg --smart-case foo --case-sensitive
```

in which case, the `--case-sensitive` flag would override the `--smart-case`
flag.

The
[question on emulating The Silver Searcher's color settings](#silver-searcher-output)
has another config file example.

If you're confused about what configuration file ripgrep is reading arguments
from, then running ripgrep with the `--debug` flag should help clarify things.
The debug output should note what config file is being loaded and the arugments
that have been read from the configuration.


<h3 name="changelog">
What's changed in ripgrep recently?
</h3>

Please consult ripgrep's [CHANGELOG](CHANGELOG.md).


<h3 name="release">
When is the next release?
</h3>

ripgrep is a project whose contributors are volunteers. A release schedule
adds undue stress to said volunteers. Therefore, releases are made on a best
effort basis and no dates **will ever be given**.

One exception to this is high impact bugs. If a ripgrep release contains a
significant regression, then there will generally be a strong push to get a
patch release out with a fix.


<h3 name="manpage">
Does ripgrep have a man page?
</h3>

Yes! Whenever ripgrep is compiled on a system with `asciidoc` enabled, then a
man page is generated from ripgrep's argv parser. After compiling ripgrep, you
can find the man page like so from the root of the repository:

```
$ find ./target -name rg.1
./target/debug/build/ripgrep-79899d0edd4129ca/out/rg.1
```

Running `man -l ./target/debug/build/ripgrep-79899d0edd4129ca/out/rg.1` will
show the man page in your normal pager.

Note that the man page's documentation for options is equivalent to the output
shown in `rg --help`. To see more condensed documentation (one line per flag),
run `rg -h`.

The man page is also included in all
[ripgrep binary releases](https://github.com/BurntSushi/ripgrep/releases).


<h3 name="complete">
Does ripgrep have support for shell auto-completion?
</h3>

Yes! Shell completions can be found in the
[same directory as the man page](#manpage)
after building ripgrep. Zsh completions are maintained separately and committed
to the repository in `complete/_rg`.

Shell completions are also included in all
[ripgrep binary releases](https://github.com/BurntSushi/ripgrep/releases).

For **bash**, move `rg.bash` to
`$XDG_CONFIG_HOME/bash_completion` or `/etc/bash_completion.d/`.

For **fish**, move `rg.fish` to `$HOME/.config/fish/completions/`.

For **PowerShell**, add `. _rg.ps1` to your PowerShell
[profile](https://technet.microsoft.com/en-us/library/bb613488(v=vs.85).aspx)
(note the leading period). If the `_rg.ps1` file is not on your `PATH`, do
`. /path/to/_rg.ps1` instead.

For **zsh**, move `_rg` to one of your `$fpath` directories.


<h3 name="order">
How can I get results in a consistent order?
</h3>

By default, ripgrep uses parallelism to execute its search because this makes
the search much faster on most modern systems. This in turn means that ripgrep
has a non-deterministic aspect to it, since the interleaving of threads during
the execution of the program is itself non-deterministic. This has the effect
of printing results in a somewhat arbitrary order, and this order can change
from run to run of ripgrep.

The only way to make the order of results consistent is to ask ripgrep to
sort the output. Currently, this will disable all parallelism. (On smaller
repositories, you might not notice much of a performance difference!) You
can achieve this with the `--sort-files` flag.

There is more discussion on this topic here:
https://github.com/BurntSushi/ripgrep/issues/152


<h3 name="multiline">
How do I search over multiple lines?
</h3>

This isn't currently possible. ripgrep is fundamentally a line-oriented search
tool. With that said,
[multiline search is a planned opt-in feature](https://github.com/BurntSushi/ripgrep/issues/176).


<h3 name="fancy">
How do I use lookaround and/or backreferences?
</h3>

This isn't currentl possible. ripgrep uses finite automata to implement regular
expression search, and in turn, guarantees linear time searching on all inputs.
It is difficult to efficiently support lookaround and backreferences in finite
automata engines, so ripgrep does not provide these features.

If a production quality regular expression engine with these features is ever
written in Rust, then it is possible ripgrep will provide it as an opt-in
feature.


<h3 name="stop-ripgrep">
How do I stop ripgrep from messing up colors when I kill it?
</h3>

Type in `color` in cmd.exe (Command Prompt) and `echo -ne "\033[0m"` on
Unix-like systems to restore your original foreground color.

In PowerShell, you can add the following code to your profile which will
restore the original foreground color when `Reset-ForegroundColor` is called.
Including the `Set-Alias` line will allow you to call it with simply `color`.

```powershell
$OrigFgColor = $Host.UI.RawUI.ForegroundColor
function Reset-ForegroundColor {
	$Host.UI.RawUI.ForegroundColor = $OrigFgColor
}
Set-Alias -Name color -Value Reset-ForegroundColor
```

PR [#187](https://github.com/BurntSushi/ripgrep/pull/187) fixed this, and it
was later deprecated in
[#281](https://github.com/BurntSushi/ripgrep/issues/281). A full explanation is
available
[here](https://github.com/BurntSushi/ripgrep/issues/281#issuecomment-269093893).


<h3 name="silver-searcher-output">
How do I make the output look like The Silver Searcher's output?
</h3>

Use the `--colors` flag, like so:

```
rg --colors line:fg:yellow      \
   --colors line:style:bold     \
   --colors path:fg:green       \
   --colors path:style:bold     \
   --colors match:fg:black      \
   --colors match:bg:yellow     \
   --colors match:style:nobold  \
   foo
```

Alternatively, add your color configuration to your ripgrep config file (which
is activated by setting the `RIPGREP_CONFIG_PATH` environment variable to point
to your config file). For example:

```
$ cat $HOME/.config/ripgrep/rc
--colors=line:fg:yellow
--colors=line:style:bold
--colors=path:fg:green
--colors=path:style:bold
--colors=match:fg:black
--colors=match:bg:yellow
--colors=match:style:nobold
$ RIPGREP_CONFIG_PATH=$HOME/.config/ripgrep/rc rg foo
```


<h3 name="rg-other-cmd">
When I run <code>rg</code>, why does it execute some other command?
</h3>

It's likely that you have a shell alias or even another tool called `rg` which
is interfering with ripgrep. Run `which rg` to see what it is.

(Notably, the Rails plug-in for
[Oh My Zsh](https://github.com/robbyrussell/oh-my-zsh/wiki/Plugins#rails) sets
up an `rg` alias for `rails generate`.)

Problems like this can be resolved in one of several ways:

* If you're using the OMZ Rails plug-in, disable it by editing the `plugins`
  array in your zsh configuration.
* Temporarily bypass an existing `rg` alias by calling ripgrep as
  `command rg`, `\rg`, or `'rg'`.
* Temporarily bypass an existing alias or another tool named `rg` by calling
  ripgrep by its full path (e.g., `/usr/bin/rg` or `/usr/local/bin/rg`).
* Permanently disable an existing `rg` alias by adding `unalias rg` to the
  bottom of your shell configuration file (e.g., `.bash_profile` or `.zshrc`).
* Give ripgrep its own alias that doesn't conflict with other tools/aliases by
  adding a line like the following to the bottom of your shell configuration
  file: `alias ripgrep='command rg'`.


<h3 name="rg-alias-windows">
How do I create an alias for ripgrep on Windows?
</h3>

Often you can find a need to make alias for commands you use a lot that set
certain flags. But PowerShell function aliases do not behave like your typical
linux shell alias. You always need to propagate arguments and `stdin` input.
But it cannot be done simply as
`function grep() { $input | rg.exe --hidden $args }`

Use below example as reference to how setup alias in PowerShell.

```powershell
function grep {
    $count = @($input).Count
    $input.Reset()

    if ($count) {
        $input | rg.exe --hidden $args
    }
    else {
        rg.exe --hidden $args
    }
}
```

PowerShell special variables:

* input - is powershell `stdin` object that allows you to access its content.
* args - is array of arguments passed to this function.

This alias checks whether there is `stdin` input and propagates only if there
is some lines. Otherwise empty `$input` will make powershell to trigger `rg` to
search empty `stdin`.


<h3 name="powershell-profile">
How do I create a PowerShell profile?
</h3>

To customize powershell on start-up, there is a special PowerShell script that
has to be created. In order to find its location, type `$profile`.
See
[Microsoft's documentation](https://technet.microsoft.com/en-us/library/bb613488(v=vs.85).aspx)
for more details.

Any PowerShell code in this file gets evaluated at the start of console. This
way you can have own aliases to be created at start.


<h3 name="pipe-non-ascii-windows">
How do I pipe non-ASCII content to ripgrep on Windows?
</h3>

When piping input into native executables in PowerShell, the encoding of the
input is controlled by the `$OutputEncoding` variable. By default, this is set
to US-ASCII, and any characters in the pipeline that don't have encodings in
US-ASCII are converted to `?` (question mark) characters.

To change this setting, set `$OutputEncoding` to a different encoding, as
represented by a .NET encoding object. Some common examples are below. The
value of this variable is reset when PowerShell restarts, so to make this
change take effect every time PowerShell is started add a line setting the
variable into your PowerShell profile.

Example `$OutputEncoding` settings:

* UTF-8 without BOM: `$OutputEncoding = [System.Text.UTF8Encoding]::new()`
* The console's output encoding:
  `$OutputEncoding = [System.Console]::OutputEncoding`

If you continue to have encoding problems, you can also force the encoding
that the console will use for printing to UTF-8 with
`[System.Console]::OutputEncoding = [System.Text.Encoding]::UTF8`. This
will also reset when PowerShell is restarted, so you can add that line
to your profile as well if you want to make the setting permanent.
