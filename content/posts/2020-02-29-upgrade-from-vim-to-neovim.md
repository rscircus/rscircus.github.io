---
categories:
- Code
date: "2020-02-29T00:00:00Z"
excerpt_separator: <!-- more -->
sub_title: Setting up Neovim on Ubuntu/Linux
tags:
- vim
- neovim
- config
title: Upgrade from vim to Neovim
---

How to perform an easy transition from vim to neovim? The step had to come as vim's code base is a mess as many have written already. These days evil-mode in Emacs felt even better than the early love. Let's get on it and enjoy asynchronous job control and lua scripting.

<!--more-->

Neovim's advantages over vim:

- asynchronous job control
- asynchronous job control (yepp, this is intentional)
- (mostly) compatible with vim pluggins
- embedded terminal emulator
- and [many more](https://github.com/neovim/neovim#features)...

## Installation on Ubuntu

Get the unstable neovim (I need a higher than Ubuntu 18.04 batteries-included NVIM v0.2.2 version) ppa:

```bash
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt-get update
sudo apt-get install neovim
```

For instance [vim-go](https://github.com/fatih/vim-go) requires at least `NVIM v0.3.1`.

Assuming you are using the fantastic plugin manager [vim-plug](https://github.com/junegunn/vim-plug), let's get vim almost to the migration done mark:


## Configuration

Copy your configuration over to the neovim standard directories. Most things will work.

```bash
cp -r ~/.vim/plugged ~/.config/nvim/plugged/
cp -r ~/.vim/autoload ~/.config/nvim/autoload/
cp ~/.vim/.vimrc ~/.config/nvim/init.vim                   # or wherever your vimrc resides
```

If you want to keep the *same* config for both, vim and nvim, use this in your `~/.config/nvim/init.vim`:

```bash
set runtimepath^=~/.vim runtimepath+=~/.vim/after
let &packpath = &runtimepath
source ~/.vimrc
```

## Getting Started

Get started with:

```
:help vim-differences
```

PS: After I read the above, I decided to start with a blank `init.vim` and discarded the steps above to do a major clean up and deprecate my `.vimrc`. 🤷
