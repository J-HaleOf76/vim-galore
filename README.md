![vim-galore](https://raw.githubusercontent.com/mhinz/vim-galore/master/pics/vim-galore.png)

---

#### [Basics](#basics-1)
- [Buffers, windows, tabs?](#buffers-windows-tabs)
- [Active, loaded, listed, named buffers?](#active-loaded-listed-named-buffers)
- [Colorschemes?](#colorschemes)

#### [Usage](#usage-1)
- [Managing plugins](#managing-plugins)

#### [Debugging](#debugging-1)
- [General tips](#general-tips)
- [Profiling startup time](#profiling-startup-time)
- [Profiling at runtime](#profiling-at-runtime)

#### [List of colorschemes](#list-of-colorschemes-1)

#### [List of plugins](#list-of-plugins-1)
- [Fuzzy finders](#fuzzy-finders)
- [Version control](#version-control)

---

## Basics

#### Buffers, windows, tabs?

Vim is a text editor. Everytime text is shown, the text is part of a **buffer**.
Each file will be opened in its own buffer. Plugins show stuff in their own
buffers etc.

Buffers have many attributes, e.g. whether the text it contains is modifiable,
or whether it is associated with a file and thus needs to be synchronized to
disk on saving.

**Windows** are viewports _onto_ buffers. If you want to view several files at
the same time or even different locations of the same file, you use windows.

And please, please don't call them _splits_. You can split a window in two, but
that doesn't make them _splits_.

Windows can be split vertically or horizontally and the heights and widths of
existing windows can be altered, too. Therefore you can use whatever window
layout you prefer.

A **tab page** (or just tab) is a collection of windows. Thus, if you want to
use multiple window layouts, use tabs.

Putting it in a nutshell, if you start Vim without arguments, you'll have one
tab page that holds one window that shows one buffer.

By the way, the buffer list is global and you can access any buffer from any
tab.

#### Active, loaded, listed, named buffers?

Run Vim like this `vim file1`. The file's content will be loaded into a buffer.
You have a **loaded buffer** now. The content of the buffer is only synchronized
to disk (written back to the file) if you save it within Vim.

Since the buffer is also shown in a window, it's also an **active buffer**. Now
if you load another file via `:e file2`, `file1` will become a **hidden buffer**
and `file2` the active one.

Both buffers are also **listed**, thus they will get listed in the output of
`:ls`. Plugin buffers or help buffers are often marked as unlisted, since
they're not regular files you usually edit with a text editor. Listed and
unlisted buffers can be shown via `:ls!`.

**Unnamed buffers**, also often used by plugins, are buffers that don't have an
associated filename. E.g. `:enew` will create an unnamed scratch buffer. Add
some text and write it to disk via `:w /tmp/foo`, and it will become a named
buffer.

#### Colorschemes?

Colorschemes are the way to style your Vim. Vim consists of many components and
each of those can be customized with different colors for the foreground,
background and a few other attributes like bold text etc. They can be set like
this:
```viml
:highlight Normal ctermbg=1 guibg=red
```
This would paint the background of the editor red. See `:h :highlight` for more
information.

So, colorschemes are mostly a collection of `:highlight` commands.

Actually, most colorschemes are really 2 colorschemes! The example above sets
colors via `ctermbg` and `guibg`. The former definition will only be used if Vim
was started in a terminal emulator, e.g. xterm. The latter will be used in
graphical environements like gVim.

If you ever happen to use a certain colorscheme in Vim running in a terminal
emulator and the colors don't look like the colors in the screenshot at all,
chances are that the colorscheme only defined colors for the GUI.

I use [gruvbox](https://github.com/morhetz/gruvbox) for the GUI and
[janah](https://github.com/mhinz/vim-janah) for the terminal.

More colorschemes: [here](#list-of-colorschemes-1)

## Usage

#### Managing plugins

[Pathogen](https://github.com/tpope/vim-pathogen) was the first popular tool for
managing plugins. Actually it just adjusts the _runtimepath_ (`:h 'rtp'`) to
include all the things put under a certain directory. You have have to clone the
repositories of the plugins there yourself.

Real plugin managers expose commands that help you installing and updating
plugins from within Vim. Hereinafter is a list of commonly used plugin managers
in alphabetic sequence:

- [neobundle](https://github.com/Shougo/neobundle.vim)
- [plug](https://github.com/junegunn/vim-plug)
- [vim-addon-manager](https://github.com/MarcWeber/vim-addon-manager)
- [vundle](https://github.com/VundleVim/Vundle.vim)

Plug is my favorite, but your mileage may vary.

## Debugging

#### General tips

If you encounter a strange behaviour, try reproducing it like this:

```
vim -u NONE -N
```

This will start Vim without vimrc (thus default settings) and in nocompatible
mode (which makes it use Vim defaults instead of vi defaults). (See `:h
--noplugin` for other combinations of what to load at start.)

If you can still reproduce it now, it's most likeley a bug in Vim itself! Report
it to the [vim_dev](https://groups.google.com/forum/#!forum/vim_dev) mailing
list. Most of the time the issue won't be resolved at this time and you'll have to
further investigate.

Often plugin updates introduce new/changed/faulty behaviour. If you're using a
plugin manager, comment them out until you find the culprit.

Issue is still not resolved? If it's not a plugin, it must be your other
settings, so maybe your options or autocmds etc.

Time to use binary search. Repeatedly split the search space in two until you
find the culprit line. Due to the nature of binary division, it won't take many
steps.

In practice it works like this: Put the `:finish` command in the middle of your
vimrc. Vim will skip everything after it. If it still happens, the problem is in
the active upper half. Move the `:finish` to the middle of _that_ half.
Otherwise the issue is in the inactive lower half. Move the `:finish` to the
middle of _that_ half. And so on.

#### Profiling startup time

Vim startup feels slow? Time to crunch some numbers:

```
vim --startuptime /tmp/startup.log +q && vim /tmp/startup.log
```

The first column is the most important as it shows the elapsed absolute time. If
there is a big jump in time between two lines, the second line is either a very
big file or a file with faulty VimL code that is worth investigating.

#### Profiling at runtime

Vim provides a built-in capability for profiling at runtime and is a great way
to find slow code in your environment.

First and foremost, check if `:version` shows `+profile`, which means that the
`profile` feature is enabled. Otherwise you're using a Vim with a smaller
_feature set_. You want a Vim built with the **huge** feature set (see `:h
:version`). Many distros install a Vim with minimal feature set by default, so
you need to install a package called `vim-x11` or `vim-gtk` (yes, even if you
don't use gvim) for more features.

With that said, we're ready for profiling now. The `:profile` command takes a
bunch of sub-commands for specifying what to profile.

If you want to profile _everything_, do this:

```
:profile start /tmp/profile.log
:profile file *
:profile func *
<do something in Vim>
<quit Vim>
```

Vim keeps the profiling information in memory and only writes it out to the
logfile on exit. (Neovim has fixed this using `:profile dump`).

Have a look at `/tmp/profile.log`. All code that was executed during profiling
will be shown. Every line, how often it was executed and how much time it took.

Most of the time that will be plugin code the user isn't familiar with, but if
you're investigating a certain issue, jump to the bottom of the log. Here are
two different sections `FUNCTIONS SORTED ON TOTAL TIME` and `FUNCTIONS SORTED ON
SELF TIME` that are worth gold. On a quick glance you can see, if a certain
function is taking too long.

## List of colorschemes

Here's a list of commonly used colorschemes:

- [base16](https://github.com/chriskempson/base16-vim)
- [gotham](https://github.com/whatyouhide/vim-gotham)
- [gruvbox](https://github.com/morhetz/gruvbox)
- [janah](https://github.com/mhinz/vim-janah)
- [jellybeans](https://github.com/nanotech/jellybeans.vim)
- [molokai](https://github.com/tomasr/molokai)
- [railscasts](https://github.com/jpo/vim-railscasts-theme)
- [solarized](https://github.com/altercation/vim-colors-solarized) (or a lighter variant: [flattened](https://github.com/romainl/flattened))
- [tomorrow](https://github.com/chriskempson/vim-tomorrow-theme)
- [vividchalk](https://github.com/tpope/vim-vividchalk)

## List of plugins

#### Fuzzy finders

- [command-t](https://github.com/wincent/Command-T)
- [ctrlp](https://github.com/ctrlpvim/ctrlp.vim.git)
- [fzf](https://github.com/junegunn/fzf)
- [unite](https://github.com/Shougo/unite.vim)

#### Version control

- [fugitive](https://github.com/tpope/vim-fugitive)
- [gitgutter](https://github.com/airblade/vim-gitgutter)
- [lawrencium](https://bitbucket.org/ludovicchabant/vim-lawrencium)
- [signify](https://github.com/mhinz/vim-signify)
- [vcscommand](https://github.com/vim-scripts/vcscommand.vim)