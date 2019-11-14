# nvim-common-lsp

WIP Common configurations for Language Servers.

This repository aims to be a central location to store configurations for
Language Servers which leverages Neovim's built-in LSP client `vim.lsp` for the
client backbone. The `vim.lsp` implementation is made to be customizable and
greatly extensible, but most users just want to get up and going. This
plugin/library is for those people, although it still let's you customize
things as much as you want in addition to the defaults that this provides.

**NOTE**: Requires current Neovim master as of 2019-11-13

**CONTRIBUTIONS ARE WELCOME!**

There's a lot of language servers in the world, and not enough time.  See
[`lua/common_lsp/texlab.lua`](https://github.com/norcalli/nvim-common-lsp/blob/master/lua/common_lsp/texlab.lua)
and
[`lua/common_lsp/skeleton.lua`](https://github.com/norcalli/nvim-common-lsp/blob/master/lua/common_lsp/skeleton.lua)
for examples and ask me questions in the [Neovim
Gitter](https://gitter.im/neovim/neovim) to help me complete configurations for
*all the LSPs!*

If you don't know where to start, you can pick one that's not in progress or
implemented from [this excellent list compiled by the coc.nvim
contributors](https://github.com/neoclide/coc.nvim/wiki/Language-servers) or
[this other excellent list from the emacs lsp-mode
contributors](https://github.com/emacs-lsp/lsp-mode#supported-languages)
and create a new file under `lua/common_lsp/SERVER_NAME.lua`.
- For a simple server which should only ever have one instance for the entire
neovim lifetime, I recommend copying `lua/common_lsp/texlab.lua`.
- For servers which should have a different instance for each project root, I
recommend copying `lua/common_lsp/gopls.lua`.

## Progress

Implemented:
- [gopls](https://github.com/norcalli/nvim-common-lsp#gopls) (has some errors)
- [texlab](https://github.com/norcalli/nvim-common-lsp#texlab)

Planned servers to implement (by me, but contributions welcome anyway):
- [clangd](https://clang.llvm.org/extra/clangd/Installation.html)
- [ccls](https://github.com/MaskRay/ccls)
- [lua-language-server](https://github.com/sumneko/lua-language-server)
- [rust-analyzer](https://github.com/rust-analyzer/rust-analyzer)

In progress:
- ...

## Install

`Plug 'norcalli/nvim-common-lsp'`

## Use

From vim:
```vim
call common_lsp#texlab({})
call common_lsp#gopls({})

" These are still TODO, but will be done.
call common_lsp#clangd({})
call common_lsp#ccls({})
call common_lsp#tsserver({})

" Or using a dynamic name.
call common_lsp#setup("texlab", {})
call common_lsp#setup("gopls", {})
```

From Lua:
```lua
require 'common_lsp'.texlab.setup {
  name = "texlab_fancy";
  log_level = vim.lsp.protocol.MessageType.Log;
  settings = {
    latex = {
      build = {
        onSave = true;
      }
    }
  }
}

local common_lsp = require 'common_lsp'

-- Customize how to find the root_dir
common_lsp.gopls.setup {
  root_dir = common_lsp.util.root_pattern(".git");
}

-- Build the current buffer.
require 'common_lsp'.texlab.buf_build(0)
```

```
These are functions to set up servers more easily with some server specific
defaults and more server specific things like commands or different
diagnostics.

Servers may define extra functions on the `common_lsp.SERVER` table, e.g.
`common_lsp.texlab.buf_build({bufnr})`.

The main setup signature will be:

common_lsp.SERVER.setup({config})

  {config} is the same as |vim.lsp.start_client()|, but with some
  additions and changes:

  {root_dir}
    May be required (depending on the server).
    `function(filename, bufnr)` which is called on new candidate buffers to
    attach to and returns either a root_dir or nil.

    If a root_dir is returned, then this file will also be attached. You
    can optionally use {filetype} to help pre-filter by filetype.

    If a root_dir is returned which is unique from any previously returned
    root_dir, a new server will be spawned with that root_dir.

    If nil is returned, the buffer is skipped.

    See |common_lsp.util.search_ancestors()| and the functions which use it:
    - |common_lsp.util.root_pattern(patterns...)| finds an ancestor which
    - contains one of the files in `patterns...`. This is equivalent
    to coc.nvim's "rootPatterns"
    - More specific utilities:
      - |common_lsp.util.find_git_root()|
      - |common_lsp.util.find_node_modules_root()|
      - |common_lsp.util.find_package_json_root()|

  {name}
    Defaults to the server's name.

  {filetypes}
    A set of filetypes to filter for consideration by {root_dir}.
    Can be left empty.
    A server may specify a default value.

  {log_level}
    controls the level of logs to show from build processes and other
    window/logMessage events. By default it is set to
    vim.lsp.protocol.MessageType.Warning instead of
    vim.lsp.protocol.MessageType.Log.

  {settings}
    This is a table, and the keys are case sensitive. This is for the
    window/configuration event responses.
    Example: `settings = { keyName = { subKey = 1 } }`

  {on_attach}
    `function(client)` will be executed with the current buffer as the
    one the {client} is being attaching to. This is different from
    |vim.lsp.start_client()|'s on_attach parameter, which passes the {bufnr} as
    the second parameter instead. This is useful for running buffer local
    commands.

  {on_new_config}
    `function(new_config)` will be executed after a new configuration has been
    created as a result of {root_dir} returning a unique value. You can use this
    as an opportunity to further modify the new_config or use it before it is
    sent to |vim.lsp.start_client()|.
```

# LSP Implementations

## clangd

https://clang.llvm.org/extra/clangd/Installation.html

clangd relies on a [JSON compilation database](https://clang.llvm.org/docs/JSONCompilationDatabase.html) specified
as compile_commands.json or, for simpler projects, a compile_flags.txt.



common_lsp.clangd.setup({config})
common_lsp#setup("clangd", {config})

```
  Default Values:
    capabilities = default capabilities, with offsetEncoding utf-8
    cmd = { "clangd", "--background-index" }
    filetypes = { "c", "cpp", "objc", "objcpp" }
    log_level = 2
    on_init = function to handle changing offsetEncoding
    root_dir = root_pattern("compile_commands.json", "compile_flags.txt", ".git")
    settings = {}
```
## texlab

https://texlab.netlify.com/

A completion engine built from scratch for (la)tex.



common_lsp.texlab.setup({config})
common_lsp#setup("texlab", {config})

```
  Commands:
  - TexlabBuild: Build the current buffer
  
  Default Values:
    cmd = { "texlab" }
    filetypes = { "tex", "bib" }
    log_level = 2
    root_dir = vim's starting directory
    settings = {
      latex = {
        build = {
          args = { "-pdf", "-interaction=nonstopmode", "-synctex=1" },
          executable = "latexmk",
          onSave = false
        }
      }
    }
```
## gopls

https://github.com/golang/tools/tree/master/gopls

Google's lsp server for golang.



common_lsp.gopls.setup({config})
common_lsp#setup("gopls", {config})

```
  Default Values:
    cmd = { "gopls" }
    filetypes = { "go" }
    log_level = 2
    root_dir = root_pattern("go.mod", ".git")
    settings = {}
```
