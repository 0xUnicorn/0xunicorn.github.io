---
layout: post
title:  "Native LSP config in Neovim V0.11"
date:   2025-03-28
author: Unicorn
tags: Neovim
---

Last time I did a major rewrite of my vim config, was back around 2022 when I changed from vimscript to all lua.
Which was a major improvement for doing small tweaks here and there and making dynamic configurations.

Now a couple of years later, a new version `0.11` has been released with some features I'm really keen about.

- Easier native LSP config
- Builtin auto-completion (cmp) using the `omnifunc`
- Improved the `vim.lsp.buf.hover()` to support markdown tree-sitter highlighting.

In this post I'm gonna go through my config migration from using `nvim-lspconfig` to using the new `vim.lsp.enable` interface.

## Neovim LSP with `nvim-lspconfig`

Neovim added native LSP support back in v0.5, but it never felt like native LSP support as the configuration was
a nightmare, so `nvim-lspconfig` came to the rescue. Making configuration of the builtin LSP very easy and intuitive.

Example of a language server configuration for `vscode-html-language-server`:

```lua
require("lspconfig")["html"].setup{
    on_attach = on_attach,
    flags = lsp_flags,
    capabilities = CAPABILITIES,
    filetypes = { "html" }
}
```

The `on_attach` param is used for executing code during the attachment of a LSP client.
I used this feature for applying different dynamic configs as:

```lua
local function on_attach(client, bufnr)
    lsp_keymaps(bufnr)
    lsp_highlight_document(client)
    lsp_diagnostic_config()
    lsp_icons()
    lsp_handlers()
end
```

## Neovim native LSP configuration

From v0.11 neovim now support two new API interfaces: `vim.lsp.config()` and `vim.lsp.enable()`.
Therefor LSP configuration can now be done 100% independent of nvim-lspconfig, in a very intuitive way.

### How to load language servers

The first thing was figuring out how to do the initial LSP config for language servers.

Reading through the [newsletter](https://gpanders.com/blog/whats-new-in-neovim-0-11/) helped out a lot
with migrating the config.

The new version supports loading language server configurations dynamically by creating a `lsp` folder
in the neovim config root like so: `~/.config/nvim/lsp/`, and adding lua files returning a dictionary.

The former `html` language server config now looks like this:

__neovim/lsp/html.lua:__
```lua
return {
    cmd = { 'vscode-html-language-server', '--stdio' },
    filetypes = { 'html' }
}
```

and for enabling the language server call the new `vim.lsp.enable()` interface from your lua config like so:

__neovim/lua/init.lua:__
```lua
vim.lsp.enable({'html'})
```

or with multiple language servers (a corresponding `lsp/<lang>.lua` is required):

__neovim/lua/init.lua:__
```lua
vim.lsp.enable({
    'ansible',
    'bash',
    'css',
    'html',
    'json',
    'lua',
    'pyright',
    'yaml',
})
```

