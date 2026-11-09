# vim-rex-navigator.nvim

Move between Neovim windows and Rex panes with the same
`ctrl+h/j/k/l` keys, like
[vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator) does
for tmux.

It has two halves: a Neovim plugin that hands focus to Rex when you hit the
edge of Neovim, and a small Rex module that hands the keys to Neovim when it's
the focused pane.

## Requirements

- Neovim 0.10 or later
- The `rex` CLI on your `PATH`
- Neovim running inside a Rex pane (`$REX_SESSION` set). Outside Rex the
  plugin only moves between Neovim windows.

## Installation

### Neovim

The plugin has no `setup()` and no default mappings: map the keys to
`require("vim-rex-navigator").navigate(direction)`, where `direction` is
`"left"`, `"down"`, `"up"` or `"right"`. Mapping terminal mode too lets you
leave a `:terminal` window the same way.

With [lazy.nvim](https://github.com/folke/lazy.nvim):

```lua
local function navigate(direction)
    return function()
        require("vim-rex-navigator").navigate(direction)
    end
end

return {
    "gonstoll/vim-rex-navigator.nvim",
    keys = {
        {mode = {"n", "t"}, "<c-h>", navigate("left")},
        {mode = {"n", "t"}, "<c-j>", navigate("down")},
        {mode = {"n", "t"}, "<c-k>", navigate("up")},
        {mode = {"n", "t"}, "<c-l>", navigate("right")},
    },
}
```

With `vim.pack` (Neovim 0.12 or later):

```lua
vim.pack.add({"https://github.com/gonstoll/vim-rex-navigator.nvim"})

for key, direction in pairs({h = "left", j = "down", k = "up", l = "right"}) do
    vim.keymap.set({"n", "t"}, "<c-" .. key .. ">", function()
        require("vim-rex-navigator").navigate(direction)
    end)
end
```

> [!TIP]
> If you also use vim-tmux-navigator, give each plugin the keys only where it
> applies. With lazy.nvim, add `cond = vim.env.TMUX == nil` to this plugin's
> spec and `cond = vim.env.TMUX ~= nil` to vim-tmux-navigator's.

### Rex

Download the Rex module into your Rex config directory:

```sh
curl -fLo ~/.config/rex/vim-rex-navigator.lua \
    https://raw.githubusercontent.com/gonstoll/vim-rex-navigator.nvim/master/vim-rex-navigator.lua
```

Then bind the keys in `~/.config/rex/init.lua`:

```lua
local navigator = require("./vim-rex-navigator")

rex.bind("ctrl+h", navigator.navigate("left"))
rex.bind("ctrl+j", navigator.navigate("down"))
rex.bind("ctrl+k", navigator.navigate("up"))
rex.bind("ctrl+l", navigator.navigate("right"))
```

The module passes the keys through to panes running `nvim`, `vim` or `fzf`, so
they can move inside themselves. To pass them to other programs, add those to
`PASS_THROUGH` at the top of the file.

## How it works

- Away from Neovim's edge, the keys move between Neovim windows as usual.
- At the edge, Neovim asks Rex to focus the neighbouring pane in that
  direction.
- A floating window never hands focus to Rex.
- A zoomed Rex pane keeps the focus, since moving would cancel the zoom.
- Outside Rex, the keys only move between Neovim windows.
- If a `rex` call fails, the plugin shows a warning with `vim.notify`.

## Running tests

```sh
tests/run.sh
```

The tests run Neovim headless against a fake `rex` (`tests/bin/rex`) that
records each call, so you don't need Rex installed.

## License

[MIT](LICENSE)
