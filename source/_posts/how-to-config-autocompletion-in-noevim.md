---
title: 如何配置NeoVim的自动补全
date: 2023-08-02 16:53:28
tags:
---


## 无需配置的补全

在这一篇中，我们会来讲解一下如何配置NeoVim的自动补全。在开始讲我们的插件之前，我需要先提一嘴，Vim/NeoVim在没有安装任何插件的情况下，也是有自动补全功能的，这个模式叫做：`Insert and Replace（插入和替换）`，具体的文档可以查看 `:h ins-completion`。Vim默认提供了一系列的补全功能，比如你可以补全Buffer中使用过的字符串、文件名等等等。除了这些预设好的补全之外，Vim还提供了一个叫做`omni completion`的功能，默认情况下你可以通过在插入模式下按`Ctrl_x Ctrl_o` 来启动这个补全。正如这个名字暗示的那样，这个补全是个通用补全，你可以设置Vim/NeoVim中的`omnifunc`来向这个补全中添加内容。实际上[neovim/nvim-lspconfig](https://github.com/neovim/nvim-lspconfig)的推荐配置里有这样一句配置：

```lua
vim.bo[ev.buf].omnifunc = 'v:lua.vim.lsp.omnifunc'
```

这句话的意思就是让当前Buffer的`omnifunc`等于lsp提供的`omnifunc`。如果你的配置文件里加上了这句话，你可以试试在你的代码文件中，按下`Ctrl_x Ctrl_o`来启动`omni completion`，你应该可以看到一些最基础的补全了。

## 自动补全

当然这种补全显然不太好用，首先功能非常受限，其次每次补全还需要按一下快捷键，这个着实不太方便，我们需要更好的工具，而这里我们用到的最主要的一个插件就是[nvim-cmp](https://github.com/hrsh7th/nvim-cmp)（之后以CMP代替）。CMP自己的官方定义是一个补全引擎。这个是什么意思呢？补全引擎的意思是，它会整合一系列的补全源（Sources），作为补全结果显示出来，然后你需要去配置，如何确认这些补全结果，在什么时候使用这些补全源，补全源的显示顺序是什么等一系列的事。

所以在安装了CMP之后，你还不能直接使用CMP，你还需要安装一系列的补全源，如果你去看CMP的官方推荐配置的话，它还提到了如下的一系列插件：

```vimscript
Plug 'neovim/nvim-lspconfig'
Plug 'hrsh7th/cmp-nvim-lsp'
Plug 'hrsh7th/cmp-buffer'
Plug 'hrsh7th/cmp-path'
Plug 'hrsh7th/cmp-cmdline'
Plug 'hrsh7th/nvim-cmp'
```

可以这些插件提供的都是补全源，补全源我粗略的分成三类：

1. 基于LSP的补全，这部分部分补全比如函数名，模块名，变量名等功能

2. Snippets的补全，这部分负责给你补全一些预先提供好的代码块，比如一个函数的定义等
3. 一些小内容的补全，这些负责的东西就比较多了，比如文件的路径，Vim的命令，Buffer中的单词等。

## 基础的配置

在有了补全引擎和补全源之后，我们就可以看一些基础的配置，最为基本的CMP配置应该包含如下内容。
```lua
  local cmp = require'cmp'
  cmp.setup({
    snippet = {
      -- REQUIRED - you must specify a snippet engine
      expand = function(args)
        vim.fn["vsnip#anonymous"](args.body) -- For `vsnip` users.
        -- require('luasnip').lsp_expand(args.body) -- For `luasnip` users.
        -- require('snippy').expand_snippet(args.body) -- For `snippy` users.
        -- vim.fn["UltiSnips#Anon"](args.body) -- For `ultisnips` users.
      end,
    },
    mapping = cmp.mapping.preset.insert({
      ['<C-b>'] = cmp.mapping.scroll_docs(-4),
      ['<C-f>'] = cmp.mapping.scroll_docs(4),
      ['<C-Space>'] = cmp.mapping.complete(),
      ['<C-e>'] = cmp.mapping.abort(),
      ['<CR>'] = cmp.mapping.confirm({ select = true }), -- Accept currently selected item. Set `select` to `false` to only confirm explicitly selected items.
    }),
    sources = cmp.config.sources({
      { name = 'nvim_lsp' },
      { name = 'vsnip' }, -- For vsnip users.
      -- { name = 'luasnip' }, -- For luasnip users.
      -- { name = 'ultisnips' }, -- For ultisnips users.
      -- { name = 'snippy' }, -- For snippy users.
    }, {
      { name = 'buffer' },
    })
  })
```

对于一个cmp的基础配置来说，主要有三个部分（其实只有两个）。第一个也是最重要的就是：`sources`。你需要在这里告诉你的cmp这个补全引擎，你需要有哪些补全源，比如常见的`nvim_lsp`，这个补全源的数据是来自LSP，其次还有比如我们之前提到的`buffer`等。你可以自由的排序和分组这些补全源。第二重要的就是`mappings`，这个部分是规定了CMP是如何进行补全操作，在默认情况下，CMP是通过上下键来选择补全内容，然后按回车键进行补全。如果你需要不同的补全风格，可以查看项目[Wiki](https://github.com/hrsh7th/nvim-cmp/wiki/Example-mappings)中提供的其他补全快捷键风格的mappings。其实对于简单的补全来说，你配置完这两部分，已经可以使用了。但是如果你还需要使用snippets的话，还得安装注释启用snippets。这么做的原因是一般情况下snippets也是某种引擎，所以你得让两个引擎能配合使用。

## 针对LSP的配置

为了能使用LSP提供的信息，你需要对你的LSP进行一些额外的配置，让它的信息能暴露给cmp-nvim-lsp使用。你需要告诉cmp-nvim-lsp你的LSP有哪些capabilities。下面是官方的配置：

```lua
-- Set up lspconfig.
local capabilities = require('cmp_nvim_lsp').default_capabilities()
-- Replace <YOUR_LSP_SERVER> with each lsp server you've enabled.
require('lspconfig')['<YOUR_LSP_SERVER>'].setup {
  capabilities = capabilities
}
```

## 额外的补全

在你写完上述的配置之后，你的自动补全应该可以正常运作了，但是如果你想要一些额外的自动补全，比如在你输入某个特定的字符之后才会触发，或者你想要在某个特定的Mode下，才会触发自动补全，你需要进行一些额外的配置，下面是一些来自官方的例子：
```lua
 -- Set configuration for specific filetype.
  cmp.setup.filetype('gitcommit', {
    sources = cmp.config.sources({
      { name = 'git' }, -- You can specify the `git` source if [you were installed it](https://github.com/petertriho/cmp-git).
    }, {
      { name = 'buffer' },
    })
  })

  -- Use buffer source for `/` and `?` (if you enabled `native_menu`, this won't work anymore).
  cmp.setup.cmdline({ '/', '?' }, {
    mapping = cmp.mapping.preset.cmdline(),
    sources = {
      { name = 'buffer' }
    }
  })

  -- Use cmdline & path source for ':' (if you enabled `native_menu`, this won't work anymore).
  cmp.setup.cmdline(':', {
    mapping = cmp.mapping.preset.cmdline(),
    sources = cmp.config.sources({
      { name = 'path' }
    }, {
      { name = 'cmdline' }
    })
  })
```

可以看到上述的配置中，我们针对不同的情况：`filetype`，`cmdline`和不同的触发单词`{ '/', '?' }`、`':'`做了单独的配置。

## 推荐的试验性配置

```lua
experimental = {
  ghost_text = true,
}
```

我推荐大家在`setup`中启用`ghost_text`选项，这个选项会让CMP预先渲染出要补全的内容，非常的方便。
