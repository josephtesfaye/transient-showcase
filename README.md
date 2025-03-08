# Transient Showcase

<!-- !!!THIS FILE HAS BEEN GENERATED!!! Edit transient-showcase.org -->

Code examples for interactive explanations of [transient](https://github.com/magit/transient).

This guide assumes you have minimal knowledge of Emacs, some programming experience in elisp and non-lisp languages, and have at least seen [screenshots](https://magit.vc/screenshots/) of `magit`.

These examples serve to **illustrate** the documentation in transient's manual, which probably ships with your Emacs, accessible via `M-x info-display-manual transient`.

## How to use

If you are reading the README.md on Github, this markdown file is exported as a preview for transient-showcase.org and to appear in search results etc.

There are two better ways:

-   Open the transient-showcase.org file in Emacs and run examples as literate org.
-   Install the package to run commands and read their source. Start with the `tsc-showcase` command.


### Using as literate org document

If you open the org file in Emacs, it will switch to Org mode and you can run individual source blocks with `org-babel-execute-src-blk` on the block.


### Using as an installable package

If you install the package, you can read source for each example with the normal `describe-command`. All commands are under `tsc-*` prefix. Somewhat useful suffixes are under `tsc-suffix-*` while less useful ones under `tsc--suffix-*`. They will come in handy when you are developing new applications.

Installations for straight and elpaca:

```elisp

;; using package-vc (built-in)
(package-vc-install
 '(transient-showcase
   :url "https://github.com/positron-solutions/transient-showcase.git"))
(require 'transient-showcase) ; now you can access the commands

;; using elpaca (recommended to add a rev for reproducibility)
(use-package transient-showcase
  :ensure (transient-showcase
           :host github
           :repo "positron-solutions/transient-showcase"))

;; using straight use-package with custom recipe
(use-package transient-showcase
  :straight '(transient-showcase
              :type git :host github
              :repo "positron-solutions/transient-showcase"))
```

> [!TIP]
>  While the exported markdown version of this file is also the README for this repository, it's not intended to be used directly or by copy-pasting.  Many links will only open in Emacs.


### Running Examples in Org Mode

This is a basic transient, using an anonymous lambda interactive command as its only suffix.

```elisp

(transient-define-prefix tsc-hello ()
  "Prefix that is minimal and uses an anonymous command suffix."
  [("s" "call suffix"
    (lambda ()
      (interactive)
      (message "Called a suffix")))])

;; First, use M-x org-babel-execute-src-blk to cause `tsc-hello' to be
;; defined
;; Second, M-x `eval-last-sexp' with your point at the end of the line below
;; (tsc-hello)

```

After executing the block above, you can `execute-extended-command` (**M-x**) and select `tsc-hello` to show this transient. All transient prefixes are also commands that show up in (**M-x**)

If the example above is hard to read, review some [elisp](info:elisp#Top) [syntax](#orgaba08e8) and typical forms.


# Table of Contents

 - [Terminology](#Terminology)
  - [Prefixes and Suffixes](#Prefixes-and-Suffixes)
  - [Nesting Prefixes](#Nesting-Prefixes)
  - [Infix](#Infix)
  - [Summary](#Summary)
- [Declaring - Equivalent Forms](#Declaring---Equivalent-Forms)
  - [The Shorthand Forms](#The-Shorthand-Forms)
  - [Keyword Arguments](#Keyword-Arguments)
  - [Macro Child Definition Style](#Macro-Child-Definition-Style)
  - [Overriding slots in the prefix definition](#Overriding-slots-in-the-prefix-definition)
  - [Quoting Note for Vectors](#Quoting-Note-for-Vectors)
- [Groups & Layouts](#Groups-&-Layouts)
  - [Descriptions](#Descriptions)
  - [Layouts](#Layouts)
  - [Manually setting group class](#Manually-setting-group-class)
  - [Pad Keys](#Pad-Keys)
- [Nesting & Flow Control](#Nesting-&-Flow-Control)
  - [Single vs Repeat Commands](#Single-vs-Repeat-Commands)
  - [Nesting](#Nesting)
  - [Return Without Setup](#Return-Without-Setup)
  - [Pre-Commands Explained](#Pre-Commands-Explained)
  - [Combining With Interactive](#Combining-With-Interactive)
- [Using & Managing State](#Using-&-Managing-State)
  - [Ad-Hoc Elisp State](#Ad-Hoc-Elisp-State)
  - [Transient State & Persistence](#Transient-State-&-Persistence)
- [Input Sentence Construction](#Input-Sentence-Construction)
- [Controlling Visibility](#Controlling-Visibility)
  - [Visibility Predicates](#Visibility-Predicates)
  - [Inapt (Temporarily Inappropriate)](#Inapt-(Temporarily-Inappropriate))
  - [Levels](#Levels)
- [Advanced](#Advanced)
  - [Dynamically generating layouts](#Dynamically-generating-layouts)
  - [Modifying layouts](#Modifying-layouts)
  - [Using prefix scope in children](#Using-prefix-scope-in-children)
  - [Custom Infix Types](#Custom-Infix-Types)
- [Appendixes](#Appendixes)
  - [EIEIO - OOP in Elisp](#EIEIO---OOP-in-Elisp)
  - [Debugging](#Debugging)
  - [Layout Hacking](#Layout-Hacking)
  - [Hooks](#Hooks)
  - [Preludes](#Preludes)
  - [Essential Elisp](#Essential-Elisp)
- [Further Reading](#Further-Reading)

<a id="org8b25579"></a>

# Terminology

Transient means temporary. Transient gets its name from the temporary keymap and the popup UI for displaying that keymap. Emacs has a similar idea built-in with [set-transient-map](elisp:(describe-function 'set-transient-map)) for a temporary high-precedence keymap.


<a id="org5cf0073"></a>

## Prefixes and Suffixes

The hello transient user input sequence is:

`Prefix -> Suffix`

-   The **prefix** is the command you invoke first, such as `magit-dispatch`
-   A **suffix** is a command displayed in the transient UI, such as `magit-stage`

```elisp
(magit-dispatch) ; same as pressing 'h' in magit-status buffer
```

The keymap and UI display is frequently referred to as "a transient". "Prefix" and "a transient" are almost the same thing. Invoking a prefix will show a transient. They are inseparable ideas.


### Conceptual similarity to Emacs prefix arguments

**Setting [prefix arguments](https://emacsdocs.org/docs/emacs/Prefix-Keymaps) with `universal-argument` (`C-u`) is a distinct, separate behavior that is part of Emacs.**

With prefix arguments, you "call" commands with extra arguments, like you would a function.

A transient prefix can set some states and its suffix can then use these states to tweak its behavior. The difference is that within the lifecycle of a transient UI, and coordinating with transient's state persistence, you can create much more complex input to your commands. You can use commands to construct phrases for other commands.

To see a short example of prefix arguments being used within a transient prefix, see [the scope example](#orgcdc6bd0).


<a id="org37213e4"></a>

## Nesting Prefixes

A prefix can also be bound as a suffix, enabling *nested* prefixes. A user input sequence with nested transients might look like:

`Prefix -> Sub-Prefix -> Sub-Prefix -> Suffix`

For example, in the `magit-dispatch` transient (`?`), `l` for `magit-log` is a nested transient. `b` for `all branches` is the suffix command `magit-log-all-branches`.

See [Flow Control](#org77d8b3f) for nested transient examples with both sub-prefixes and suffixes that do no exit.


<a id="org271977f"></a>

## Infix

Some suffixes need to hold state, toggling or storing an argument. Infixes are specialized suffixes to set and hold state. A user input sequence with infixes:

`Prefix -> Infix -> Infix -> Suffix`

See [Infix examples](#orgc42c62d) to get a better idea.


<a id="orge66e88a"></a>

## Summary

-   **Prefixes** display the pop-up UI and bind the keymap.
-   **Suffixes** are commands bound within a prefix
-   **Infixes** are a specialized suffix for storing and setting state
-   A **Suffix** may be yet another **Prefix**, in which case the transient is nested


<a id="org0feaede"></a>

# Declaring - Equivalent Forms

You can declare the same behavior 3-4 ways

-   Shorthand forms within `transient-define-prefix` macro allow shorthand binding of suffixes & commands or creation of infixes directly within the layout definition.

-   Macros for suffixes and infix definition streamline defining commands while also defining how they will behave in a layout.

-   Keyword arguments `(:foo val1 :bar val2)` are interpreted by the macros and used to set slots (OOP attributes) on prefix, group, and suffix objects. Similar forms for declaring suffixes can be used to modify them when declaring a layout. Very specific control over layouts also uses these forms.

```elisp
;; slots & methods that can be set / overridden in children
(describe-symbol transient-child)
```

-   Custom classes using EIEIO (basically elisp OOP) can change methods deeper in the implementation than you can reach with slots. `describe-symbol` is a quick way to look at the methods.

```elisp
;; slots & methods that can be set / overridden in suffixes
(describe-symbol transient-suffix)
```

See the [EIEIO Appendix](#org6901288) for introduction to exploring EIEIO objects and classes.


<a id="orge8d3997"></a>

## The Shorthand Forms

Binding suffixes with the `("key" "description" suffix-or-command)` form within a group is extremely common. The `transient-define-prefix` macro evaluates this into a suffix.

```elisp

(transient-define-prefix tsc-wave ()
  "Prefix that waves at the user"
  [("w" "wave" tsc-suffix-wave)
   ("e" "eval-expression" eval-expression)])
;; tsc-suffix-wave is a simple command from wave-prelude

;; (tsc-wave)

```

Both commands and suffixes declared with the `transient-define-suffix` macro can be used. It's a good reason to use `private--namespace` style names for suffix since you don't usually want to call them directly.


<a id="org930f715"></a>

## Keyword Arguments

You can customize the slot value (OOP attribute implemented with EIEIO) of the transient, groups, and suffixes by adding extra `:foo value` style pairs.

Not all behaviors have a shorthand form, so as you use more behaviors, you will see more of the keyword argument style API. Here we use the `:transient` property, set to true, meaning the suffix won't exit the transient.

```elisp

(transient-define-prefix tsc-wave-keyword-args ()
  "Prefix that waves at the user persistently."
  [("e" "wave eventually & stay" tsc--wave-eventually :transient t)
   ("s" "wave surely & leave" tsc--wave-surely :transient nil)])

;; (tsc-wave-keyword-args)

```

Launch the command, wave several times (note timestamp update) and then exit with (**C-g**).


<a id="orgf907c9d"></a>

## Macro Child Definition Style

The `transient-define-suffix` macro can help if you need to bind a command in multiple places and only override some properties for some prefixes. It makes the prefix definition more compact at the expense of a more verbose command.

```elisp

(transient-define-suffix tsc-suffix-wave-macroed ()
  "Prefix that waves with macro-defined suffix."
  :transient t
  :key "T"
  :description "wave from macro definition"
  (interactive)
  (message "Waves from a macro definition at: %s" (current-time-string)))

;; Suffix definition creates a command
;; (tsc-suffix-wave-macroed)
;; Because that's where the suffix object is stored
;; (get 'tsc-suffix-wave-macroed 'transient--suffix)

```

```elisp

;; tsc-suffix-wave-suffix defined above
(transient-define-prefix tsc-wave-macro-defined ()
  "Prefix to wave using a macro-defined suffix."
  [(tsc-suffix-wave-macroed)])
;; note, information moved from prefix to the suffix.

;; (tsc-wave-macro-defined)

```


<a id="orgf9a6fa7"></a>

## Overriding slots in the prefix definition

Even if you define a property via one of the macros, you can still override that property in the later prefix definition. The example below overrides the `:transient`, `:description`, and `:key` properties of the `tsc-suffix-wave` suffix defined above:

```elisp

(defun tsc--wave-override ()
  "Vanilla command used to override suffix's commands."
  (interactive)
  (message "This suffix was overridden.  I am what remains."))

(transient-define-prefix tsc-wave-overridden ()
  "Prefix that waves with overridden suffix behavior."
  [(tsc-suffix-wave-macroed
    :transient nil
    :key "O"
    :description "wave overridingly"
    :command tsc--wave-override)]) ; we overrode what the suffix even does

;; (tsc-wave-overridden)

```

If you just list the key and symbol followed by properties, it is also a supported shorthand suffix form:

`("wf" tsc-suffix-wave :description "wave furiously")`


<a id="org5a5a86f"></a>

## Quoting Note for Vectors

Inside the `[ ...vectors... ]` in `transient-define-prefix`, you don't need to quote symbols because in the vector, everything is a literal. When you move a shorthand style `:property symbol` out to the `transient-define-suffix` form, which is a list, you might need to quote the symbol as `:property 'symbol`.


<a id="org59c9fc4"></a>

# Groups & Layouts

To define a transient, you need at least one group. Groups are vectors, delimited as `[ ...group... ]`.

There is basic layout support and you can use it to collect or differentiate commands.

If you begin a group vector with a string, you get a group heading. Groups also support some [properties](https://magit.vc/manual/transient/Group-Specifications.html#Group-Specifications). The [group class](elisp:(describe-symbol transient-group)) also has a lot of information.


<a id="org3718be7"></a>

## Descriptions

Very straightforward. Just make the first element in the vector a string or add a `:description` property, which can be a function.

In the prefix definition of suffixes, the second string is a description.

The `:description` key is applied last and therefore wins in ambiguous declarations.

```elisp

(transient-define-prefix tsc-layout-descriptions ()
  "Prefix with descriptions specified with slots."
  ["Let's Give This Transient a Title\n" ; yes the newline works
   ["Group One"
    ("wo" "wave once" tsc-suffix-wave)
    ("wa" "wave again" tsc-suffix-wave)]

   ["Group Two"
    ("ws" "wave some" tsc-suffix-wave)
    ("wb" "wave better" tsc-suffix-wave)]]

  ["Bad title" :description "Group of Groups"
   ["Group Three"
    ("k" "bad desc" tsc-suffix-wave :description "key-value wins")
    ("n" tsc-suffix-wave :description "no desc necessary")]
   [:description "Key Only Def"
                 ("wt" "wave too much" tsc-suffix-wave)
                 ("we" "wave excessively" tsc-suffix-wave)]])

;; (tsc-layout-descriptions)

```


### Dynamic Descriptions

**Note:** The property list style for dynamic descriptions is the same for both prefixes and suffixes. Add `:description symbol-or-lambda-form` to the group vector or suffix list.

```elisp

(transient-define-prefix tsc-layout-dynamic-descriptions ()
  "Prefix that generate descriptions dynamically when transient is shown."
  ;; group using function-name to generate description
  [:description current-time-string
                ("-s" "--switch" "switch=") ; switch just to cause updates
                ;; single suffix with dynamic description
                ("wa" tsc-suffix-wave
                 :description (lambda ()
                                (format "Wave at %s" (current-time-string))))]
  ;; group with anonymoous function generating description
  [:description (lambda ()
                  (format "Group %s" (org-id-new)))
                ("wu" "wave uniquely" tsc-suffix-wave)])

;; (tsc-layout-dynamic-descriptions)

```

**Note**, the uuid in the group description is generated on every key read, so multi-key sequences cause updates to the descriptions. This is not likely to be changed because layout re-rendering is necessary to indicate the partially complete key sequence. 🤓


### Displaying Information

The `transient-information` class can be used to show states that are purely informative, not having any keys. Just pass `:info` in a suffix declaration to create a display-only element. You can use a constant string or a function for reactivity.

```elisp

(defun tsc--random-info ()
  (format "Temperature outside: %d" (random 100)))

(transient-define-prefix tsc-information ()
  "Prefix that displays some information."
  ["Group Header"
   (:info "Basic info")
   (:info #'tsc--random-info)
   (:info "Use :format to remove whitespace" :format "%d")
   ("k" :info "Keys will be greyed out")
   "" ; empty line
   ("wg" "wave greenishly" tsc-suffix-wave)])

```


<a id="orgf8cf92a"></a>

## Layouts

The default behavior treats groups a little differently depending on how they are nested. For most simple groupings, this is sufficient control.


### Groups one on top of the other

Use a vector for each row.

```elisp

(transient-define-prefix tsc-layout-stacked ()
  "Prefix with layout that stacks groups on top of each other."
  ["Top Group" ("wt" "wave top" tsc-suffix-wave)]
  ["Bottom Group" ("wb" "wave bottom" tsc-suffix-wave)])

;; (tsc-layout-stacked)

```


### Groups side by side

Use a vector of vectors for columns.

```elisp

(transient-define-prefix tsc-layout-columns ()
  "Prefix with side-by-side layout."
  [["Left Group" ("wl" "wave left" tsc-suffix-wave)]
   ["Right Group" ("wr" "wave right" tsc-suffix-wave)]])

;; (tsc-layout-columns)

```


### Group on top of groups side by side

Vector on top of vector inside a vector.

```elisp

(transient-define-prefix tsc-layout-stacked-columns ()
  "Prefix with stacked columns layout."
  ["Top Group"
   ("wt" "wave top" tsc-suffix-wave)]

  [["Left Group"
    ("wl" "wave left" tsc-suffix-wave)]
   ["Right Group"
    ("wr" "wave right" tsc-suffix-wave)]])

;; (tsc-layout-stacked-columns)

```

**Note: Groups can have groups or suffixes, but not both. You can't mix suffixes alongside groups in the same vector. The resulting transient will error when invoked.**


### Empty strings make spaces

Groups that are empty or only space have no effect. This situation can happen with layouts that update dynamically. See [dynamic layouts](#org375878f).

```elisp

(transient-define-prefix tsc-layout-spaced-out ()
  "Prefix lots of spacing for users to space out at."
  ["" ; cannot add another empty string because it will mix suffixes with groups
   ["Left Group"
    ""
    ("wl" "wave left" tsc-suffix-wave)
    ("L" "wave lefter" tsc-suffix-wave)
    ""
    ("bl" "wave bottom-left" tsc-suffix-wave)
    ("z" "zone\n" zone)] ; the newline does pad

   [[]] ; empty vector will do nothing

   [""] ; vector with just empty line has no effect

   ;; empty group will be ignored
   ;; (useful for hiding in dynamic layouts)
   ["Empty Group\n"]

   ["Right Group"
    ""
    ("wr" "wave right" tsc-suffix-wave)
    ("R" "wave righter" tsc-suffix-wave)
    ""
    ("br" "wave bottom-right" tsc-suffix-wave)]])

;; (tsc-layout-spaced-out)

```


### A Grid

So, you put columns into rows that are in columns and stuff like that. This can be achieved with or without explicit column settings.

```elisp

(transient-define-prefix tsc-layout-the-grid ()
  "Prefix with groups in a grid-like arrangement."

  [:description
   "The Grid\n" ; must use slot or macro is confused
   ["Left Column" ; note, no newline
    ("ltt" "left top top" tsc-suffix-wave)
    ("ltb" "left top bottom" tsc-suffix-wave)
    ""
    ("lbt" "left bottom top" tsc-suffix-wave)
    ("lbb" "left bottom bottom" tsc-suffix-wave)] ; note, no newline

   ["Right Column\n"
    ("rtt" "right top top" tsc-suffix-wave)
    ("rtb" "right top bottom" tsc-suffix-wave)
    ""
    ("rbt" "right bottom top" tsc-suffix-wave)
    ("rbb" "right bottom bottom\n" tsc-suffix-wave)]])

;; (tsc-layout-the-grid)

```

**Note**, only `transient-columns`, not `transient-column` can act as a group of groups.


<a id="org706677b"></a>

## Manually setting group class

If you need to override the class that the `transient-define-prefix` macro would normally use.

```elisp

(transient-define-prefix tsc-layout-explicit-classes ()
  "Prefix with group class used to explicitly specify layout."
  [ :class transient-row "Row"
    ("l" "wave left" tsc-suffix-wave)
    ("r" "wave right" tsc-suffix-wave)]
  [ :class transient-column "Column"
    ("t" "wave top" tsc-suffix-wave)
    ("b" "wave bottom" tsc-suffix-wave)])

;; (tsc-layout-explicit-classes)

```


<a id="orgb072dd1"></a>

## Pad Keys

To align descriptions, set the group's :pad-keys to t

```elisp

(transient-define-prefix tsc-layout-padded-keys ()
  "Prefix with padded keys to align descriptions."
  ["Padded Column"
   :class transient-column
   :pad-keys t
   ("t" "wave top" tsc-suffix-wave) ; spaces will be inserted after t
   ("realyongk" "wave bottom" tsc-suffix-wave)])

;; (tsc-layout-padded-keys)

```

Use this if you have different lengths of key sequences or your transient is dynamic and not all keys will have the same length all the time.


<a id="org77d8b3f"></a>

# Nesting & Flow Control

Many transients call other transients. This allows you to express similar behaviors as interactive commands that ask you for multiple arguments using the minibuffer.

Transient has more options for retaining some state across several transients, making it easier to compose commands and to retain intermediate states for rapidly achieving series of actions over similar inputs.


<a id="orga1d0138"></a>

## Single vs Repeat Commands

Sometimes you want to execute multiple commands without re-opening the transient. It's the same idea as [god mode](https://github.com/emacsorphanage/god-mode), Evil repeat, or repeat maps.

```elisp

(transient-define-prefix tsc-stay-transient ()
  "Prefix where some suffixes do not exit."
  ["Exit or Not?"

   ;; this suffix will not exit after calling sub-prefix
   ("we" "wave & exit" tsc-wave-overridden)
   ("ws" "wave & stay" tsc-suffix-wave :transient t)])

;; (tsc-stay-transient)

```

**Note**, if `tsc-wave` was used in both exit & stay, the `:transient` slot would be clobbered and we would only get one behavior. Beware of re-using the same object instances in the same layout. Move the `:transient` slot override between the two suffixes to see the change in behavior.


<a id="org11f66c0"></a>

## Nesting

Nesting is putting transients inside other transients, creating user-input sequences like:

`Prefix -> Sub-Prefix -> Suffix`


### Binding a Sub-Prefix

This is the most simple way to create nesting.

```elisp

(transient-define-prefix tsc--simple-child ()
  ["Simple Child"
   ("wc" "wave childishly" tsc-suffix-wave)])

(transient-define-prefix tsc-simple-parent ()
  "Prefix that calls a child prefix."
  ["Simple Parent"
   ("w" "wave parentally" tsc-suffix-wave)
   ("b" "become child" tsc--simple-child)])

;; (tsc--simple-child)
;; (tsc-simple-parent)

```

-   Nesting & Repeating

    Declaring a nested prefix that "returns" to its parent has a convenient shorthand form.
    
    ```elisp
    
    (transient-define-prefix tsc-simple-parent-with-return ()
      "Prefix with a child prefix that returns."
      ["Parent With Return"
       ("w" "wave parentally" tsc-suffix-wave)
       ("b" "become child with return" tsc--simple-child :transient t)])
    
    ;; Child does not "return" when called independently
    ;; (tsc--simple-child)
    ;; (tsc-simple-parent-with-return)
    
    ```


### Setting up another transient manually

If you call `(transient-setup 'transient-command-symbol)`, you will activate a replacement transient.

This form is useful if you want a command to *perhaps* load yet another transient in some situation. You may even just want to load the same transient with different context, such as passing in a new [scope](#orgcdc6bd0).

```elisp

(transient-define-suffix tsc-suffix-setup-child ()
  "A suffix that uses `transient-setup' to manually load another transient."
  (interactive)
  ;; note that it's usually during the post-command side of calling the
  ;; command that the actual work to set up the transient will occur.
  ;; This is an implementation detail because it depends if we are calling
  ;; `transient-setup' while already transient or not.
  (transient-setup 'tsc--simple-child))

(transient-define-prefix tsc-parent-with-setup-suffix ()
  "Prefix with a suffix that calls `transient-setup'."
  ["Simple Parent"
   ("wp" "wave parentally" tsc-suffix-wave :transient t) ; remain transient

   ;; You may need to specify a different pre-command (the :transient) key
   ;; because we need to clean up this transient or create some conditions
   ;; to trigger the following transient correctly.  This example will
   ;; work with `transient--do-replace' or no custom pre-command

   ("bc" "become child" tsc-suffix-setup-child
    :transient transient--do-replace)])

;; (tsc-parent-with-setup-suffix)

```

⚠️ When the child is calling `transient-setup`, it will not be possible to use `transient--do-return` or `transient--do-recurse` to get back to the parent unless you explicitly cooperate with the transient state implementation, which may not be stable between versions.


<a id="org31f2488"></a>

## Return Without Setup

Sometimes you can complete your work without asking the user for more input. In the custom body for a prefix, if you decline to call `transient-setup`, then the command will just return normally and will not show a prefix menu.

Below is a nested transient.

-   The body form of the nested child can decline to call `transient-setup` leading to a simple return and no menu display
-   The parent uses `transient--do-recurse` to make it's child "return" to it
-   The "radiations" command in the child explicitly overrides this, using `transient--do-exit` so that it *does not* return to the parent

These possible values for `:transient` have been updated a few times. See the [transient](info:transient#Transient State) manual.

```elisp

(defvar tsc--complex nil "Show complex menu or not.")

(transient-define-suffix tsc--toggle-complex ()
  "Toggle `tsc--complex'."
  :transient t
  :description (lambda () (format "toggle complex: %s" tsc--complex))
  (interactive)
  (setf tsc--complex (not tsc--complex))
  (message (propertize (concat "Complexity set to: "
                               (if tsc--complex "true" "false"))
                       'face 'success)))

(transient-define-prefix tsc-complex-messager ()
  "Prefix that sends complex messages, unles `tsc--complex' is nil."
  ["Send Complex Messages"
   ("s" "snow people"
    (lambda () (interactive)
      (message (propertize "snow people! ☃" 'face 'success))))
   ("k" "kitty cats"
    (lambda () (interactive)
      (message (propertize "🐈 kitty cats! 🐈" 'face 'success))))
   ("r" "radiations"
    (lambda () (interactive)
      (message (propertize "Oh no! radiation! ☢" 'face 'error)))
    ;; radiation is dangerous!
    :transient transient--do-exit)]

  (interactive)
  ;; The command body either sets up the transient or simply returns
  ;; This is the "early return" we're talking about.
  (if tsc--complex
      (transient-setup 'tsc-complex-messager)
    (message "Simple and boring!")))

(transient-define-prefix tsc-simple-messager ()
  "Prefix that toggles child behavior!"
  [["Send Message"
    ;; using `transient--do-recurse' causes suffixes in tsc-child to perform
    ;; `transient--do-return' so that we come back to this transient.
    ("m" "message" tsc-complex-messager :transient transient--do-recurse)]
   ["Toggle Complexity"
    ("t" tsc--toggle-complex)]])

;; (tsc-simple-messager) ; toggle complexity on

;; Because `tsc--complex' is in a defvar, its behavior persists when called
;; independently.  Because `tsc-simple-messager' is not in the menu stack when
;; called this way, no return will be performed.
;; (tsc-complex-messager)
```


<a id="org1541a5e"></a>

## Pre-Commands Explained

Before evaluating the command of a suffix, a pre-command function is called and creates the conditions for the suffix to run and for the post-command behavior to decide what to do next. It's usually the "what to do next" part that motivates us to choose a specific pre-command. **Not all pre-commands are compatible with all situations and suffixes!**

If the prefix has any infixes, the pre-command may "export" them. If the pre-command calls `transient-export` then it will also add those infix states to `transient-history`. Only exported infixes can be read within the suffix, which means the correct pre-command is a requirement for using `transient-args`.

The pre-command function will also set up some states so that transient's post-command behavior can figure out if it needs to exit, save values, or setup another transient.

The pre-command is stored in the `:transient` slot and holds a function symbol. When using macros to define prefixes, you will often see `:transient` set to `t`. In `transient-define-prefix` and `transient-define-suffix`, the `t` value is actually translated to a pre-command of `transient--do-call` or `transient--do-recurse` depending on the situation. These export arguments and then result in some menu flow.

The [official long manual](https://magit.vc/manual/transient.html#Transient-State) has some more detail. These examples should prepare you to visualize the forms used in those explanations.


### Note

The following variables and functions at varying points in command lifecycles:

-   `transient-current-command`
-   `transient--command`
-   `transient-current-prefix`
-   `transient--prefix`
-   `transient-args`

During the pre-command and post-command, these can change. Recently some functions such as `transient-prefix-object` were created to "always" do the right thing.

When you are overriding the pre-command, you may discover things such as the result of `transient-args` changing. Calling any of the other lifecycle methods such as `transient-setup` may further mutate states. Customizing this behavior is akin to working on transient itself and does require knowledge of its behavior.

If you need to watch states during the lifecycle, check [Debugging](#org0382dc1)


<a id="orgedb679d"></a>

## Combining With Interactive

You can mix normal Emacs completion & user query flows with transient UI's. Commands with interactive forms that retrieve user input can also be bound as suffixes and will behave normally when called.

See [Interactive codes](info:elisp#Interactive Codes) listed in the Elisp manual for a list of the short codes. The `interactive` docstring also contains a list. You can also construct argument lists manually.

```elisp
(transient-define-suffix tsc--suffix-interactive-buffer-name (buffer-name)
  "An interactive suffix that obtains a buffer name from the user.
This uses the short interactive code."
  (interactive "b")
  (message "You selected: %s" buffer-name))

(transient-define-suffix tsc--suffix-interactive-string (user-input)
  "An interactive suffix that evalutates its arguments exlicitly."
  (interactive (list
                (read-string "Please just tell me what you want!: ")))
  (message "I think you want: %s" user-input))

(transient-define-prefix tsc-interactive ()
  "Prefix with interactive suffixes."
  ["Interactive Command Suffixes"
   ("s" "enter string" tsc--suffix-interactive-string)
   ("b" "select buffer" tsc--suffix-interactive-buffer-name)
   ;; using a normal command with a user query in its interactive form
   ("f" "find file" find-file)])

;; (tsc-interactive)
```


<a id="org6a49527"></a>

# Using & Managing State

There are a lot of ways to handle state, some specific to transient. The [flow control](#org77d8b3f) examples in the previous section mainly covered how to get from one command to the other. This section covers how to save values and then read them later.

To spark your imagination, here's a non-exhaustive list of how to get data into your commands:

-   Interactive forms
-   Prefix arguments (`C-u` universal argument)
-   Setting the scope in `transient-setup`
-   Obtaining a scope in a custom `transient-init-scope` method on a prefix (version 0.8.0+) or a suffix
-   Default values in prefix definition
-   Saved values of infixes
-   Saved values in other infixes / prefixes with shared `history-key`
-   User-set infix values from the current or parent prefix
-   Ad-hoc values in regular `defvar` and `defcustom` etc
-   Read values from a custom file
-   Reading values from another, perhaps distant prefix
-   Arguments passed into interactive commands to call them as normal elisp functions

There are roughly two roads you can go down, which are not exclusive:

1.  Controlling Elisp programs. You will mostly store state using regular Elisp techniques and facilities. You will tend to use the `:description` slot or an `:info` object to display state. Suffixes will modify Elisp state without using transient to specially hold or persist state.

2.  CLI porcelians that make use of infixes and transient history to store state and concatenate completed argument strings from `transient-args` for calling programs like `git`.


<a id="org43774dc"></a>

## Ad-Hoc Elisp State

Firstly, Elisp already has a lot of state, state management, and state persistence facilities. State is mainly stored in values declared in `defvar` or `defcustom` forms. These can be buffer local or global.

The `defcustom` values can be persisted via customize's own persistance in `custom-file`. For `defvar` forms, you can save values to a file and rehydrate them yourself, saving them as frequently and at whatever opportunities, with whatever granularity your program require.

You can display state using `:info` objects and `:description` slots on suffixes. This is the recommended way for non-CLI applications for which infixes are likely excessive indirection.


### Defvars & Defcustom

First let's declare our states and make some commands to update them.

```elisp
(defvar tsc-creativity-subjective "I Just press buttons on my gen-AI"
  "An unverifiable statement about the user's creativity.")

(defvar tsc-creatitity-objective 30
  "User's creativity percentile as assesed by our oracle")

(defun tsc-creativity-subjective-update (creativity)
  "Update the users creativity assessment subjectively."
  (interactive (list (read-string "User subjective creativity: "
                                  tsc-creativity-subjective)))
  (setq tsc-creativity-subjective creativity)
  (message "Subjective creativity updated: %s" tsc-creativity-subjective))

(defun tsc-creativity-objective-update (creativity)
  "Update the users creativity assessment objectively."
  (interactive (list (read-number "User objective creativity: "
                            tsc-creativity-objective)))
  (if (and (integerp creativity) (>= 100 creativity -1))
      (progn (setq tsc-creativity-objective creativity)
             (message "Objective creativity updated: %d"
                      tsc-creativity-objective))
    (user-error "Only integers between 0 and 100 allowed")))
```

You can call `M-x` `tsc--creativity-subjective-update` to test this out setting states with these commands.

Next, let's make some functions to format the states into strings for display and then wire them together within a prefix.

```elisp
(defun tsc--creativity-subjective-describe ()
  "Describe command and display current subjective state."
  (format "subjective: %s" (propertize tsc-creativity-subjective
                                       'face 'transient-value)))

(defun tsc--creativity-objective-describe ()
  "Describe command and display current objective state."
  (format "objective: %s"
          (propertize (number-to-string tsc-creativity-objective)
                      'face 'transient-value)))

;; When we can't jsut use a symbol for what we want to display, write a function
(defun tsc--creativity-display ()
  "Returns a formatted assessment of the users value as a human being."
  (format "User creativity score of %s self-assesses: %s"
          (propertize tsc-creativity-subjective 'face 'transient-value)
          (propertize (number-to-string tsc-creatitity-objective)
                      'face 'transient-value)))

(transient-define-prefix tsc-defvar-settings ()
  "A prefix demonstrating file-based ad-hoc persistence."
  ;; Note the sharpquote (#') used to distinguish a symbol from just a function
  ["Creativity\n"
   (:info #'tsc--creativity-display :format " %d")
   " "
   ("d" tsc-creativity-subjective-update :transient t
    :description tsc--creativity-subjective-describe)
   ("o" tsc-creativity-objective-update :transient t
    :description tsc--creativity-objective-describe)])

;; (tsc-defvar-settings)
```

-   Buffer Locals

    Both defvar and defcustom support being made buffer local by default, so any time they are updated, they become buffer local. The `mode-line-format` is such a variable.
    
    ```elisp
    (defvar-local tsc--mode-line-memento nil
      "A value we can use to restore the `mode-line-format'.")
    
    (defun tsc--toggle-mode-line ()
      "Save and restore the mode line like a pro."
      (interactive)
      (if (null tsc--mode-line-memento)
          (setq tsc--mode-line-memento
                (buffer-local-set-state mode-line-format
                                        "Wh000pTY D000PTY D0000!"))
        (buffer-local-restore-state tsc--mode-line-memento)
        (setq tsc--mode-line-memento nil))
      ;; The mode line won't always redraw if we don't tell the command
      ;; loop about what we did.
      (force-window-update))
    
    (transient-define-prefix tsc-buffer-local ()
      ["Mode Line Gizmo"
       ("m" "toggle modeline" tsc--toggle-mode-line :transient t)])
    
    ;; (tsc-buffer-local)
    ```
    
    It is possible, but at least rare, that you may call a function while transient's UI buffer is the `current-buffer`. Check the `transient--shadowed-buffer` in case of problems.
    
    Note, you should **definitely** know [buffer-local variables](info:elisp#Buffer-Local Variables) very well. This is a foundation Emacs Lisp programming concept.


### File Persistance

Time to make use of homoiconicity! When you write a Lisp form in code to store some data, you can literally write the hydration form into the file so that you just rehydrate the data by loading the file. 🆒

```elisp
;; This just sets the default group for the following defcustom
(defgroup tsc-creativity nil "Creativity" :group 'local)

;; This defvar is a bit longer than strictly necessary.  Lots of users load
;; no-littering early in their init to make Elisp programs save files in more
;; uniform locations.  This expression respects no-littering or works without
;; it.
(defcustom tsc-creativity-file
  (if (and (featurep 'no-littering) (require 'no-littering nil t))
      (no-littering-expand-var-file-name "tsc-creativity.el")
    (expand-file-name "tsc-creativity.el" user-emacs-directory))
  "Where settings are saved to."
  :type 'file)

(defun tsc-creativity-save ()
  "Save the current creativity states."
  (interactive)
  (with-temp-buffer
    " *tsc-peristence*"
    (pp            ; pretty print
     ;; Like writing a macro, you just use quasi-quoting to stitch
     ;; together the structue you want to be in the output.
     `(setq tsc-creativity-subjective ,tsc-creativity-subjective
            tsc-creativity-objective ,tsc-creativity-objective)
     (current-buffer))
    (write-file tsc-creativity-file)
    (message "Transient showcase setting saved!")))

(defun tsc-creativity-load ()
  "Yes, just load what we wrote."
  (interactive)
  (if (file-exists-p tsc-creativity-file)
      (load tsc-creativity-file)
    (user-error "No saved settings exist")))

(defun tsc-creativity-visit-settings ()
  "Show us what we wrote."
  (interactive)
  (if (file-exists-p tsc-creativity-file)
      (find-file tsc-creativity-file)
    (user-error "No saved settings exist")))
```

In the note that we have to sharp-quote (with `'#`) the functions because the `transient-define-prefix` has no other way to know if we mean to use a variable or the function of the same name (Lisp 2).

```elisp
(transient-define-prefix tsc-persistent-settings ()
"A prefix demonstrating file-based ad-hoc persistence."
:refresh-suffixes t
;; Note the sharpquote (#') used to distinguish a symbol from just a function in
;; the :info class.  Info can understand a variable or a function as its value.
["Creativity"
 (:info #'tsc--creativity-display :format " %d")
 ("d" tsc-creativity-subjective-update :transient t
  :description tsc--creativity-subjective-describe)
 ("o" tsc-creativity-objective-update :transient t
  :description tsc--creativity-objective-describe)]
["Persistence"
 ("s" "save" tsc-creativity-save :transient t)
 ("l" "load" tsc-creativity-load :transient t
  :inapt-if-not (lambda () (file-exists-p tsc-creativity-file)))
 ("v" "visit" tsc-creativity-visit-settings :transient t
  :inapt-if-not (lambda () (file-exists-p tsc-creativity-file)))])
```


### Scope in Elisp

Scope is the `:scope` slot on prefixes and suffixes. The value is usually initialized when starting the transient. You should use scope like a function argument, but one that is longer-lived, usually for the entire lifetime of a prefix.

Suffixes that rely on a scope being set but can also be called independently may instead create a scope dynamically in their `:init-scope` method if no prefix is active. It is also possible to read the current scope using `transient-scope` if a prefix is currently active.

See the [Scope](#orgcdc6bd0) section because both Elisp programs and CLI integrations can make use of scope.


### Interactive

Interactive forms are a great way to briefly query the user for information needed to run a command. See [Combining With Interactive](#orgedb679d).


<a id="org04cca78"></a>

## Transient State & Persistence

Transient also has special support for persisting infix values across invocations. The interfaces and behavior are a good fit for CLI integrations. **Coupled with [custom infix types](#orgb1d3d3e), you can create some seriously rich user expression.** Some slots intended for this use case can also be valuable when doing ad-hoc integration with regular Elisp program state.


### Infixes

Functions need arguments. Infixes are specialized suffixes with behavior defaults that make sense for setting and storing values for consumption in suffixes. It's like passing arguments into the suffix. They also have support for persisting state across invocations and Emacs sessions.

-   Basic Infixes

    Infix classes built-in all descend from `transient-infix` and can be seen clearly in the `eieio-browse`. View their slots and documentation with `(describe-class transient-infix)` etc. Here you can see what most infixes look like and how they behave.
    
    ```elisp
    
    ;; infix defined with a macro
    (transient-define-argument tsc--exclusive-switches ()
      "This is a specialized infix for only selecting one of several values."
      :class 'transient-switches
      :argument-format "--%s-snowcone"
      :argument-regexp "\\(--\\(grape\\|orange\\|cherry\\|lime\\)-snowcone\\)"
      :choices '("grape" "orange" "cherry" "lime"))
    
    (transient-define-prefix tsc-basic-infixes ()
      "Prefix that just shows off many typical infix types."
      ["Infixes"
    
       ;; from macro
       ("-e" "exclusive switches" tsc--exclusive-switches)
    
       ;; shorthand definitions
       ("-b" "switch with shortarg" ("-w" "--switch-short"))
       ;; note :short-arg != :key
    
       ("-s" "switch" "--switch")
       ( "n" "no dash switch" "still works")
       ("-a" "argument" "--argument=" :prompt "Let's argue because: ")
    
       ;; a bit of inline EIEIO in our shorthand
       ("-n" "never empty" "--non-null=" :always-read t  :allow-empty nil
        :init-value (lambda (obj) (oset obj value "better-than-nothing")))
    
       ("-c" "choices" "--choice=" :choices (foo bar baz))]
    
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-basic-infixes)
    
    ```

-   Reading Infix Values

    **Reminder** in the section on [pre-commands](#org1541a5e) the discussion about the `:transient` mentions that the values available in a suffix body depend on whether the pre-command called `transient--export` before evaluating the suffix body.
    
    There are three basic ways to read infixes:
    
    -   `(transient-args transient-current-command)` and parse manually
    -   `(transient-arg-value "--argument-" (transient-args transient-current-command)`
    -   `(transient-suffixes transient-current-command)` and retrieve your fully hydrated suffix

-   Lisp Variables

    Lisp variables are currently at an experimental support level. They way they work is to report and set the value of a lisp symbol variable. It's a hybrid of the ideas of the infix class for CLI and regular Elisp program state. Because they aren't necessarilly intended to be printed as crude CLI arguments, they **DO NOT** appear in `(transient-args 'prefix)` but this is fine because you can just use the variable.
    
    Customizing this class can be useful when working with objects and functions that exist entirely in elisp.
    
    ```elisp
    
    (defvar tsc--position '(0 0) "A transient prefix location.")
    
    (transient-define-infix tsc--pos-infix ()
      "A location, key, or command symbol."
      :class 'transient-lisp-variable
      :transient t
      :prompt "An expression such as (0 0), \"p\", nil, 'tsc--msg-pos: "
      :variable 'tsc--position)
    
    (transient-define-suffix tsc--msg-pos ()
      "Message the element at location."
      :transient 'transient--do-call
      (interactive)
      ;; lisp variables are not sent in the usual (transient-args) list.
      ;; Just read `tsc--position' directly.
      (let ((suffix (transient-get-suffix
                     transient-current-command tsc--position)))
        (message "%s" (oref suffix description))))
    
    (transient-define-prefix tsc-lisp-variable ()
      "A prefix that updates and uses a lisp variable."
      ["Location Printing"
       [("p" "position" tsc--pos-infix)]
       [("m" "message" tsc--msg-pos)]])
    
    ;; (tsc-lisp-variable)
    
    ```


### Scope

The scope is somewhat like a function argument, but a bit longer lived. You will likely intialize it when starting a prefix and then read it in suffixes. It is not persited although you can do so manually.

The prefix `:scope` is initialized either when calling `transient-setup` or during the interactive form of the prefix body.

Suffixes can then read back that scope in their body by calling `transient-scope`. The suffix object is given the scope and can use it to alter its own display or behavior. The layout also can interpret the scope while it is initializing. Suffixes may have a `transient-init-scope` method to obtain a scope when called directly without a prefix.

```elisp

(transient-define-suffix tsc--read-prefix-scope ()
  "Read the scope of the prefix."
  :transient 'transient--do-call
  (interactive)
  (let ((scope (transient-scope)))
    (message "scope: %s" scope)))

(transient-define-suffix tsc--double-scope-re-enter ()
  "Re-enter the current prefix with double the scope."
  ;; :transient 'transient--do-replace ; builds up the stack
  :transient 'transient--do-exit
  (interactive)
  (let ((scope (transient-scope)))
    (if (numberp scope)
        (transient-setup transient-current-command
                         nil nil :scope (* scope 2))
      (message
       (propertize
        (format "scope was non-numeric! %s" scope) 'face 'warning))
      (transient-setup transient-current-command))))

(transient-define-suffix tsc--update-scope-with-prefix-re-enter (new-scope)
  "Re-enter the prefix with double the scope."
  ;; :transient 'transient--do-replace ; builds up the stack
  :transient 'transient--do-exit ; do not build up the stack
  (interactive "P")
  (message "universal arg: %s" new-scope)
  (transient-setup transient-current-command nil nil :scope new-scope))

(transient-define-prefix tsc-scope (scope)
  "Prefix demonstrating use of scope."

  [:description
   (lambda () (format "Scope: %s" (transient-scope)))
   [("r" "read scope" tsc--read-prefix-scope)
    ("d" "double scope" tsc--double-scope-re-enter)
    ("o" "update scope (use prefix argument)"
     tsc--update-scope-with-prefix-re-enter)]]
  (interactive "P")
  (transient-setup 'tsc-scope nil nil :scope scope))

;; Setting an interactive argument for `eval-last-sexp' is a
;; little different
;; (let ((current-prefix-arg 4)) (call-interactively 'tsc-scope))

;; (tsc-scope)
;; Then press "C-u 4 o" to update the scope
;; Then d to double
;; Then r to read
;; ... and so on
;; C-g to exit
```

-   Reading Scope Directly

    When writing predicates that read or write the scope using `oref` and `oset`, call `transient-prefix-object` to obtain the correct prefix object for all (most) circumstances. This function correctly handles the edge case where `transient--prefix` must temporarily be used.

-   TODO Errata with prefix arg (`C-u` universal argument).

    Key binding sequences, such as `wa` instead of single-key prefix bindings, will unset the prefix argument (the old-school Emacs `C-u` prefix argument, not the prefix's scope or other explicit arguments)
    
    **Possibly a bug in transient.**


### Prefix Value & History

Briefly, there are three locations for state you need to be aware of for this section:

-   Each transient's prefix object has a `:value` that is updated by both `transient-set` and `transient-save`
-   The values obtained from `transient-args` are usually quite ephemeral and don't even persist beyond the body of form of the suffixes you usually read them in
-   `transient-values` contains saved values that are used to re-hydrate the prefix `:value` slot when the prefix is created
-   `transient-history` is used to make it faster for the user to flip through previous states (which can have independent histories for infixes and prefixes). These are never used unless calling `transient-history-prev` and `transient-history-next`.

We can get this as a list of strings for any prefix by calling `transient-args` on `transient-current-command` in the suffix's interactive form. If you know the command you want the value of, you can use its symbol instead of `transient-current-command`.

This is related to history keys. If you set the arguments and then save them using (`C-x s`) for the command `transient-save`, not only will the transient be updated with the new value, but if you call the child independently, it can still read the value from the suffix.

```elisp

(transient-define-suffix tsc-suffix-eat-snowcone (args)
  "Eat the snowcone!
This command can be called from it's parent, `tsc-snowcone-eater' or independently."
  :transient t
  ;; you can use the interactive form of a command to obtain a default value
  ;; from the user etc if the one obtained from the parent is invalid.
  (interactive (list (transient-args 'tsc-snowcone-eater)))

  ;; `transient-arg-value' can (with varying success) pick out individual
  ;; values from the results of `transient-args'.

  (let ((topping (transient-arg-value "--topping=" args))
        (flavor (transient-arg-value "--flavor=" args)))
    (message "I ate a %s flavored snowcone with %s on top!" flavor topping)))

(transient-define-prefix tsc-snowcone-eater ()
  "Prefix demonstrating set & save infix persistence."

  ;; This prefix has a default value that tsc-suffix-eat-snowcone can see
  ;; even before the prefix has been called.
  :value '("--topping=fruit" "--flavor=cherry")

  ;; always-read is used below so that you don't save nil values to history
  ["Arguments"
   ("-t" "topping" "--topping="
    :choices ("ice cream" "fruit" "whipped cream" "mochi")
    :always-read t)
   ("-f" "flavor" "--flavor="
    :choices ("grape" "orange" "cherry" "lime")
    :always-read t)]

  ;; Definitely check out the =C-x= menu
  ["C-x Menu Behaviors"
   ("S" "save snowcone settings"
    (lambda () (interactive) (message "saved!") (transient-save))
    :transient t)
   ("R" "reset snowcone settings"
    (lambda () (interactive) (message "reset!") (transient-reset))
    :transient t)]

  ["Actions"
   ("m" "message arguments" tsc-suffix-print-args)
   ("e" "eat snowcone" tsc-suffix-eat-snowcone)])

;; First call will use the transient's default value
;; M-x tsc-suffix-eat-snowcone or `eval-last-sexp' below
;; (call-interactively 'tsc-suffix-eat-snowcone)
;; (tsc-snowcone-eater)
;; Eat some snowcones with different flavors
;; ...
;; ...
;; ...
;; Now save the value and exit the transient.
;; When you call the suffix independently, it can still read the saved values!
;; M-x tsc-suffix-eat-snowcone or `eval-last-sexp' below
;; (call-interactively 'tsc-suffix-eat-snowcone)

```

It's worth bringing up the `transient-show-common-commands` variable. **You may want to set this when working on the history support for your transients.** Otherwise, just remember the (`C-x`) menu inside transients.


### History Keys

History lets you **set** infixes using prior values. It's per-prefix, per-suffix usually. Using previous examples like `tsc-snowcone-eater`, you can flip through history using:

-   `C-x p` for `transient-history-prev`
-   `C-x n` for `transient-history-next`

These bindings are revealed when `transient-show-common-commands` is `t` or when you hit the `C-x` prefix.

However, what if you **don't** want a unique history for some infixes or even prefixes?

**Note** As a more advanced example, using EIEIO and dynamic layout techniques to modify the slot of `:history-key`, you can also make unique histories for the same prefix/infix by setting that slot value depending on the context you want unique histories for.

The following example can demonstrate the behavior with some user effort:

```elisp

(transient-define-prefix tsc-ping ()
  "Prefix demonstrating history sharing."

  :history-key 'highly-unique-name

  ["Ping"
   ("-g" "game" "--game=")
   ("p" "ping the pong" tsc-pong)
   ("a" "print args" tsc-suffix-print-args :transient nil)])

(transient-define-prefix tsc-pong ()
  "Prefix demonstrating history sharing."

  :history-key 'highly-unique-name

  ["Pong"
   ("-g" "game" "--game=")
   ("p" "pong the ping" tsc-ping)
   ("a" "print args" tsc-suffix-print-args :transient nil)])

;; (tsc-ping)
;; Okay here's where it gets weird
;; 1.  Set the value of game to something and remember it
;; 2.  Press a to print the args
;; 3.  Re-open tsc-ping.
;; 4.  C-x p to load the previous history, see the old value?
;; 5.  p to switch to the tsc-pong transient
;; 6.  C-x p to load the previous history, see the old value from tsc-ping???
;; 7. Note that tsc-pong uses the same history as tsc-ping!

```

-   Detangling with Initialization, Setting, and Saving

    Set values show up in the prefix's `:value` slot.
    
    ```elisp
    
    (oref (plist-get (symbol-plist 'tsc-ping) 'transient--prefix) value)
    
    ```
    
    The prefix value will get the last value that was **set** using `transient-set`.
    
    However, the prefix value shown in `transient-values` is only updated when calling `transient-save`.
    
    Saved values show up in `transient-values`. If you save `tsc-ping`, you can see the saved value here:
    
    ```elisp
    
    (assoc 'tsc-ping transient-values)
    
    ```
    
    **These two values may be independent.** They are written at the same time when calling `transient-save`. During prefix initialization, the `:value` is written from `transient-values`.
    
    Play with the `tsc-snowcone-eater` and `tsc-ping` and `tsc-pong` in the `C-x` menu while also looking at what gets stored in `transient-values`, `transient-history` and the prefix's slots.
    
    When you re-evaluate the prefix or reload Emacs, you will see the result of initialization from `transient-values`.


### Disabling Set / Save on an Infix

To disable saving and setting values, causing a prefix to always end up using the default value, set the `:unsavable` slot to `t`.

```elisp

(transient-define-prefix tsc-goldfish ()
  "A prefix that cannot remember anything."
  ["Goldfish"
   ("-r" "rememeber" "--i-remember="
    :unsavable t ; infix isn't saved
    :always-read t ; infix always asks for new value
    ;; overriding the method to provide a starting value
    :init-value (lambda (obj) (oset obj value "nothing")))
   ("a" "print args" tsc-suffix-print-args :transient nil)])

;; (tsc-goldfish)

```

Try to update `remember` and then set and save it in the `C-x` menu. Reload it. It will never pay attention to history or setting & saving the transient value.


### Setting or Saving Every Time a Suffix is Used

```elisp

(transient-define-suffix tsc-suffix-remember-and-wave ()
  "Wave, and force the prefix to set it's saveable infix values."
  (interactive)

  ;; (transient-reset) ; forget
  (transient-set) ; save for this session
  ;; If you combine reset save with reset, you get a reset for future
  ;; sessions only.
  ;; (transient-save) ; save for this and future sessions
  ;; (transient-reset-value some-other-prefix-object)

  (message "Waves at user at: %s.  You will never be forgotten." (current-time-string)))

(transient-define-prefix tsc-elephant ()
  "A prefix that always remembers its infixes."
  ["Elephant"
   ("-r" "rememeber" "--i-remember="
    :always-read t)
   ("w" "remember and wave" tsc-suffix-remember-and-wave)
   ("a" "print args (skips remembering)" tsc-suffix-print-args
    :transient nil)])

;; (tsc-elephant)

```

-   Default Values

    Every transient prefix has a value. It's a list. You can set it to create defaults for switches and arguments.
    
    ```elisp
    (transient-define-prefix tsc-default-values ()
      "A prefix with a default value."
    
      :value '("--toggle" "--value=5")
    
      ["Arguments"
       ("t" "toggle" "--toggle")
       ("v" "value" "--value=" :prompt "an integer: ")]
    
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-default-values)
    ```
    
    **Note**, after setting or saving a value on this transient using the `C-x` menu, the next time the transient is set up, it will have a different value. If you want the default to return, use `transient-reset` in your suffix.

-   Readers

    Readers are the mechanism to provide completions and to enforce input validity of infixes.
    
    ```elisp
    (transient-define-prefix tsc-enforcing-inputs ()
      "A prefix with enforced input type."
    
      ["Arguments"
       ("v" "value" "--value=" :prompt "an integer: " :reader transient-read-number-N+)]
    
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-enforcing-inputs)
    ```
    
    Setting the reader can be used to enforce rules of valid input. See [Advanced/Custom Infix Types](#orgb1d3d3e) for an example of writing a custom reader that validates input and assigning that reader via the class method instead of the `:reader` slot.
    
    The section on [flow control](#org77d8b3f) & [managing state](#org6a49527) has more information about controlling elisp applications.


### Switches & Arguments Again

The shorthand forms in `transient-define-prefix` are heavily influenced by the CLI style switches and arguments that transient was built to control. Most shorthand forms look like so:

`("key" "description" "argument")`

The macro will select the infix's exact class depending on how you write `:argument`. If you write something ending in `=` such as `--value=` then you get `:class transient-option` but if not, the default is a `:class transient-switch`

Call `describe-symbol` with `describe-symbol transient-option` and `describe-symbol transient-switch` to see a full document of their slots and methods.

If you need an argument with a space instead of the equal sign, use a space and force the infix to be an argument by setting `:class transient-option`.

```elisp
(transient-define-prefix tsc-switches-and-arguments (arg)
  "A prefix with switch and argument examples."
  [["Arguments"
    ("-s" "switch" "--switch")
    ("-a" "argument" "--argument=")
    ("t" "toggle" "--toggle")
    ("v" "value" "--value=")]

   ["More Arguments"
    ("-f" "argument with forced class" "--forced-class "
     :class transient-option)
    ("I" "argument with inline" ("-i" "--inline-shortarg="))
    ("S" "inline shortarg switch" ("-n" "--inline-shortarg-switch"))]]

  ["Commands"
   ("w" "wave some" tsc-suffix-wave)
   ("s" "show arguments" tsc-suffix-print-args)])
;; use to `tsc-suffix-print-args' to analyze the switch values

;; (tsc-switches-and-arguments)
```

-   Argument and Infix Macros

    If you need to fine-tune a switch (boolean infix), use `transient-define-infix`. Likewise, use `transient-define-argument` for fine-tuning an argument. The class definitions can be used as a reference while the [manual](https://magit.vc/manual/transient/Suffix-Slots.html#Slotsc-of-transient_002dinfix) provides more explanation.
    
    ```elisp
    
    (transient-define-infix tsc--random-init-infix ()
      "Switch on and off."
      :argument "--switch"
      :shortarg "-s" ; will be used for :key when key is not set
      :description "switch"
      :init-value (lambda (obj)
                    (oset obj value
                          (eq 0 (random 2))))) ; write t with 50% probability
    
    (transient-define-prefix tsc-maybe-on ()
      "A prefix with a randomly intializing switch."
      ["Arguments"
       (tsc--random-init-infix)]
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-maybe-on)
    ;; (tsc-maybe-on)
    ;; ...
    ;; Run the command a few times to see the random initialization of
    ;; `tsc--random-init-infix'
    ;; It will only take more than ten tries for one in a thousand users.
    ;; Good luck.
    
    ```

-   Choices

    Choices can be set for an argument. The property API and `transient-define-argument` are equivalent for configuring choices. You can either hard-code or generate choices.
    
    ```elisp
    
    (transient-define-argument tsc--animals-argument ()
      "Animal picker."
      :argument "--animals="
      ;; :multi-value t
      ;; :multi-value t means multiple options can be selected at once, such as:
      ;; --animals=fox,otter,kitten etc
      :class 'transient-option
      :choices '("fox" "kitten" "peregrine" "otter"))
    
    (transient-define-prefix tsc-animal-choices ()
      "Prefix demonstrating selecting animals from choices."
      ["Arguments"
       ("-a" "--animals=" tsc--animals-argument)]
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-animal-choices)
    
    ```
    
    -   Choices shorthand in prefix definition
    
        Choices can also be defined in a shorthand form. Use `:class 'transient-option` if you need to force a different class to be used.
        
        ```elisp
        
        (transient-define-prefix tsc-animal-choices-shorthand ()
          "Prefix demonstrating the shorthand style of defining choices."
          ["Arguments"
           ("-a" "Animal" "--animal=" :choices ("fox" "kitten" "peregrine" "otter"))]
          ["Show Args"
           ("s" "show arguments" tsc-suffix-print-args)])
        
        ;; (tsc-animal-choices-shorthand)
        
        ```

-   Mutually Exclusive Switches

    An argument with `:class transient-switches` may be used if a set of switches is exclusive. The key will likely *not* match the short argument. Regex is used to tell the interface that you are entering one of the choices. The selected choice will be inserted into `:argument-format`. The `:argument-regexp` must be able to match any of the valid options.
    
    **The UX on mutually exclusive switches is a bit of a pain to discover. You must repeatedly press `:key` in order to cycle through the options.**
    
    ```elisp
    
    (transient-define-argument tsc--snowcone-flavor ()
      :description "Flavor of snowcone."
      :class 'transient-switches
      :key "f"
      :argument-format "--%s-snowcone"
      :argument-regexp "\\(--\\(grape\\|orange\\|cherry\\|lime\\)-snowcone\\)"
      :choices '("grape" "orange" "cherry" "lime"))
    
    (transient-define-prefix tsc-exclusive-switches ()
      "Prefix demonstrating exclusive switches."
      :value '("--orange-snowcone")
    
      ["Arguments"
       (tsc--snowcone-flavor)]
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-exclusive-switches)
    
    ```

-   Incompatible Switches

    If you need to prevent arguments in a group from being set simultaneously, you can set the prefix property `:incompatible` and a list of the long-style argument.
    
    Use a list of lists, where each sub-list is the long argument style. Match the string completely, including use of `=` in both arguments and switches.
    
    ```elisp
    (transient-define-prefix tsc-incompatible ()
      "Prefix demonstrating incompatible switches."
      ;; update your transient version if you experience #129 / #155
      :incompatible '(("--switch" "--value=")
                      ("--switch" "--toggle" "--flip")
                      ("--argument=" "--value=" "--special-arg="))
    
      ["Arguments"
       ("-s" "switch" "--switch")
       ("-t" "toggle" "--toggle")
       ("-f" "flip" "--flip")
    
       ("-a" "argument" "--argument=")
       ("v" "value" "--value=")
       ("C-a" "special arg" "--special-arg=")]
    
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-incompatible)
    ```

-   TODO Short Args

    **This section is incomplete. Maybe Magit contains better answers.**
    
    Sometimes the `:shortarg` in a CLI doesn't exactly match the `:key:` and `:argument`, so it can be specified manually.
    
    The `:shortarg` concept could be used to help use man-pages or only for [transient-detect-key-conflicts](https://magit.vc/manual/transient.html#index-transient_002ddetect_002dkey_002dconflicts) but it's not clear what behavior it changes.
    
    Shortarg cannot be used for exclusion excluding other options (prefix `:incompatible`) or setting default values (prefix `:value`).

-   Dynamic Choices

    See `transient-infix-read` for actual code. This method uses the prefix's history and then delecates to `completing-read` or `completing-read-multiple`. The `:choices` key coresponds to the `COLLECTION` argument passed to completing reads.
    
    **Note**, using a function for completions can appear to require a daunting amount of behavior if you read the manual [section on programmed completions](info:elisp#Programmed Completion). If you however just return a list of options, even when FLAG is not t, everything seems just fine.
    
    ```elisp
    (defun tsc--animal-choices (_complete-me _predicate flag)
      "Programmed completion for animal choice.
    _COMPLETE-ME: whatever the user has typed so far
    _PREDICATE: function you should use to filter candidates (only nil seen so far)
    FLAG: request for metadata (which can be disrespected)"
    
      ;; if you want to respect metadata requests, here's what the form might
      ;; look like, but no behavior was observed.
      (if (eq flag 'metadata)
          '(metadata . '((annotation-function . (lambda (c) "an annotation"))))
    
        ;; when not handling a metadata request from completions, use some
        ;; logic to generate the choices, possibly based on input or some time
        ;; / context sensitive process.  FLAG will be `t' when these are
        ;; reqeusted.
        (if (eq 0 (random 2))
            '("fox" "kitten" "otter")
          '("ant" "peregrine" "zebra"))))
    
    (transient-define-prefix tsc-choices-with-completions ()
      "Prefix with completions for choices."
      ["Arguments"
       ("-a" "Animal" "--animal="
        :always-read t ; don't allow unsetting, just read a new value
        :choices tsc--animal-choices)]
      ["Show Args"
       ("s" "show arguments" tsc-suffix-print-args)])
    
    ;; (tsc-choices-with-completions)
    ```


### Dispatching args into a process

If you want to call a command line application using the arguments, you might need to do a bit of work processing the arguments. The following example uses cowsay.

-   Cowsay doesn't actually have a `message` argument, So we end up stripping it from the arguments and re-assembling something `call-process` can use.

-   Cowsay supports more options, but for the sake of keeping this example small (and to refocus effort on transient itself), the set of all CLI options are not fully supported.

There's some errata about this example:

-   The predicates don't update the transient. `(transient--redisplay)` doesn't do the trick. We could use `transient--do-replace` and `transient-setup`, but that would lose existing state unless we run `transient-set`

-   The predicate needs to be exists & not empty (but doesn't matter yet)

✨ If you are working on a CLI tool in order to fit a transient interface, consider a JSON-RPC process because you can build a normal command interface and dispatch it with transient even if you skip the CLI argument handling facilities. CLI's are more fragile than JSON-RPC, and JSON-RPC processes can retain state.

```elisp
(defun tsc--quit-cowsay ()
  "Kill the cowsay buffer and exit."
  (interactive)
  (kill-buffer "*cowsay*"))

(defun tsc--cowsay-buffer-exists-p ()
  "Visibility predicate."
  (not (equal (get-buffer "*cowsay*") nil)))

(transient-define-suffix tsc--cowsay-clear-buffer (&optional buffer)
  "Delete the *cowsay* buffer.  Optional BUFFER name."
  :transient 'transient--do-call
  :if 'tsc--cowsay-buffer-exists-p
  (interactive) ; todo look at "b" interactive code

  (save-excursion
    (let ((buffer (or buffer "*cowsay*")))
      (set-buffer buffer)
      (delete-region 1 (+ 1 (buffer-size))))))

(transient-define-suffix tsc--cowsay (&optional args)
  "Run cowsay."
  (interactive (list (transient-args transient-current-command)))
  (let* ((buffer "*cowsay*")
         ;; TODO ugly
         (cowmsg (if args (transient-arg-value "--message=" args) nil))
         (cowmsg (if cowmsg (list cowmsg) nil))
         (args (if args
                   (seq-filter
                    (lambda (s) (not (string-prefix-p "--message=" s))) args)
                 nil))
         (args (if args
                   (if cowmsg
                       (append args cowmsg)
                     args)
                 cowmsg)))

    (when (tsc--cowsay-buffer-exists-p)
      (tsc--cowsay-clear-buffer))
    (apply #'call-process "cowsay" nil buffer nil args)
    (switch-to-buffer buffer)))

(transient-define-prefix tsc-cowsay ()
  "Say things with animals!"
  ;; only one kind of eyes is meaningful at a time
  :incompatible '(("-b" "-g" "-p" "-s" "-t" "-w" "-y"))

  ["Message"
   ("m" "message" "--message=" :always-read t)]
  ;; always-read, so clear by entering empty string
  [["Built-in Eyes"
    ("b" "borg" "-b")
    ("g" "greedy" "-g")
    ("p" "paranoid" "-p")
    ("s" "stoned" "-s")
    ("t" "tired" "-t")
    ("w" "wired" "-w")
    ("y" "youthful" "-y")]
   ["Actions"
    ("c" "cowsay" tsc--cowsay :transient transient--do-call)
    ""
    ("d" "delete buffer" tsc--cowsay-clear-buffer)
    ("q" "quit" tsc--quit-cowsay)]])

;; (tsc-cowsay)
```


<a id="orgfd72d76"></a>

# Input Sentence Construction

In the end, transient menus and keymaps both simply convert keystrokes into commands. Several commands can be viewed as a sentence. Keymaps and transient have identical capabilities to construct sentences.

Because transient commands tend to be used together in clusters, it is more common for them to be stateful, which re-uses user input from earlier commands, making richer input expression.

Using flow control, you can create wizard-like menus to assemble complex state. You can display state, making it easier to see what complex commands will do. The bindings are contextual, so while the input language is rich, it is intuitive.

Using all of the flow control and state management capabilities, you can enable users to rapidly construct complex command sentences, sentences with phrases. You can basically make a user interface as expressive as elisp.

If you *think* in terms of partially constructed elisp expressions, you can do more than if the user has to re-enter the context for commands over and over. For a moment, relax your idea of an infix to include any suffix that is non-terminal and updates state for subsequent commands to consume.

A user input sequence like this:

`Prefix -> Interactive -> Sub-Prefix -> Infix -> Infix -> Infix -> Suffix -> Suffix -> ... -> Suffix`

Is basically the same as doing this in elisp:

```elisp

(let ((xyz (Sub-Prefix (Prefix (Interactive))))
      (a (infix-a))
      (b (infix-b))
      (c (infix-c)))
  (suffix xyz a b c)
  (suffix xyz a b c)
  ;; ...and so on
  (suffix xyz a b c))

```

When re-issuing slightly modified commands in a CLI, such as adding an extra switch, this is how the command language works. However, CLI input usually requires remembering lots of switches and using editing style workflows rather than a modal keymap specific to that task.


<a id="org262016e"></a>

# Controlling Visibility

At times, you need a prefix to show or hide certain options depending on the context.


<a id="orge9d5fb5"></a>

## Visibility Predicates

Simple [predicates](https://magit.vc/manual/transient/Predicate-Slots.html#Predicate-Slots) at the group or element level exist to hide parts of the transient when they wouldn't be useful at all in the situation.

```elisp

(defvar tsc-busy nil "Are we busy?")

(defun tsc--busy-p () "Are we busy?" tsc-busy)

(transient-define-suffix tsc--toggle-busy ()
  "Toggle busy."
  (interactive)
  (setf tsc-busy (not tsc-busy))
  (message (propertize (format "busy: %s" tsc-busy)
                       'face 'success)))

```

Open the following example in buffers with different modes (or change modes manually) to see the different effects of the mode predicates.

```elisp

(transient-define-prefix tsc-visibility-predicates ()
  "Prefix with visibility predicates.
Try opening this prefix in buffers with modes deriving from different
abstract major modes."
  ["Empty Groups Not Displayed"
   ;; in org mode for example, this group doesn't appear.
   ("we" "wave elisp" tsc-suffix-wave :if-mode emacs-lisp-mode)
   ("wc" "wave in C" tsc-suffix-wave :if-mode cc-mode)]

  ["Lists of Modes"
   ("wm" "wave multiply" tsc-suffix-wave :if-mode (dired-mode gnus-mode))]

  [["Function Predicates"
    ;; note, after toggling, the transient needs to be re-displayed for the
    ;; predicate to take effect
    ("tb" "toggle busy" tsc--toggle-busy :transient t)
    ("bw" "wave busily" tsc-suffix-wave :if tsc--busy-p)]

   ["Programming Actions"
    :if-derived prog-mode
    ("pw" "wave programishly" tsc-suffix-wave)
    ("pe" "wave in elisp" tsc-suffix-wave :if emacs-lisp-mode)]
   ["Special Mode Actions"
    :if-derived special-mode
    ("sw" "wave specially" tsc-suffix-wave)
    ("sd" "wave dired" tsc-suffix-wave :if-mode dired-mode)]
   ["Text Mode Actions"
    :if-derived text-mode
    ("tw" "wave textually" tsc-suffix-wave)
    ("to" "wave org-modeishly" tsc-suffix-wave :if-mode org-mode)]])

;; (tsc-visibility-predicates)

```


<a id="org99a9bab"></a>

## Inapt (Temporarily Inappropriate)

"Greyed out" suffixes. Inapt is better if an option is temporarily unavailable but the user should still know it exists.

✨ Adjust this example to use `:if` instead of `:inapt-if` to see the difference between visibility and inapt.

```elisp

(defun tsc--switch-on-p ()
  (transient-arg-value
   "--switch"
   (transient-args transient-current-command)))

(transient-define-prefix tsc-inapt ()
  "Prefix that configures child with inapt predicates."
  :refresh-suffixes t ; important for updating inapt! (1)
  ["Options"
   ("-s" "switch" "--switch"
    ;; we want to see the most recent value in `transient-args' (2)
    :transient transient--do-call)]

  ["Appropriate Suffixes"
   ("s" "switched" tsc--wave-switchedly
    :transient t
    :inapt-if-not tsc--switch-on-p)
   ("u" "unswitched" tsc--wave-unswitchedly
    :transient t
    :inapt-if tsc--switch-on-p)]

  ["Appropiate Group"
   :inapt-if-not tsc--switch-on-p
   ("q" "query" tsc--wave-inquisitively)
   ("w" "write" tsc--wave-writingly)])

;; (tsc-inapt)

```

1.  **Note**, like visibility predicates, `inapt-` predicates do not **by default** take effect until the transient has its layout fully redone. You must set the `:refresh-suffixes` slot on the **prefix**.

2.  The use of `transient--do-call` has a nuance. Normally, changing an infix does not update the most recent value seen when calling `transient-args`. However, when the infix uses `transient--do-call` as it's pre-command, the value seen will be updated. This allows `tsc--switch-on-p` to see the updated value and for the inapt predicates to function properly.


### Errata

Calling `transient-args` with the symbol of the current command (`tsc-inapt`) in `tsc--switch-on-p` causes infinite recursion as `transient-args` attempts to hydrate the transient instead of re-using the value that already is in motion via `transient-current-command`.


<a id="orgf55ca43"></a>

## Levels

Levels are another way to control visibility.

-   As a developer, you set levels to optionally expose or hide children in a prefix.
-   As a user, you change the prefix's level and the levels of suffixes to customize what's visible in the transient.

**Lower levels are more visible. Setting the level higher reveals more suffixes.** 1-7 are valid levels.

The user can adjust levels within a transient prefix by using (**C-x l**) for `transient-set-level`. The default active level is 4, stored in `transient-default-level`. The default level for children is 1, stored in `transient--default-child-level`.

Per-suffix and per-group, the user can set the level at which the child will be visible. Each prefix has an active level, remembered per prefix. If the child level is less-than-or-equal to the child level, the child is visible.

A hidden group will hide a suffix even if that suffix is at a low enough level. Issue #153 has some additional information about behavior that might get cleaned up.


### Defining group & suffix levels

Adding default levels for children is as simple as adding integers at the beginning of each list or vector. If some commands are not likely to be used, instead of making the hard choice to include them or not, you can provide them, but tell the user in your README to set higher levels.

```elisp

(transient-define-prefix tsc-levels-and-visibility ()
  "Prefix with visibility levels for hiding rarely used commands."

  [["Setting the Current Level"
    ;; this binding is normally not displayed.  The value of
    ;; `transient-show-common-commands' controls this by default.
    ("C-x l" "set level" transient-set-level)
    ("s" "show level" tsc-suffix-show-level)]

   [2 "Per Group" ;; 1 is the default default-child-level
      ("ws" "wave surely" tsc--wave-surely)
      (3"wn" "wave normally" tsc--wave-normally)
      (5"wb" "wave non-essentially" tsc--wave-non-essentially)]

   [3 "Per Group Somewhat Useful"
      ("wd" "wave definitely" tsc--wave-definitely)]

   [6 "Groups hide visible children"
      (1 "wh" "wave hidden" tsc--wave-hidden)]

   [5 "Per Group Rarely Useful"
      ("we" "wave eventually" tsc--wave-eventually)]])

;; (tsc-levels-and-visibility)

```


### Using the Levels UI

Press (**C-x l**) to open the levels UI for the user. Press (**C-x l**) again to change the active level. Press a key such as "we" to change the level for a child. After you cancel level editing with (**C-g**), you will see that children have either become visible or invisible depending on the changes you made.

**While a child may be visible according to its own level, if it's hidden within the group, the user's level-setting UI for the prefix will contradict what's actually visible. The UI does not allow setting group levels.**


<a id="org43fd2f5"></a>

# Advanced

The previous sections are designed to go breadth-first so that you can get core ideas first. The following examples expand on combinations of several ideas or subclassing & customizing rarely used slots.

Some of these examples are approaching the complexity of just reading [magit source](elisp:(find-library "magit")).


<a id="org375878f"></a>

## Dynamically generating layouts

While you can cover many cases using predicates, layouts, and visibility, **sometimes you really do want to generate a list of commands.**

**Note**, beware that you could be creating a lot of suffix objects if the forms you use generate unique symbols. These will pollute command completions over time, so probably don't do that.

[transient-setup-children](https://magit.vc/manual/transient.html#index-transient_002dsetup_002dchildren)

This is a group method that can be overridden in order to modify or eliminate some children from display. If you need a central place for children to coordinate some behavior, this may work for you.

```elisp

(transient-define-prefix tsc-generated-child ()
  "Prefix that uses `setup-children' to generate single child."

  ["Replace this child"
   ;; Let's override the group's method
   :setup-children
   (lambda (_) ; we don't care about the stupid suffix

     ;; remember to return a list
     (list (transient-parse-suffix
            'transient--prefix
            '("r" "replacement" (lambda ()
                                  (interactive)
                                  (message "okay!"))))))

   ;; This child will not be visible when you run the example because it is
   ;; replaced dynamically when the transient is set up
   ("s" "haha stupid suffix" (lambda ()
                               (interactive)
                               (message "You should replace me!")))])

;; (tsc-generated-child)

```

`transient--parse-child` takes the same configuration format as `transient-define-prefix`. You can see the layout format in the [layout hacking appendix](#orge18eaeb). `transient--prarse-group` works almost exactly the same, just for groups.

The same thing, but parsing an entire group spec:

```elisp

(transient-define-prefix tsc-generated-group ()
  "Prefix that uses `setup-children' to generate a group."

  ["Replace this child"
   ;; Let's override the group's method
   :setup-children
   (lambda (_) ; we don't care about the stupid suffix

     ;; the result of parsing here will be a group
     (transient-parse-suffixes
      'transient--prefix
      ["Group Name" ("r" "replacement" (lambda ()
                                         (interactive)
                                         (message "okay!")))]))

   ;; This child will not be visible when you run the example because it is
   ;; replaced dynamically when the transient is set up
   ("s" "haha stupid suffix" (lambda ()
                               (interactive)
                               (message "You should replace me!")))])

;; (tsc-generated-group)

```

If you need to define a dynamic group class, override `transient-setup-children`. It will work almost entirely the same as the examples above. Set your group class in the prefix using the `:class` key.

**Note** you can replace `transient--prefix` with `tsc-generated-group` in the example above. `transient--prefix` is just a variable that happens to hold the prefix during layout. The quoting is strange because of the outer `transient-define-prefix` macro.

**Note** you don't need to be inside of a layout body to hack around with dynamic layouts. Mess around in [ielm](elisp:(ielm))):

```elisp

(transient--parse-child 'magit-dispatch '("a" "action" (lambda () (interactive) (message "hey"))))

```


### TODO Question

-   These functions do mostly the same job. Why do we need to specify a prefix for `transient-parse-suffixes`, for scope etc?


<a id="orgc932d74"></a>

## Modifying layouts

In this example, we will make a transient that can add new commands to itself.

```elisp

(defun tsc--self-modifying-add-command (command-symbol sequence)
  (interactive "CSelect a command: \nMkey sequence: ")

  ;; Generate an infix that will call the command and add it to the
  ;; second group (index 1 at the 0th position)
  (transient-insert-suffix
    'tsc-self-modifying
    '(0 1 0) ; set the child in `tsc-inception' for help with this argument
    (list sequence (format "Call %s" command-symbol) command-symbol :transient t))

  ;; we must re-enter the transient to force the layout update
  (transient-setup 'tsc-self-modifying))

(transient-define-prefix tsc-self-modifying ()
  "Prefix that uses `transient-insert-suffix' to add commands to itself."

  [["Add New Commands"
    ("a" "add command" tsc--self-modifying-add-command)]
   ["User Defined"
    ""]]) ; blank line suffix creates an insertion point

;; (tsc-self-modifying)

```

Exercises for the reader:

-   Use the interactive form to read an elisp expression and create an anonymous command
-   Add a command to remove suffixes
-   Create a suffix editor interface, modifying the description, key, command, or other slots

See the `transient-insert-suffix` for documentation on the `LOC` argument.


<a id="org050dd4c"></a>

## Using prefix scope in children

Call `transient-scope`!


### Obtaining Missing Scope

Because suffixes are basically also commands (riding in the same symbol plist), a suffix can be called independently. In this case, if its expecting to read the scope from the prefix when there is no prefix, it might fail.

Therefore, a method called `transient-init-scope` can be overridden and used at the correct point in the life-cycle for the suffix to correct the issue.

**Note**, the behavior is actually quite ad-hoc. You will read the prefix yourself and then decide if you want to use some fallback.

There is a perfectly short example in [Magit source](https://github.com/magit/magit/blob/40fb3d20026139ad1c3a3d9069b40d7d61bf8786/lisp/magit-transient.el#L56-L61) for the custom `magit--git-variable` subclass of the `transient-variable` infix.

Each infix instance is declared in `transient-define-infix`, potentially with a `:scope` slot, possibly holding a function.

If it's holding a function, that function will be used as a backup during initialization in case there is no prefix or it has nothing in its `:scope` slot.


<a id="orgb1d3d3e"></a>

## Custom Infix Types

Not everything is a string or boolean. You may want to represent complex objects in your transient infixes. If your objects can be re-hydrated from some serialized ID, you may want history support.

If you need to set and display a custom type, use the simple OOP techniques of [EIEIO](#org6901288). Also check the [suffix value methods](info:transient#Suffix Value Methods) section of the transient manual. The following example applies these ideas.

**Essential behaviors for your custom infix:**

-   Defining a reader to set the infix with user input
-   `:prompt` slot's default form, `:initform` for asking the user for input
-   `transient-init-value` to re-hydrate saved values
-   `transient-infix-value` so that setting & saving persist what you want to rehydrate
-   `transient-format-value` to display a user-meaningful form for your value

We will also use some layout introspection. This makes the example a bit more complex, but represents a real custom infix type with real serialization and elisp objects backing it:

-   `transient-get-suffix` To get suffix by **key**, **location**, or **command symbol**
-   Getting a description from raw layout children (not EIEIO objects). See [Layout Hacking](#orge18eaeb).

This example is a bit intimidating because the serialized value we are storing and re-hydrating is a layout child location, the LOC argument seen in transient programming. It maps to an actual layout child, which we introspect and later modify. The point of the example is to let the user handle a simple value that we can also persist but to use a more complex object that might only exist at runtime. If this example makes little sense, try making an example with just a string or number before you start your own data type.

```elisp

;; The children we will be picking can be of several forms.  The
;; transient--layout symbol property of a prefix is a vector of vectors,
;; lists, and strings.  It's not the actual eieio types or we would use
;; `transient-format-description' to just ask them for the descriptions.
(defun tsc--layout-child-desc (layout-child)
  "Get the description from LAYOUT-CHILD.
LAYOUT-CHILD is a transient layout vector or list."
  (let ((description
         (cond
          ((vectorp layout-child)
           (or (plist-get (aref layout-child 2) :description)
               "<group, no desc>")) ; group
          ((stringp layout-child) layout-child) ; plain-text child
          ((listp layout-child)
           (plist-get (elt layout-child 2) :description)) ; suffix
          (t
           (message
            (propertize "You traversed into a child's list elements!"
                        'face 'warning))
           (format "(child's interior) element: %s" layout-child)))))
    (cond
     ;; The description is sometimes a callable function with no arguments,
     ;; so let's call it in that case.  Note, the description may be
     ;; designed for one point in the transient's lifecycle but we could
     ;; call it in a different one, causing its behavior to change.
     ((functionp description) (apply description))
     (t description))))

;; We repeat the read using a lisp expression from `read-from-minibuffer' to
;; get the LOC key for `transient-get-suffix' until we get a valid result.
;; This ensures we don't store an invalid LOC.
(defun tsc-child-infix--reader (prompt initial-input history)
  "Read a location and check that it exists within the current transient.
PROMPT, INITIAL-INPUT, and HISTORY are forwarded to `read-from-minibuffer'."
  (let ((command (oref transient--prefix command))
        (success nil))
    (while (not success)
      (let* ((loc (read (read-from-minibuffer
                         prompt initial-input nil nil history)))
             (child (ignore-errors (transient-get-suffix command loc))))
        (if child (setq success loc)
          (message (propertize
                    (format
                     "Location could not be found in prefix %s"
                     command)
                    'face 'error))
          (sit-for 3))))
    success))

;; Inherit from variable abstract class
(defclass tsc-child-infix (transient-variable)
  ((value-object :initarg value-object :initform nil)
   ;; this is a new slot for storing the hydrated value.  we re-use the
   ;; value infrastructure for storing the serialization-friendly value,
   ;; which is basically a suffix addres or id.

   (reader :initform #'tsc-child-infix--reader)
   (prompt :initform
           "Location, a key \"c\",\ suffix-command-symbol like\ tsc--wave-normally or coordinates like (0 2 0): ")))

;; We have to define this on non-abstract infix classes.  See
;; `transient-init-value' in transient source.  The method on
;; `transient-argument' class was used to make this example, but it
;; does support a lot of behaviors.  In short, the prefix has a value
;; and you rehydrate the infix by looking into the prefix's value to
;; find the suffix value.  Because our stored value is basically a
;; serialization, we rehydrate it to be sure it's a valid value.
;; Remember to handle values you can't rehydrate.
(cl-defmethod transient-init-value ((obj tsc-child-infix))
  "Set the `value' and `value-object' slots using the prefix's value."

  ;; in the prefix declaration, the initial description is a reliable key
  (let ((variable (oref obj description)))
    (oset obj variable variable)

    ;; rehydrate the value if the prefix has one for this infix
    (when-let* ((prefix-value (oref transient--prefix value))
                ;; (argument (and (slot-boundp obj 'argument)
                ;;   (oref obj argument)))
                (value (cdr (assoc variable prefix-value)))
                ;; rehydrate
                (value-object (transient-get-suffix
                               (oref transient--prefix command) value)))
      (oset obj value value)
      (oset obj value-object value-object))))

(cl-defmethod transient-infix-set ((obj tsc-child-infix) value)
  "Update `value' slot to VALUE.
Update `value-object' slot to the value corresponding to VALUE."
  (let* ((command (oref transient--prefix command))
         (child (ignore-errors (transient-get-suffix command value))))
    (oset obj value-object child)
    (oset obj value (if child value nil)))) ; TODO a bit ugly

;; If you are making a suffix that needs history, you need to define
;; this method.  The example here almost identical to the method
;; defined for `transient-option',
(cl-defmethod transient-infix-value ((obj tsc-child-infix))
  "Return our actual value for rehydration later."

  ;; Note, returning a cons for the value is very flexible and will
  ;; work with homoiconicity in persistence.
  (cons (oref obj variable) (oref obj value)))

;; Show user's a useful representation of your ugly value
(cl-defmethod transient-format-value ((obj tsc-child-infix))
  "All transient children have some description we can display.
Show either the child's description or a default if no child is selected."
  (if-let* ((value (and (slot-boundp obj 'value) (oref obj value)))
            (value-object (and (slot-boundp obj 'value-object)
                               (oref obj value-object))))
      (propertize
       (format "(%s)" (tsc--layout-child-desc value-object))
       'face 'transient-value)
    (propertize "¯\_(ツ)_/¯" 'face 'transient-inactive-value)))

;; Now that we have our class defined, we can create an infix the usual
;; way, just specifying our class
(transient-define-infix tsc--inception-child-infix ()
  :class tsc-child-infix)

;; All set!  This transient just tests our or new toy.
(transient-define-prefix tsc-inception ()
  "Prefix that picks a suffix from its own layout."

  [["Pick a suffix"
    ("-s" "just a switch" "--switch") ; makes history value structure apparent
    ("c" "child" tsc--inception-child-infix)]

   ["Some suffixes"
    ("s" "wave surely" tsc--wave-surely)
    ("d" "wave definitely" tsc--wave-definitely)
    ("e" "wave eventually" tsc--wave-eventually)
    ("C" "call & exit normally" tsc--wave-normally :transient nil)]

   ["Read variables"
    ("r" "read args" tsc-suffix-print-args )]])

;; (tsc-inception)
;;
;; Try setting the infix to "e" (yes, include quotes)
;; Try: (1 2)
;; Try: tsc--wave-normally
;;
;; Observe that the LOC you enter is displayed using the description at that
;; point
;;
;; Set the infix and re-open it with C-x s, C-g, and M-x tsc-inception
;; Observe that the set value persists across invocations
;;
;; Save the infix, with C-x C-s, re-evaluate the prefix, and open the
;; prefix again.
;;
;; Try flipping through history, C-x n, C-x p
;; Now do think of doing things like this with org ids, magit-sections,
;; buffers etc.

```

This is a difficult example, but once you understand the pieces, you can see some of the magit variables in action like `magit--git-variable` and its many subclasses.

Revisit the section on [detangling setting, saving and history](#org8ff2432). Watching the values update will make it clear what representations are being stored, where, and when.


### Reading custom infix values

**Note**, however you store and rehydrate will affect how you read, so try to make it just work with `transient-read-arg`, unlike this example (TODO).

```elisp

(transient-define-suffix tsc--inception-update-description ()
  "Update the description of of the selected child."
  (interactive)
  (let* ((args (transient-args transient-current-command))
         (description (transient-arg-value "--description=" args))
         ;; This is the part where we read the other infix.  It's
         ;; similar to how we find the value during rehydration, but
         ;; hard-coding the infix's argument, "child", which is used
         ;; in its `transient-infix-value' method.
         (loc (cdr (assoc "child" args)))
         (layout-child (transient-get-suffix 'tsc-inception-update loc)))

    ;; Once again, do different bodies based on what we found at the
    ;; layout locition.  This complexity is beacuse of the data we
    ;; are operating on, not the transient methods we needed to
    ;; implement.
    (cond
     ((or (listp layout-child) ; child
          (vectorp layout-child) ; group
          (stringp layout-child)) ; string child
      (if (stringp layout-child) ; plain-text child
          (transient-replace-suffix 'tsc-inception-update loc description)

        (plist-put (elt layout-child 2) :description description)))
     (t (message
         (propertize
          (format "Don't know how to modify whatever is at: %s" loc)
                     'face 'warning))))

    ;; re-enter the transient manually to display the modified layout
    (transient-setup transient-current-command)))

(transient-define-prefix tsc-inception-update ()
  "Prefix that picks and updates its own suffix."

  [["Pick a suffix"
    ("c" "child" tsc--inception-child-infix :argument "child")]

   ["Update the description!"
    ("-d" "description" "--description=")
    ("u" "update" tsc--inception-update-description
     :transient transient--do-exit)]

   ["Some suffixes"
    ("s" "wave surely" tsc--wave-surely)
    ("d" "wave definitely" tsc--wave-definitely)
    ("e" "wave eventually" tsc--wave-eventually)
    ("C" "call & exit normally" tsc--wave-normally :transient nil)]

   ["Read variables"
    ("r" "read args" tsc-suffix-print-args )]])

;; (tsc-inception-update)
;;
;; 1. Press 'c' to start picking a suffix.  For example, enter the string "e"
;; 2. Press 'C-x s' to set the values of this transient for the future
;; 3. Then set the description, anything, no quotes
;; 4. Then press 'u' the suffix's you picked with the new description!
;;
;; Using a transient to modify a transient (⊃｡•́‿•̀｡)⊃━✿✿✿✿✿✿
;;
;; Observe that the set values are persisted across invocations.
;; Saving also works.  This makes it easier to set the description
;; multiple times in succession.  The Payoff when building larger
;; applications like magit rapidly adds up.

```


### TODO Errata

Modifying the very outer group doesn't quite work. It's probably a degenerate layout object, meaning setting a description doesn't cause it to behave like a group with a heading. Maybe outer groups have a different data structure? **An exercise left to the reader**

The flow control for re-display is slightly fighting the history implementation. It would be better if we could retain values while triggering a redraw without even more hacking & state manipulation.


<a id="orgf9b0bbf"></a>

# Appendixes


<a id="org6901288"></a>

## EIEIO - OOP in Elisp

Emacs lisp ships with eieio, a close cousin to the Common Lisp Object System. It's OOP. There are classes & subclasses. You can inherit into new classes and override methods to customize behaviors.

You can use eieio API's to explore transient objects. Let's look at some transients you have already:

```elisp

;; The plist for a prefix command contains a `transient-prefix' object in the
;; `transient--prefix' key and a vector layout in `transient--layout'
;; symbol-plist
(symbol-plist 'magit-dispatch)

;; getting the values from the symbol plist
(get 'magit-dispatch 'transient--prefix)

;; equivalent but longer
;; (plist-get (symbol-plist 'magit-dispatch) 'transient--prefix)

(let ((prefix-object
       (get 'magit-dispatch) 'transient--prefix))

  ;; printing the current slot values for that object
  (object-write prefix-object)

  ;; ;; Object transient-prefix-20997da
  ;; (transient-prefix "transient-prefix-20997da"
  ;;   :command magit-dispatch  :info-manual "(magit)Top")

  ;; getting the class of an object
  (eieio-object-class prefix-object) ; transient-prefix

  ;; opening the help documents for the class, which shows all methods and
  ;; slot forms
  (describe-symbol transient-prefix))

```


### Typical OOP

Like all OOP, the three things you want to do are:

-   Override methods

    `cl-defmethod` and sometimes `cl-call-next-method`

-   Override default values

    Inside the `defclass` form, you can set slots that you don't like. `:initform` is a default value. `:initarg` configures which argument to pick up from the class constructor.

-   Read & Update

    `oref` and `oset`

-   Call Methods

    `(method-name object arguments)`

-   Introspection

    See methods like `slot-boundp` in the EIEIO [eieio method index](info:eieio#Function Index)


### Transient's defclasses and their inheritance

Here's a list of all of transient's `defclass` and their ancestry. This is how it is in 2022.

```elisp

(eieio-browse) ; shows all known classes and their ancestry

;;     +--transient-child
;;     |    +--transient-group
;;     |    |    +--transient-subgroups
;;     |    |    +--transient-columns
;;     |    |    +--transient-row
;;     |    |    +--transient-column
;;     |    +--transient-suffix
;;     |         +--magit--git-submodule-suffix
;;     |         +--transient-infix
;;     |              +--transient-variable
;;     |              |    +--magit--git-variable
;;     |              |    |    +--magit--git-branch:upstream
;;     |              |    |    +--magit--git-variable:urls
;;     |              |    |    +--magit--git-variable:choices
;;     |              |    |         +--magit--git-variable:boolean
;;     |              |    +--transient-lisp-variable
;;     |              +--transient-argument
;;     |                   +--transient-switches
;;     |                   +--transient-option
;;     |                   |    +--transient-files
;;     |                   +--transient-switch
;;     +--transient-prefix
;;          +--magit-log-prefix
;;          |    +--magit-log-refresh-prefix
;;          +--magit-diff-prefix
;;               +--magit-diff-refresh-prefix

```


### View Class Methods and Attributes

Using `describe-symbol` is extremely handy for viewing the class slots and methods.

Classes used in transient that you are likely to want to know the slots for:

[transient-prefix](elisp:(describe-symbol 'transient-prefix)) [transient-suffix](elisp:(describe-symbol 'transient-suffix)) [transient-infix](elisp:(describe-symbol 'transient-infix)) [transient-argument](elisp:(describe-symbol 'transient-argument)) [transient-variable](elisp:(describe-symbol 'transient-variable))

[The eieio docs](https://www.gnu.org/software/emacs/manual/html_mono/eieio.html#Inheritance) have a more wordy treatment. The class system has a lot of behavior that can be faster at times to just understand through description.


<a id="org0382dc1"></a>

## Debugging

There is a lot of support for both print-line and step-through debugging.


### Print debug messages

Just set `transient--debug` to t. [(setq transient-debug t)](elisp:(setq transient--debug t))

You will get a lot of logs visible in `*Messages*` via [view-echo-message-area](elisp:(view-echo-message-area)) the next time you run a transient.

```text

-- setup              (cmd: tsc-layout-rows-explicit, event: "M-x", exit: nil)
-- stack-zap          (cmd: tsc-layout-rows-explicit, event: "M-x", exit: nil)
-- init-transient     (cmd: tsc-layout-rows-explicit, event: "M-x", exit: nil)
push transient--transient-map
push transient--redisplay-map
-- post-command       (cmd: tsc-layout-rows-explicit, event: "M-x", exit: nil)
-- pre-command        (cmd: transient-update, event: "w", exit: nil)
pop  transient--redisplay-map
-- post-command       (cmd: transient-update, event: "w", exit: nil)
pop  transient--redisplay-map
push transient--redisplay-map
-- pre-command        (cmd: tsc-suffix-wave, event: "w l", exit: nil)
-- stack-zap          (cmd: tsc-suffix-wave, event: "w l", exit: nil)
-- pre-exit           (cmd: tsc-suffix-wave, event: "w l", exit: t)
pop  transient--transient-map
pop  transient--redisplay-map
Waves at the user at: Sat Nov 12 22:38:20 2022.
-- post-command       (cmd: tsc-suffix-wave, event: "w l", exit: t)
-- post-exit          (cmd: tsc-suffix-wave, event: "w l", exit: t)

```


### Watching evaluation in Edebug

**Edebug works with transients. There is much support in transient to facilitate using edebug.**

For watching the flow control around your command, especially helpful for debugging behavior around setup, layout, or suffix dispatch, you might want to watch your transient in Edebug.

[Edebug](https://www.youtube.com/watch?v=odkYXXYOxpo) basic introduction video (10 min).

In short:

-   goto your [transient source](elisp:(find-library "transient"))
-   instrument a function you want to watch with `edebug-defun`
-   call the transient / suffix that triggers entry of that function
-   use `SPC` to step forward, `c` to continue, `i` to enter a function call, or `h` for help etc

First watch the debug output to gain an idea of how your code flows with the transient code. Then instrument transient behaviors such as `transient--post-exit` and use `i` to `edebug-step-in` to calls of interest.

When you are done, remember to use [`edebug-remove-instrumentation`](elisp:(edebug-remove-instrumentation)) so that you can go on without every transient you open trying to call the debugger.

-   Debugging Macro Forms

    Because edebug works on defuns while suffixes are defined with macros, you may need to macro exand in order to come up with something debuggable.


<a id="orge18eaeb"></a>

## Layout Hacking

First you need to export the layout data structures.

```elisp

;; Let's look at the layout
(let ((prefix-layout (plist-get
                      (symbol-plist 'magit-dispatch) 'transient--layout)))

  (type-of prefix-layout) ; cons

  (listp prefix-layout) ; t

  (length prefix-layout) ; 3

  ;; Each group in the list is a vector
  (vectorp (car prefix-layout)) ; t

  ;; the attributes are key-value pairs used to create the class
  ;; instance when the transient is shown.

  ;; the nested contents will be lists of vectors for groups and
  ;; lists of lists for suffixes.

  (elt (car prefix-layout) 0) ; first element is a priority
  (elt (car prefix-layout) 1) ; second is a type name
  (elt (car prefix-layout) 2)) ; contents & attributes

;; A sample layout

;; ([1 transient-column nil
;;     ((1 transient-suffix
;;         (:key "i" :description "Ignore" :command magit-gitignore))
;;      (1 transient-suffix
;;         (:key "I" :description "Init" :command magit-init))
;;      (1 transient-suffix
;;         (:key "j" :description "Jump to section"
;;          :command magit-status-jump :if-mode magit-status-mode))
;;      (1 transient-suffix
;;         (:key "j" :description "Display status"
;;          :command magit-status-quick
;;          :if-not-mode magit-status-mode)))])

```

You might find this helpful when constructing [dynamic layouts](#org375878f).


<a id="org16f4cc6"></a>

## Hooks

Just a reminder, some hooks exist. Use `describe-variable` and complete with `transient hook` for the most recent list of hooks.


<a id="org70c4fb4"></a>

## Preludes

Definitions that are not that interesting on their own but are used in examples.


### `tsc-suffix-wave` Command

```elisp

(defun tsc-suffix-wave ()
  "Wave at the user."
  (interactive)
  (message "Waves at the user at: %s." (current-time-string)))

```


### `tsc-suffix-show-level`

```elisp

(transient-define-suffix tsc-suffix-show-level ()
  "Show the current transient's level."
  :transient t
  (interactive)
  (message "Current level: %s" (oref transient-current-prefix level)))

```


### `tsc--define-waver`

```elisp

;; Because command names are used to store and lookup child levels, we have
;; define a macro to generate unqiquely named wavers.  See #153 at
;; https://github.com/magit/transient/issues/153
(defmacro tsc--define-waver (name)
  "Define a new suffix with NAME tsc--wave-NAME."
  `(transient-define-suffix ,(intern (format "tsc--wave-%s" name)) ()
     ,(format "Wave at the user %s" name)
     :transient t
     (interactive)
     (message (format "Waves %s at %s" ,name (current-time-string)))))

;; Each form results in a unique suffix definition.
(tsc--define-waver "surely")
(tsc--define-waver "normally")
(tsc--define-waver "non-essentially")
(tsc--define-waver "definitely")
(tsc--define-waver "eventually")
(tsc--define-waver "hidden")
(tsc--define-waver "inquisitively")
(tsc--define-waver "writingly")
(tsc--define-waver "switchedly")
(tsc--define-waver "unswitchedly")

```


### `tsc-suffix-print-args`

Here's a suffix that reads the transient's infix values, the prefix's scope, and any universal argument (`C-u 4` etc).

```elisp

(transient-define-suffix tsc-suffix-print-args (the-prefix-arg)
  "Report the PREFIX-ARG, prefix's scope, and infix values."
  :transient 'transient--do-call
  (interactive "P")
  (let ((args (transient-args (oref transient-current-prefix command)))
        (scope (transient-scope)))
    (message "prefix-arg: %S \nprefix's scope value: %S \ntransient-args: %S"
             the-prefix-arg scope args)))

;; tsc-suffix-print-args command is incidentally created

```


<a id="orgaba08e8"></a>

## Essential Elisp

If you were hit in the face with the first example, you need to learn basic Elisp. This is not an Elisp guide. Here's some starting points:

-   [transient-define-prefix](info:elisp#Macros) is a macro that creates a command and attaches a `transient-prefix` object to the command symbol's plist.
-   [lambda](info:elisp#Lambda) is a macro to create an anonymous function
-   [interactive](info:elisp#Using Interactive) is a macro that makes the function compatible with the command interface, the `M-x` or `execute-extended-command` menu
-   [The brackets](info:elisp#Vector Functions) are just vector syntax.
    
    Besides the other ways to evaluate elisp used in this README, try `ielm`.
    
    Use the built-in elisp manual by calling the command `elisp-index-search`. See shortdocs for functions using `shortdoc-display-groups`.
    
    The EIEIO and CL manuals are independent from the Elisp manual for some reason. EIEIO is pretty short and not used much once you get the hang of it. `info-display-manual`
    
    [Common Lisp manual](info:cl#Top) you don't really need the common lisp manual for working with transient. Don't be alarmed when you see EIEIO using functions like `cl-call-next-method`


<a id="orgb74feb4"></a>

# Further Reading

-   [**The Transient Manual**](info:transient#Top) ([web link](https://magit.vc/manual/transient.html)) contains more detailed explanation of behavior. The examples here should allow you to visualize what is being described. This guide and the manual should be your first and second sources.

-   [**Transient source**](elisp:(find-library "transient")) ([web link](https://github.com/magit/transient/blob/master/lisp/transient.el)) is all in one file. Source code is always more accurate than manual descriptions, even if some behavior implementations are a bit scattered.

-   [**Magit source**](elisp:(find-library "magit")) ([web link](https://github.com/magit/magit/search?q=transient)) contains numerous examples of transient being used in a big, full-feature application. Search the source for "transient" and you will find many prefixes, suffixes, and custom classes. The smallest examples may be harder to find and most combine many behaviors at once.
