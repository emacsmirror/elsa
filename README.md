# <img align="right" src="https://raw.githubusercontent.com/nashamri/elsa-logo/master/elsa-logo-transparent.png" width="133" height="100"> Elsa - Emacs Lisp Static Analyser [![Build Status](https://travis-ci.org/emacs-elsa/Elsa.svg?branch=master)](https://travis-ci.org/emacs-elsa/Elsa)

<p align="center">(Your favourite princess now in Emacs!)</p>

[![Coverage Status](https://coveralls.io/repos/github/Fuco1/Elsa/badge.svg?branch=master)](https://coveralls.io/github/Fuco1/Elsa?branch=master) [![Paypal logo](https://img.shields.io/badge/PayPal-Donate-orange.svg?logo=paypal)](https://www.paypal.com/cgi-bin/webscr?cmd=_s-xclick&hosted_button_id=A5PMGVKCQBT88) [![Patreon](https://img.shields.io/badge/Patreon-Become%20a%20patron-orange.svg?logo=patreon)](https://www.patreon.com/user?u=3282358)

Elsa is a tool that analyses your code without loading or running it.
It can track types and provide helpful hints when things don't match
up before you even try to run the code.

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [State of the project](#state-of-the-project)
- [Non-exhaustive list of features](#non-exhaustive-list-of-features)
    - [Detect dead code](#detect-dead-code)
    - [Enforce style rules](#enforce-style-rules)
    - [Look for suspicious code](#look-for-suspicious-code)
    - [Track types of expressions](#track-types-of-expressions)
    - [Understand functional overloads](#understand-functional-overloads)
- [How do I run it](#how-do-i-run-it)
    - [makem.sh](#makemsh)
    - [Cask](#cask)
    - [Flycheck integration](#flycheck-integration)
- [Configuration](#configuration)
    - [Analysis extension](#analysis-extension)
    - [Rulesets](#rulesets)
- [Type annotations](#type-annotations)
- [How can I contribute to this project](#how-can-i-contribute-to-this-project)
- [F.A.Q.](#faq)
    - [What's up with the logo?](#whats-up-with-the-logo)
- [For developers](#for-developers)
    - [How to write an extension for your-favourite-package](#how-to-write-an-extension-for-your-favourite-package)
    - [How to write a ruleset](#how-to-write-a-ruleset)

<!-- markdown-toc end -->


# State of the project

We are currently in a very early *ALPHA* phase.  API is somewhat
stable but the type system and annotations are under constant
development.  Things might break at any point.

# Non-exhaustive list of features

Here comes a non-exhaustive list of some more interesting features.

The error highlightings in the screenshots are provided by [Elsa
Flycheck extension](https://github.com/emacs-elsa/flycheck-elsa).

Everything you see here actually works, this is not just for show!

## Detect dead code

### Detect suspicious branching logic

![](./images/dead-code-1.png)

![](./images/dead-code-2.png)

### Find unreachable code in short-circuiting forms

![](./images/unreachable-code-1.png)

## Enforce style rules

### Provide helpful tips for making code cleaner

![](./images/useless-code-1.png)

![](./images/useless-code-2.png)

### Add custom rules for your own project with rulesets

![](./images/custom-ruleset-1.png)

### Make formatting consistent

![](./images/formatting-1.png)

## Look for suspicious code

### Find references to free/unbound variables

![](./images/unbound-variable-1.png)

### Don't assign to free variables

![](./images/unbound-variable-2.png)

### Detect conditions which are always true or false

![](./images/always-nil-1.png)

![](./images/always-non-nil-1.png)

### Make sure functions are passed enough arguments

![](./images/number-of-args-1.png)

### Make sure functions are not passed too many arguments

![](./images/number-of-args-2.png)

## Track types of expressions

### Check types of arguments passed to functions for compatibility

![](./images/type-inference-1.png)

![](./images/type-inference-2.png)

![](./images/type-inference-3.png)

## Understand functional overloads

`downcase` can take a string and return a string or take an int and
return an int.  Because we pass a string variable `s`, we can
disambiguate which overload of the function must be used and we can
derive the return type of the function as `string` instead of `(or
string int)`.

![](./images/functional-deps-1.png)

If we pass an input which doesn't match any overload, Elsa will show a
helpful report of what overloads are available and what argument
didn't match.

![](./images/functional-deps-2.png)

# How do I run it

Elsa can be run with [makem.sh](https://github.com/alphapapa/makem.sh)
or with [Cask](https://github.com/cask/cask).

## makem.sh

Using `makem.sh`, simply run this command from the project root
directory, which installs and runs Elsa in a temporary sandbox:

    ./makem.sh --sandbox lint-elsa

To use a non-temporary sandbox directory named `.sandbox` and avoid
installing Elsa on each run:

1.  Initialize the sandbox: `./makem.sh -s.sandbox --install-deps --install-linters`.
2.  Run Elsa: `./makem.sh -s.sandbox lint-elsa`.

See `makem.sh`'s documentation for more information.

## Cask

### [RECOMMENDED] Using packaged version

This method uses [Cask](https://github.com/cask/cask) and installs
Elsa from [MELPA](http://melpa.org/#/).

1. Add `(depends-on "elsa")` to `Cask` file of your project.
2. Run `cask install`.
3. `cask exec elsa FILE-TO-ANALYSE [ANOTHER-FILE...]` to analyse the file.

### Using development version

To use the development version of Elsa, you can clone the repository
and use the `cask link` feature to use the code from the clone.

1. `git clone https://github.com/emacs-elsa/Elsa.git` somewhere to your computer.
2. Add `(depends-on "elsa")` to `Cask` file of your project.
3. Run `cask link elsa <path-to-elsa-repo>`.
4. `cask exec elsa FILE-TO-ANALYSE [ANOTHER-FILE...]` to analyse the file.

## Flycheck integration

If you use [flycheck](https://github.com/flycheck/flycheck) you can
use the [flycheck-elsa](https://github.com/emacs-elsa/flycheck-elsa)
package which integrates Elsa with Flycheck.

# Configuration

By default Elsa core comes with very little built-in logic, only
understanding the elisp [special
forms](https://www.gnu.org/software/emacs/manual/html_node/elisp/Special-Forms.html).

However, we ship a large number of extensions for popular packages
such as `eieio`, `cl`, `dash` or even `elsa` itself.

You can configure Elsa by adding an `Elsafile.el` to your project.
The `Elsafile.el` should be located next to the `Cask` file.

There are multiple ways to extend the capabilities of Elsa.

## Analysis extension

One is by providing special analysis rules for more forms and
functions where we can exploit the knowledge of how the function
behaves to narrow the analysis down more.

For example, we can say that if the input of `not` is `t`, the return
value is always `nil`.  This encodes our domain knowledge in form of
an analysis rule.

All the rules are added in form of extensions.  Elsa has few core
extensions for most common built-in functions such as list
manipulation (`car`, `nth`...), predicates (`stringp`, `atomp`...),
logical functions (`not`, ...) and so on.  These are automatically
loaded because the functions are so common virtually every project is
going to use them.

Additional extensions are provided for popular external packages such
as [dash.el](https://github.com/magnars/dash.el).  To use them, add to
your `Elsafile.el` the `register-extensions` form, like so

``` emacs-lisp
(register-extensions
 dash
 ;; more extensions here
 )
```

## Rulesets

After analysis of the forms is done we have all the type information
and the AST ready to be further processed by various checks and rules.

These can be (non-exhaustive list):

* Stylistic, such as checking that a variable uses `lisp-case` for
  naming instead of `snake_case`.
* Syntactic, such as checking we are not wrapping the else branch of
  `if` with a useless `progn`.
* Semantic, such as checking that the condition of `if` does not
  always evaluate to `non-nil` (in which case the `if` form is
  useless).

Elsa provides some built-in rulesets and more can also be used by loading extensions.

To register a ruleset, add the following form to `Elsafile.el`

``` emacs-lisp
(register-ruleset
 dead-code
 style
 ;; more rulesets here
 )
```

# Type annotations

In Elisp users are not required to provide type annotations to their
code.  While at many places the types can be inferred there are
places, especially in user-defined functions, where we can not guess
the correct type (we can only infer what we see during runtime).

Read the type annotations [documentation](./docs/type-annotations.org)
for more information on how to write your own types.

# How can I contribute to this project

Open an issue if you want to work on something (not necessarily listed
below in the roadmap) so we won't duplicate work.  Or just give us
feedback or helpful tips.

You can provide type definitions for built-in functions by extending
`elsa-typed-builtin.el`.  There is plenty to go.  Some of the types
necessary to express what we want might not exist or be supported yet,
open an issue so we can discuss how to model things.

# F.A.Q.

## What's up with the logo?

See the [discussion](https://github.com/emacs-elsa/Elsa/issues/80).

# For developers

After calling `(require 'elsa-font-lock)` there is a function
`elsa-setup-font-lock` which can be called from `emacs-lisp-mode-hook`
to set up some additional font-locking for Elsa types.

## How to write an extension for your-favourite-package

## How to write a ruleset
