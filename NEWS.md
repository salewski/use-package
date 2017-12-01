# Changes

## 2.4

### Breaking changes

- `use-package` no longer requires `diminish` as a dependency, allowing people
  to decide whether they want to use diminish or delight. This means that if
  you do use diminish, you'll now need to pull it into your configuration
  before any use of the `:diminish` kewyord. For example:

  ``` elisp
      (use-package diminish :ensure t)
  ```
  
- Emacs 24.3 or higher is now a requirement.

### Other changes

- Upgrade license to GPL 3.
- New `:hook` keyword.
- New keywords `:custom (foo1 bar1) (foo2 bar2)` etc., and `:custom-face`.
- New `:magic` and `:magic-fallback` keywords.
- New `:defer-install` keyword.
- New customization variable `use-package-enable-imenu-support`.
- Allow `:diminish` to take no arguments.
- Support multiple symbols passed to `:after`, and a mini-DSL using `:all` and
  `:any`.
- `:mode` and `:interpreter` can now accept `(rx ...)` forms.
- Using `:load-path` without also using `:ensure` now implies `:ensure nil`.
- `:bind (:map foo-map ...)` now defers binding in the map until the package
  has been loaded.
- Print key bindings for keymaps in `describe-personal-keybindings`.
- When `use-package-inject-hooks` is non-nil, always fire `:init` and
  `:config` hooks.
- Documentation added for the `:after`, `:defer-install`, `:delight`,
  `:requires`, `:when` and `:unless` keywords.

### Bug fixes

- Repeating a bind no longer causes duplicates in personal-keybindings.
- When byte-compiling, correctly output declare-function directives.
- Append to *use-package* when debugging, don't clear it.
- Don't allow :commands, :bind, etc., to be given an empty list.
- Explicit :defer t should override use-package-always-demand.

## Upgrading to 2.x

### Semantics of :init is now consistent

The meaning of `:init` has been changed: It now *always* happens before
package load, whether `:config` has been deferred or not.  This means that
some uses of `:init` in your configuration may need to be changed to `:config`
(in the non-deferred case).  For the deferred case, the behavior is unchanged
from before.

Also, because `:init` and `:config` now mean "before" and "after", the `:pre-`
and `:post-` keywords are gone, as they should no longer be necessary.

Lastly, an effort has been made to make your Emacs start even in the presence
of use-package configuration failures.  So after this change, be sure to check
your `*Messages*` buffer.  Most likely, you will have several instances where
you are using `:init`, but should be using `:config` (this was the case for me
in a number of places).

### :idle has been removed

I am removing this feature for now because it can result in a nasty
inconsistency.  Consider the following definition:

``` elisp
(use-package vkill
  :commands vkill
  :idle (some-important-configuration-here)
  :bind ("C-x L" . vkill-and-helm-occur)
  :init
  (defun vkill-and-helm-occur ()
    (interactive)
    (vkill)
    (call-interactively #'helm-occur))

  :config
  (setq vkill-show-all-processes t))
```

If I load my Emacs and wait until the idle timer fires, then this is the
sequence of events:

    :init :idle <load> :config

But if I load Emacs and immediately type C-x L without waiting for the idle
timer to fire, this is the sequence of events:

    :init <load> :config :idle

It's possible that the user could use `featurep` in their idle to test for
this case, but that's a subtlety I'd rather avoid.

### :defer now accepts an optional integer argument

`:defer [N]` causes the package to be loaded -- if it has not already been --
after `N` seconds of idle time.

```
(use-package back-button
  :commands (back-button-mode)
  :defer 2
  :init
  (setq back-button-show-toolbar-buttons nil)
  :config
  (back-button-mode 1))
```

### Add :preface, occurring before everything except :disabled

`:preface` can be used to establish function and variable definitions that
will 1) make the byte-compiler happy (it won't complain about functions whose
definitions are unknown because you have them within a guard block), and 2)
allow you to define code that can be used in an `:if` test.

Note that whatever is specified within `:preface` is evaluated both at load
time and at byte-compilation time, in order to ensure that definitions are
seen by both the Lisp evaluator and the byte-compiler, so you should avoid
having any side-effects in your preface, and restrict it merely to symbol
declarations and definitions.

### Add :functions, for declaring functions to the byte-compiler

What `:defines` does for variables, `:functions` does for functions.

### use-package.el is no longer needed at runtime

This means you should put the following at the top of your Emacs, to further
reduce load time:

``` elisp
(eval-when-compile
  (require 'use-package))
(require 'diminish)                ;; if you use :diminish
(require 'bind-key)                ;; if you use any :bind variant
```
