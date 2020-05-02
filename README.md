![Nvim + Metals](https://i.imgur.com/UvQ18ST.png)

# nvim-metals

This is a WIP [Metals](https://scalameta.org/metals/) plugin and guide for Nvim
usere utilizing the [Nvim built-in LSP
module](https://neovim.io/doc/user/lsp.html).

Keep in mind that the level of support is rapidly changing, there are bugs, and
there are missing features. Some of this is changing daily, so expect stuff to
break without warning or change.

If you're first getting starting with Metals, consider using
[coc-metals](https://github.com/scalameta/coc-metals) if you're looking for a
more feature-full and stable Metals + Nvim experience.

## Table of Contents
  - [Prerequisites](#prerequisites)
      - [Plugins](#plugins)
  - [Getting Started](#getting-started)
  - [Available Commands](#available-commands)
  - [Settings and Mappings](#settings-and-mappings)
  - [Complementary Plugins](#complementary-plugins)
      - [Completions](#completions)
      - [Diagnostics](#diagnostics)
  - [Importing Your Build](#importing-your-build)
  - [Known Limitations](#known-limitations)
  - [Roadmap](#roadmap)

## Prerequisites

Before you get started you need to ensure that you have the nighly/development
build. LSP support hasn't landed in stable yet. You can find instructions for
how to do this for you OS
[here](https://github.com/neovim/neovim/wiki/Installing-Neovim). It's best to
re-build often as LSP support is changing daily. The easiest way to ensure
you're on nightly is to to do a `nvim --version`. If you see anything `v0.4.x`
then it didn't work. You're looking for `v0.5.x`:

```vim
❯ nvim --version
NVIM v0.5.0-3de9452
...
```

### Plugins

Apart from this plugin, you'll also want to have the
[neovim/nvim-lsp](https://github.com/neovim/nvim-lsp) plugin installed. For now,
this plugin offers automated installation and some default configurations for
Metals.

Use whichever plugin manager to install both.
[vim-plug](https://github.com/junegunn/vim-plug) is probably the most common, so
I'll use that as an example:

```vim
call plug#begin('~/.vim/plugged')
  " Main necessary plugins
  Plug 'neovim/nvim-lsp'
  Plug 'ckipp01/nvim-metals'
  " Complementary plugins that I recommend and will go over down below
  Plug 'haorenW1025/completion-nvim'
  Plug 'haorenW1025/diagnostic-nvim'
call plug#end()
```

## Getting started

First things first, you need to install Metals. This functionality is provided
by the [nvim-lsp](https://github.com/neovim/nvim-lsp) plugin. It offers
automated installation of servers and basic configurations so you don't have to
do it manually.

Currently, you need the latest SNAPSHOT of Metals in order for this to work
correctly. You can find the latest SNAPSHOT version [here on the
website](https://scalameta.org/metals/docs/editors/vim.html#using-latest-metals-snapshot).

```vim
let g:metals_server_version = '0.8.4+106-5f2b9350-SNAPSHOT'
```

```vim
:LspInstall metals
```

There isn't a lot of feedback on whether or not this worked, so after you do
this, issue the following command to ensure that it's installed. This will also
show you the directory that it's installed in.

```vim
:LspInstallInfo
```

If it's installed, you should see something like the following:

```vim
{
  metals = {
    cmd = { "/Users/ckipp/.cache/nvim/nvim_lsp/metals/metals" },
    install_dir = "/Users/ckipp/.cache/nvim/nvim_lsp/metals",
    is_installed = "file"
  }
}
```

Make sure to take a look at the [`setup()`
function](https://github.com/neovim/nvim-lsp#setup-function) which will show you
how to override certain values. You can see all of the default Metals values in
the [readme](https://github.com/neovim/nvim-lsp#metals) or checkout
[nvim-lsp/lua/nvim_lsp/metals.lua](https://github.com/neovim/nvim-lsp/blob/master/lua/nvim_lsp/metals.lua).

If you don't want any of the extra stuff the other plugins offer, then just copy
the mappings under the **nvim-lsp Mappings** section to get the mappings, add in
the `autocmd` to get you completions (that you'll need to trigger yourself
without the plugin), and then copy the **lua callbacks** chunk of lua code minus
the things I'll point out below which are plugin specific:

```lua
:lua << EOF
  local nvim_lsp = require'nvim_lsp'
  local M = {}
  M.on_attach = function()
      require'diagnostic'.on_attach() -- needed for the diagnostic plugin
      require'completion'.on_attach() -- needed for the completion plugin
    end
  nvim_lsp.metals.setup{
    on_attach = M.on_attach, -- Don't include this if you aren't using the other plugins
  }
EOF
```

**Fair warning, this is probably going to change.**
If you follow the conversation
[here](https://github.com/neovim/nvim-lsp/issues/200), you'll notice a couple
things.

1. There is an automated way to install, but not uninstall or update
2. The install feature will probably go away

For now, this is still the best way to install Metals for Nvim.  If the Install
goes away, there is a decent chance I'll handle the Install / Uninstall / Update
right in the plugin.

## Available Commands

Currently, the following commands are available:

Command           |Description
------------------|-------------------------------------
`:BuildImport`    | Import the build
`:BuildConnect`   | Manually connect to the build server
`:BuildRestart`   | Restart the build server
`:CompileCascade` | Compile current open file along with all build targets that depend on it
`:Format`         | Format current buffer
`:MetalsDoctor`   | Run Metals Doctor, which will open in your browser
`:SourcesScan`    | Scan all workspace sources

## Settings and Mappings

Some very basic things are enabled by default, like inline diagnostics, but
you'll want a basic configuration for things like finding definitions and
references. The [nvim-lsp.vim](./nvim-lsp.vim) file has these in along with some
settings for the other complementary plugins. _These are opinionated_, and
catered to my work flow. There are also a two other plugin settings under
`completion-nvim` and `diagnostic-nvim` headings. Those plugins, which are
outlined below must also be installed for those settings to work. The The idea
is to use them as a base or an example and to then build off of them or change
them to your liking. The also serve as an example of how to use `lua` in your
configuration if you're not familiar with them. They also have a few vim
configurations that I'd argue are important for you to not go insane (like
having `set signcolumn=yes`). Again, edit this to your liking. They are just in
here as an example.

```vim
nnoremap <silent> gd <cmd>lua vim.lsp.buf.definition()<CR>
nnoremap <silent> K  <cmd>lua vim.lsp.buf.hover()<CR>
```

You can sort of assume what many of the mappings do, but you can find more info
on the available options by doing `:h lsp-config`. There is a lot of great info
in the `lsp` section of the docs, but also keep in mind that things are often
changing and some sections are just bare with `TODOs`.

The mappings I've included are a combination of what Metals supports and also
what Nvim LSP supports.

## Complementary Plugins

I've listed two external plugins in the examples that help the diagnostic and
completion experience. The Nvim LSP integration exposes a bunch of call-backs
that allow for easy customization. As far as I know the idea isn't to make an
extremely polished experience out of the box with Nvim LSP, but rather offer a
solid core that allows you to define custom callbacks for how you want to handle
these things. The idea is that this plugin will offer you all the necessary
Metals specific settings, while also allowing other plugins to offer the thing
they do best. This follows the [Unix
philosophy](https://en.wikipedia.org/wiki/Unix_philosophy) of tools doing one
thing, doing them well, and interfacing well with others.

### Completions

Taken from the docs:

> Nvim provides the _vim.lsp.omnifunc_ 'omnifunc' handler which allows
_i_CTRL-X_CTRL-O_ to consume LSP completion. Example config (note the use of
_v:lua_ to call Lua from Vimscript):

```vim
" Use LSP omni-completion in Scala files.
autocmd Filetype scala setlocal omnifunc=v:lua.vim.lsp.omnifunc
```

This will give you completion in Scala files, but you'd need to trigger them
using `i_CTRL-X_CTRL-O`, which you may not want. This is why I include the
following plugin:

- [completion-nvim](https://github.com/haorenW1025/completion-nvim)

```vim
Plug 'haorenW1025/completion-nvim'
```
This plugin will give you completion automatically in the floating window as
your typing. I have a few other settings listed which allow you to use `<Tab>`
to navigate the popup menu.

### Diagnostics

The diagnostics plugin I include mainly allows for easy settings and
customization for how your diagnostics are displayed. For example you can delay
the syntactic diagnostics that Metals provides while typing when you're in
insert mode. You can choose to display them as virtual text or not. Read through
the docs to get an idea of all the options.

- [diagnostic-nvim](https://github.com/haorenW1025/diagnostic-nvim)

```vim
Plug 'haorenW1025/diagnostic-nvim'
```

**Keep in mind that both of these plugins are under active development as well
and things are likely to change**.


## Importing your build

Since `window/showMessageRequest` is not yet supported in the nvim LSP module,
you need to trigger this manually. As you would normally, open your project and
then issue a `:BuildImport` command which will send the request to Metals to
import your build.

## Known limitations

- There is no `window/showMessageRequest` so you'll never get prompted to import
    your build. There is an issue for this here: https://github.com/neovim/neovim/issues/11710
- Renames aren't working correctly since Metals isn't versioning the Documents.
    You can track the Metals part of this here:
    https://github.com/scalameta/metals/issues/1668 and the Nvim side of this
    here: https://github.com/neovim/neovim/issues/12226 since they should account
    for this being `null`. Fair warning, I sent in a pr to fix this and botched
    it... so this is the second attempt.
- Multiline `textEdits` aren't being applied correctly. You can track this
    issue here: https://github.com/neovim/neovim/issues/12195
- Using the completions sometimes cause the window to move off to the side of
    your terminal and show an error. You can track this here: https://github.com/haorenW1025/completion-nvim/issues/53

##### Roadmap

- [ ] Status line displaying diagnostics
- [ ] Write up reasoning behind moving more towards the built-in lsp supported
- [ ] Figure out how to add the `New Scala File` command
- [ ] Is there a way to get `metals/status` to works?
- [ ] Use virtual text for worksheets
