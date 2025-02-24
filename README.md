# Clojure support for Sublime Text 4

<div align=center><img src="./screenshots/logo.png" width="350"></div>

This package provides Clojure support for Sublime Text and includes:

- Clojure and EDN syntax grammars (Sublime Text 3+)
- Clojure code formatter/indenter (Sublime Text 4075+)
- Clojure nREPL client (Sublime Text 4075+, nREPL 0.9+)

## Clojure syntax

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/syntaxes.png" width="463" height="362" alt="Syntaxes">

Clojure Sublimed ships with its own syntax definition for Clojure and EDN. Unlike default Clojure syntax, this package is:

- slightly more pedantic as per [EDN spec](https://github.com/edn-format/edn) and [Clojure Reader](https://clojure.org/reference/reader),
- rigorously tested,
- can be used to highlight rainbow parentheses,
- punctuation and validation _inside_ regexps,
- quoted and unquoted regions are marked for highlighting,
- semantically correct tokenization, perfect for fonts with ligatures,
- has separate EDN syntax, same way JSON is separate from JavaScript in Sublime Text.

Want to put your parser to test? Check out [syntax_test_edn.edn](./test_syntax/syntax_test_edn.edn) and [syntax_test_clojure.cljc](./test_syntax/syntax_test_clojure.cljc).

Clojure Sublimed syntax is also used by nREPL client to find form boundaries and namespaces (might be changed in the future).

## Formatter/indenter

Clojure Sublimed includes optional support for [Simple Clojure Formatting rules](https://tonsky.me/blog/clojurefmt/). It doesn’t require nREPL connection but does require `Clojure (Sublimed)` syntax to be selected for buffer.

To reformat whole file, run `Clojure Sublimed: Reindent Buffer`.

To reindent only current line(s), run `Clojure Sublimed: Reindent Lines`.

To enable reindenting / formatting on save, add `format_on_save: true` to settings. ([See how to edit settings](#editing-settings))

To enable correct indentations as you type code, rebind `Enter` to `Clojure Sublimed: Insert Newline`:

```
{"keys":    ["enter"],
 "command": "clojure_sublimed_insert_newline",
 "context": [{"key": "selector", "operator": "equal", "operand": "source.edn | source.clojure"},
             {"key": "auto_complete_visible", "operator": "equal", "operand": false},
             {"key": "panel_has_focus", "operator": "equal", "operand": false}]}
```

Best way to do it is through running `Preferences: Clojure Sublimed Key Bindings`.

## nREPL Client

Clojure Sublimed nREPL client enables interactive development from the comfort of your editor.

Principles:

- Minimal distraction. Display evaluation results inline.
- Decomplected. Eval code and nothing more.
- Server-agnostic. We work with any nREPL socket, local or over network.

Features:

- [x] evaluate code,
- [x] display evaluation results inline.
- [x] display stack traces inline,
- [x] interrupt evaluation,
- [x] eval multiple forms at once (parallel evaluation),
- [x] lookup symbol info,
- [x] show evaluation time,
- [x] bind keys to eval arbitrary code.

We intentionally excluded following features:

- [ ] Autocomplete. Static analysis is much simpler and much more reliable than requiring an always-live connection to the working app.

Look at [Sublime LSP](https://github.com/sublimelsp/LSP) with [Clojure LSP](https://github.com/clojure-lsp/clojure-lsp) or [SublimeLinter](https://github.com/SublimeLinter/SublimeLinter) with [clj-kondo](https://github.com/ToxicFrog/SublimeLinter-contrib-clj-kondo) if you need autocompletion.

Why nREPL and not Socket Server REPL/pREPL/unREPL?

- nREPL has the widest adoption,
- nREPL is machine-friendly,
- nREPL comes with batteries included (interrupt, load-file, sideload),
- nREPL is extensible via middleware,
- nREPL serialization is easier to access from Python than EDN.

Differences from [Tutkain](https://tutkain.flowthing.me/):

- nREPL instead of Socket Server REPL
- Does not have separate REPL panel
- Keeps multiple eval results on a screen simultaneously
- Can show stack traces inline in editor
- Can eval several forms in parallel
- Can eval non well-formed forms (e.g. `(+ 1 2`)
- Can eval infinite sequences
- Redirects all `*out*`/`*err*` to `System.out`/`System.err`

## Installation

1. `Package Control: Install Package` → `Clojure Sublimed`

2. Assign syntax to Clojure files:

   - open any clj/cljc/cljs file,
   - run `View` → `Syntax` → `Open all with current extension as...` → `Clojure Sublimed` → `Clojure (Sublimed)`.

## How to use

Important! Make sure you switched your syntax to `Clojure (Sublimed)`.

1. Run nREPL server.
2. Run `Clojure Sublimed: Connect` command.

### Evaluating code from buffer

From here you have three options:

`Clojure Sublimed: Evaluate` without selection evaluates topmost form around your cursor:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/eval_topmost.png" width="285" height="55" alt="Evaluate Topmost">

`Clojure Sublimed: Evaluate` with selection evaluates selected text:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/eval_selection.png" width="283" height="58" alt="Evaluate Selection">

`Clojure Sublimed: Evaluate Buffer` will evaluate the entire file:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/eval_buffer.png" width="416" height="211" alt="Evaluate Buffer">

You don’t have to wait for one form to finish evaluating to evaluate something else. Multiple things can be executed in parallel:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/eval_parallel.gif" width="353" height="151" alt="Evaluate in Parallel">

By default, Clojure Sublimed will also print evaluation time if it took more than 100 ms:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/eval_elapsed.png" width="500" height="139" alt="Elapsed time">

### Copying evaluation results

Sometimes you want to copy evaluation result. It is recommended to rebind `Cmd+C`/`Ctrl+C` from `copy` to `sublime_clojure_copy`. This will copy evaluation result if inside evaluated region and fallback to default `copy` otherwise.

### Interrupting

If your evaluation runs too long and you want to interrupt it, run `Clojure Sublimed: Interrupt Pending Evaluations`:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/interrupt.png" width="587" height="39" alt="Interrupt">

### Opening stacktrace

If your evaluation failed, put your cursor inside failed region and run `Clojure Sublimed: Toggle Stacktrace`:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/toggle_stacktrace.png" width="553" height="146" alt="Toggle Stacktrace">

Clojure Sublimed will display stacktraces in a Clojure-friendly way. Compare with the default REPL:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/stacktraces.png" width="806" height="390" alt="Stacktraces">

### Looking up symbol

To show symbol info, run `Clojure Sublimed: Toggle Symbol Info`:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/symbol_info.png" width="580" height="172" alt="Toggle Symbol Info">

Universal `Clojure Sublimed: Toggle Info` command acts as either `Toggle Stacktrace` or `Toggle Symbol Info`, depending on context.

### Binding keys to eval code

Every project is different, and sometimes it’s convenient to run a piece of code so often you’d want it on a shortcut. It might be a namespace reload, test execution, database reconnect, linter, formatter — possibilities are endless.

To support such use cases, Clojure Sublimed allows you to bind arbitrary piece of code to a keyboard shortcut. Run `Preferences: Clojure Sublimed Key Bindings` and add something like this:

```
{"keys": ["ctrl+t"],
 "command": "clojure_sublimed_eval_code",
 "args": {"code": "(clojure.test/run-all-tests)"}}
```

Then, whenever you press <key>Ctrl</key> + <key>T</key>, you’ll see the result in the status bar, like this:

<img src="https://raw.github.com/tonsky/Clojure-Sublimed/master/screenshots/eval_code.png" width="536" height="37" alt="Eval Code">

### Clearing results

Finally, to clear evaluation results run `Clojure Sublimed: Clear Evaluation Results`.

### Editing settings

To edit settings, run `Preferences: Clojure Sublimed Settings` command.

### Session-wide settings

It is sometimes desirable to set dynamic Clojure vars for the whole session. To do that, edit `"eval_shared"` setting. For example:

```
"eval_shared": "(do (set! *warn-on-reflection* true) (set! *print-namespace-maps* false))"
```

This will be applied to every evaluation.

## Default Key Bindings

Clojure Sublimed comes with no keybindings enabled by default to guarantee they won’t conflict with any other extension.

This is the recommended keymap:

Command                       | macOS                            | Windows/Linux                                   | Mnemonic
------------------------------|----------------------------------|-------------------------------------------------| -----------------------
Evaluate                      | <kbd>Ctrl</kbd> <kbd>Enter</kbd> | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>Enter</kbd> |
Evaluate Buffer               | <kbd>Ctrl</kbd> <kbd>B</kbd>     | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>B</kbd>     | [B]uffer
Interrupt Pending Evaluations | <kbd>Ctrl</kbd> <kbd>C</kbd>     | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>C</kbd>     | [C]ancel
Toggle Info                   | <kbd>Ctrl</kbd> <kbd>I</kbd>     | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>I</kbd>     | [I]nfo
Clear Evaluation Results      | <kbd>Ctrl</kbd> <kbd>L</kbd>     | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>L</kbd>     | c[L]ear
Copy Evaluation Results       | <kbd>Command</kbd> <kbd>C</kbd>  | <kbd>Ctrl</kbd> <kbd>C</kbd>                    | [C]opy
Reindent Lines                | <kbd>Ctrl</kbd> <kbd>F</kbd> | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>F</kbd> | [F]ormat
Reindent Buffer               | <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>F</kbd> | <kbd>Ctrl</kbd> <kbd>Alt</kbd> <kbd>Shift</kbd> <kbd>F</kbd> | Capital [F]ormat

To set it up, run `Preferences: Clojure Sublimed Key Bindings` command and copy example keybindings to your local Key Bindings file.

## Frequently Asked Questions

Q: REPL/eval doesn’t work

A: Make sure you are using nREPL 0.9 or later.
A: Make sure you have assigned `Clojure (Sublimed)` syntax to the file.

## Credits

Made by [Niki Tonsky](https://twitter.com/nikitonsky).

With contributions by [Jaihindh Reddy](https://github.com/jaihindhreddy).

## See also

[Writer Color Scheme](https://github.com/tonsky/sublime-scheme-writer): A color scheme optimized for long-form writing.

[Alabaster Color Scheme](https://github.com/tonsky/sublime-scheme-alabaster): Minimal color scheme for coding.

[Sublime Profiles](https://github.com/tonsky/sublime-profiles): Profile switcher.

## License

[MIT License](./LICENSE.txt)
