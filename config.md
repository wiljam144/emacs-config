
# Table of Contents

1.  [Preliminaries](#orgd304a20)
    1.  [Credentials](#org4d8c11c)
    2.  [Performance](#orgf83b75d)
    3.  [Scratch & Quit Messages](#org4769637)
2.  [Fixing Emacs Defaults](#orge7df693)
    1.  [Basics](#org1289efc)
    2.  [Visuals](#org7377f10)
    3.  [Hooks](#org704d2f6)
3.  [Important Packages](#org710384c)
    1.  [Package Management](#orgf9bbfa2)
    2.  [Evil Mode](#orgb5744e7)
    3.  [Projectile](#org0882698)
    4.  [Language Support](#orgab987f3)
    5.  [Completion](#org78d4bec)
    6.  [Visuals](#org02c71a6)
    7.  [Misc](#org71416cf)
4.  [Keymaps](#org7391378)
5.  [RSS & Mail](#orgfe8882c)
6.  [Ligatures](#orgc67d950)



<a id="orgd304a20"></a>

# Preliminaries

    ;; -*- coding: utf-8; lexical-binding: t -*-


<a id="org4d8c11c"></a>

## Credentials

Credentials are placed in separate files, so that they don't end up in public GitHub repo
accidentally.

    (load "~/.config/emacs/credentials.el")


<a id="orgf83b75d"></a>

## Performance

To squeeze out some performance.

    (setq gc-cons-threshold 100000000)
    (setq max-specpdl-size 5000)


<a id="org4769637"></a>

## Scratch & Quit Messages

    (defun wl/random-choice (items)
      (nth (random (length items)) items))
    
    (setq scratch-messages '(";; Approved by The Alphabet Bois\n"
                             ";; NSA Inside\n"
                             ";; Not Copyrighted by Anyone 1432-1785\n"
                             ";; NSA Controlled Since 1952\n"
                             ";; Spyware Embedded Deep Inside\n"
                             ";; UwU\n"
                             ";; <3\n"
                             ";; Quicksort is Quick\n"
                             ";; I Have a Smooth Brain\n"
                             ";; I Probably Have Bipolar Disorder\n"))
    
    (setq quit-messages '("You will regret this. "
                          "Come on, do it, see if I care. "
                          "Get out of here and go back to your boring programs. "))
    
    (setq initial-scratch-message (wl/random-choice scratch-messages))
    
    (add-hook 'kill-emacs-query-functions
              (lambda () (y-or-n-p (wl/random-choice quit-messages)))
              'append)


<a id="orge7df693"></a>

# Fixing Emacs Defaults


<a id="org1289efc"></a>

## Basics

Most Emacs defaults are pretty weird so let's fix the most basic ones.

    (setq
     inhibit-startup-screen t
     intial-scratch-message nil
     sentence-end-double-space nil
     ring-bell-function 'ignore
     save-interprogram-paste-before-kill t
     use-dialog-box nil
     use-short-answers nil
     load-prefer-newer t
     confirm-kill-processes nil
     native-comp-async-report-warnings-errors 'silent
     help-window-select t
     delete-by-moving-to-trash t
     scroll-preserve-screen-position t
     completions-detailed t
     next-error-message-highlight t
     read-minibuffer-restore-windows t)
    
    (setq-default indent-tabs-mode nil)
    
    (setq custom-file (make-temp-name "/tmp/"))
    
    (delete-selection-mode t)
    (savehist-mode)

Emacs supports a lot of formats, but unicode and utf-8 should be the default.

    (set-charset-priority 'unicode)
    (prefer-coding-system 'utf-8-unix)

Emacs loves to create a lot of useless backup files.

    (setq
     make-backup-files nil
     auto-save-default nil
     create-lockfiles nil)

It also has some pretty weird scrolling by default in my opinion.

    (setq
     scroll-conservatively 101
     scroll-up-aggressively 0.01
     scroll-down-aggressively 0.01
     auto-window-vscroll nil)

I also like to get the second pair automatically

    (setq electric-pair-preserve-balance t
          electric-pair-open-newline-between-pairs t
          electric-pair-delete-adjacent-pairs t)
    
    (electric-pair-mode 1)


<a id="org7377f10"></a>

## Visuals

Let's make Emacs visuals a bit more modern and informative.

To start off, Let's get rid of those useless toolbars.

    (toggle-frame-maximized)
    (menu-bar-mode -1)
    (tool-bar-mode -1)
    (scroll-bar-mode -1)

Now let's get a nice font.

    (add-to-list 'default-frame-alist '(font . "Fira Code 11"))
    (set-face-attribute 'default t :font "Fira Code 11")

I think highlight line is pretty useful.

    (require 'hl-line)
    (add-hook 'prog-mode-hook #'hl-line-mode)
    (add-hook 'text-mode-hook #'hl-line-mode)

Apart from that I like to have a ruler that shows me where 80 characters are.

    (set-face-foreground 'fill-column-indicator "#202020")
    (setq display-fill-column-indicator-column 79)
    (global-display-fill-column-indicator-mode 1)

And of course line numbers.

    (global-display-line-numbers-mode 1)
    (setq display-line-numbers-type 'relative)
    
    (defun wl/disable-line-numbers ()
      (display-line-numbers-mode -1)
      (display-fill-column-indicator-mode -1))

Indent guides are also useful for me. I had troubles initially, because the face foreground
wasn't setting up correctly, it turns out, I just had to disable the `highlight-guides-indent-guides-auto-enabled` to `nil`.

    (use-package highlight-indent-guides
          :ensure t
          :hook
          ((prog-mode text-mode conf-mode) . highlight-indent-guides-mode)
          :init
          (setq highlight-indent-guides-method 'character
                highlight-indent-guides-auto-enabled nil
                highlight-indent-guides-responsive 'top)
          :config
          (set-face-foreground 'highlight-indent-guides-character-face "#303030")
          (set-face-foreground 'highlight-indent-guides-top-character-face "#707070"))
    
    (add-hook 'after-make-frame-functions #'highlight-indent-guides-auto-set-faces)
    
    (highlight-indent-guides-mode t)


<a id="org704d2f6"></a>

## Hooks

    (add-hook 'text-mode-hook #'wl/disable-line-numbers)
    (add-hook 'before-save-hook #'delete-trailing-whitespace)


<a id="org710384c"></a>

# Important Packages


<a id="orgf9bbfa2"></a>

## Package Management

I use `use-package` as my package manager.

    (require 'package)
    (add-to-list 'package-archives
                 '("melpa" . "https://melpa.org/packages/"))
    (package-initialize)
    
    (require 'use-package)


<a id="orgb5744e7"></a>

## Evil Mode

Probably the most important part of this config.
I used Vim/Neovim before I switched to Emacs and because of that,
Vim bindings are carved into my brain permamently until the end of my days.

    (use-package evil
      :ensure t
      :init
      (setq evil-want-integration t)
      (setq evil-want-keybinding nil)
      :config
      (evil-mode))
    (use-package evil-collection
      :after evil
      :ensure t
      :config
      (evil-collection-init))


<a id="org0882698"></a>

## Projectile

    (use-package projectile
      :ensure t
      :config
      (projectile-mode +1))


<a id="orgab987f3"></a>

## Language Support


### Syntax Highlighting

Default Emacs highlighting is fine in most languages, but
`TreeSitter` is just miles better so why not use it?

    (use-package tree-sitter
      :ensure t
      :config
      (global-tree-sitter-mode)
      (add-hook 'tree-sitter-after-on-hook #'tree-sitter-hl-mode))
    (use-package tree-sitter-langs
      :ensure t)


### LSP

I use `eglot` as my LSP client.

    (use-package eglot
      :ensure t
      :hook
      ;; Do I need to tell you for what languages these are?
      (c-mode . eglot-ensure)
      (c++-mode . eglot-ensure)
      (python-mode . eglot-ensure)
    
      ;; OCaml Support
      (tuareg-mode . eglot-ensure))

I hate the inlay hints, they are a useless distraction that annoys me.

    (defun wl/eglot-config ()
      (eglot-inlay-hints-mode -1)
      (setq
       eglot-ignored-server-capabilities '(:documentHighlightProvider)
       eldoc-echo-area-use-multiline-p nil))
    
    (add-hook 'eglot-managed-mode-hook #'wl/eglot-config)


### Autocomplete

I like to have completion because I don't really like typing.
I use `company` to get autocomplete in my code.

    (use-package company
      :ensure t
      :hook
      (after-init . global-company-mode))


### Language Support

Here are some packages that provide major modes not included in emacs by default.

    (use-package tuareg
      :ensure t)
    
    (use-package markdown-mode
      :ensure t
      :init
      (setq markdown-command "multimarkdown"))


<a id="org78d4bec"></a>

## Completion

The orderless completion style is much better than the default one.

    (use-package orderless
      :ensure t
      :init
      (setq completion-styles '(orderless basic)
            completion-category-defaults nil
            completion-category-overrides '((file (styles partial-completion)))))

I also use vertico to get completions.

    (use-package vertico
      :ensure t
      :init
      (vertico-mode))


<a id="org02c71a6"></a>

## Visuals

I like the JetBrains Fleet colorsheme.

    (use-package fleetish-theme
      :ensure t
      :config
      (load-theme 'fleetish t))

The default modeline in Emacs is pretty boring and ugly.
Tha's why I use `doom-modeline`

    (use-package doom-modeline
      :ensure t
      :init (doom-modeline-mode 1))


<a id="org71416cf"></a>

## Misc

`which-key` is extremely helpful in discovering emacs functionality.

    (use-package which-key
      :ensure t
      :config
      (which-key-mode))

Editorconfig is a wonderful thing. It setups my editor automatically
to match the project requirements.

    (use-package editorconfig
      :ensure t
      :config
      (editorconfig-mode 1))

`vterm` is a great terminal that can be opened from Emacs

    (use-package elfeed
      :ensure t
      :config
      (setq elfeed-feeds
            '("https://wiljam144.github.io/digital-garden/feed.rss")))


<a id="org7391378"></a>

# Keymaps

    (defun wl/keymap (key fun)
      (define-key evil-normal-state-map (kbd key) fun))
    
    (wl/keymap "SPC p" 'projectile-command-map)
    (wl/keymap "SPC SPC" 'projectile-find-file)
    (wl/keymap "SPC e" 'mu4e)
    (wl/keymap "SPC `" 'vterm)


<a id="orgfe8882c"></a>

# RSS & Mail

I use elfeed to get RSS.

    (use-package elfeed
      :ensure t
      :config
      (setq elfeed-feeds
            ;; my own digital garden site feed
            '("https://wiljam144.github.io/digital-garden/rss.xml")))

Mail in Emacs is pretty convenient. I use `mu4e` for that.

    (require 'mu4e)
    
    (add-hook 'mu4e-compose-pre-hook (lambda () (display-line-numbers-mode -1)))
    (setq mail-user-agent 'mu4e-user-agent)
    (setq mu4e-drafts-folder "/[Gmail].Drafts")
    (setq mu4e-sent-folder "/[Gmail].Sent Mail")
    (setq mu4e-trash-folder "/[Gmail].Trash")
    (setq mu4e-attachment-dir "~/Downloads")
    
    (setq mu4e-sent-messages-behavior 'delete)
    
    (setq mu4e-get-mail-command "offlineimap")
    (setq mu4e-update-interval (* 10 60 60))
    
    (setq user-mail-address wl/mail)
    
    (setq
        user-mail-address wl/mail
        user-full-name wl/name
        mu4e-compose-signature
        (concat
         wl/name
         "\n"
         wl/website))

For sending mail `smtpmail` needs to be used.

    (require 'smtpmail)
    (setq message-send-mail-function 'smtpmail-send-it
          starttls-use-gnutls t
          smtpmail-starttls-credentials '(("smtp.gmail.com" 465 nil nil))
          smtpmail-auth-credentials
            '(("smtp.gmail.com" 465 nil nil))
          smtpmail-stream-type 'ssl
          smtpmail-default-smtp-server "smtp.gmail.com"
          smtpmail-smtp-server "smtp.gmail.com"
          smtpmail-smtp-service 465
          smtpmail-debug-info t)


<a id="orgc67d950"></a>

# Ligatures

    (use-package ligature
      :ensure t
      :config
      ;; Enable the "www" ligature in every possible major mode
      (ligature-set-ligatures 't '("www"))
      ;; Enable traditional ligature support in eww-mode, if the
      ;; `variable-pitch' face supports it
      (ligature-set-ligatures 'eww-mode '("ff" "fi" "ffi"))
      ;; Enable all Cascadia and Fira Code ligatures in programming modes
      (ligature-set-ligatures 'prog-mode
                            '(;; == === ==== => =| =>>=>=|=>==>> ==< =/=//=// =~
                              ;; =:= =!=
                              ("=" (rx (+ (or ">" "<" "|" "/" "~" ":" "!" "="))))
                              ;; ;; ;;;
                              (";" (rx (+ ";")))
                              ;; && &&&
                              ("&" (rx (+ "&")))
                              ;; !! !!! !. !: !!. != !== !~
                              ("!" (rx (+ (or "=" "!" "\." ":" "~"))))
                              ;; ?? ??? ?:  ?=  ?.
                              ("?" (rx (or ":" "=" "\." (+ "?"))))
                              ;; %% %%%
                              ("%" (rx (+ "%")))
                              ;; |> ||> |||> ||||> |] |} || ||| |-> ||-||
                              ;; |->>-||-<<-| |- |== ||=||
                              ;; |==>>==<<==<=>==//==/=!==:===>
                              ("|" (rx (+ (or ">" "<" "|" "/" ":" "!" "}" "\]"
                                              "-" "=" ))))
                              ;; \\ \\\ \/
                              ("\\" (rx (or "/" (+ "\\"))))
                              ;; ++ +++ ++++ +>
                              ("+" (rx (or ">" (+ "+"))))
                              ;; :: ::: :::: :> :< := :// ::=
                              (":" (rx (or ">" "<" "=" "//" ":=" (+ ":"))))
                              ;; // /// //// /\ /* /> /===:===!=//===>>==>==/
                              ("/" (rx (+ (or ">"  "<" "|" "/" "\\" "\*" ":" "!"
                                              "="))))
                              ;; .. ... .... .= .- .? ..= ..<
                              ("\." (rx (or "=" "-" "\?" "\.=" "\.<" (+ "\."))))
                              ;; -- --- ---- -~ -> ->> -| -|->-->>->--<<-|
                              ("-" (rx (+ (or ">" "<" "|" "~" "-"))))
                              ;; *> */ *)  ** *** ****
                              ("*" (rx (or ">" "/" ")" (+ "*"))))
                              ;; www wwww
                              ("w" (rx (+ "w")))
                              ;; <> <!-- <|> <: <~ <~> <~~ <+ <* <$ </  <+> <*>
                              ;; <$> </> <|  <||  <||| <|||| <- <-| <-<<-|-> <->>
                              ;; <<-> <= <=> <<==<<==>=|=>==/==//=!==:=>
                              ;; << <<< <<<<
                              ("<" (rx (+ (or "\+" "\*" "\$" "<" ">" ":" "~"  "!"
                                              "-"  "/" "|" "="))))
                              ;; >: >- >>- >--|-> >>-|-> >= >== >>== >=|=:=>>
                              ;; >> >>> >>>>
                              (">" (rx (+ (or ">" "<" "|" "/" ":" "=" "-"))))
                              ;; #: #= #! #( #? #[ #{ #_ #_( ## ### #####
                              ("#" (rx (or ":" "=" "!" "(" "\?" "\[" "{" "_(" "_"
                                           (+ "#"))))
                              ;; ~~ ~~~ ~=  ~-  ~@ ~> ~~>
                              ("~" (rx (or ">" "=" "-" "@" "~>" (+ "~"))))
                              ;; __ ___ ____ _|_ __|____|_
                              ("_" (rx (+ (or "_" "|"))))
                              ;; Fira code: 0xFF 0x12
                              ("0" (rx (and "x" (+ (in "A-F" "a-f" "0-9")))))
                              ;; Fira code:
                              "Fl"  "Tl"  "fi"  "fj"  "fl"  "ft"
                              ;; The few not covered by the regexps.
                              "{|"  "[|"  "]#"  "(*"  "}#"  "$>"  "^="))
      ;; Enables ligature checks globally in all buffers. You can also do it
      ;; per mode with `ligature-mode'.
      (global-ligature-mode t))

