---
title: "Jazz Up your ZSH Terminal with Prezto"
date: 2020-07-14T17:48:15+02:00
draft: false

# meta description
description: "How to customize your ZSH terminal with Prezto, complete installation and setup using zsh plugins, zsh themes and much more."

# featured image
feature: "banner.png"

# taxonomies
tags:
 - Zshell
 - Oh-My-Zsh
 - Prezto
 - Linux

# post type
type: post
---

For some time now I've been using zsh as my default Unix shell, which is an extended Bourne shell with many improvements, including some features from bash, ksh and tcsh. One of my favorite shells is fish shell. It's easy to set-up and many of the features that I use comes pre-installed, as it is not POSIX supported it has a much readable (fish) syntax for scripting.

> Why I switched from fish to zsh, and using Prezto over oh-my-zsh.

The latter can also be a drawback. Many of the bash scripts I work with are written in bash. That means that fish does not support all of the syntax. For that reason I went looking for an alternative and quickly came to [zsh](https://en.wikipedia.org/wiki/Z_shell) (pronounce it as Z shell). Despite the fact that zsh does not support POSIX by default, zsh makes it possible to emulate POSIX.

The most popular zsh framework is Oh-My-Zsh. Oh my zsh gives you a lot of options to set up your zsh environment. I've been using Oh-My-Zsh for a while and for most this is a good choice. However, after using Oh-My-Zsh, I found myself not using many of the features that Oh-My-Zsh has to offer.

So I was looking for an alternative. I quickly came to [Prezto](https://github.com/sorin-ionescu/prezto). Like Oh-My-Zsh, Prezto is a configuration framework for zsh. It comes with auto completion, aliases, function and prompt themes.

> A framework is not necessary if you want to add your own custom configurations, 
however it does make things a lot easier to set up.

*I will discuss the following: how to install zsh and Prezto, useful plugins, theming, features such as aliases and custom functions.*
My installation will be on macOS, but this will not be much different if you are a Linux user. If you are on Windows it will be slightly different. If you would like to know how to install zsh and Prezto on Windows, let me know in the comments so I can add those steps later.

## TLDR;
If you prefer to get started right away, you can download my zsh configuration [here](https://gist.github.com/rickdaalhuizen90/2ce9eb662e71024ca12e0635cfb44dab).

## Installation
If you are using macOS Catalina or higher then you may have heard that they have replaced bash with zsh as default shell.

Run to see what your current shell is
```bash
echo $SHELL
```
If zsh is not installed, you can use Homebrew. Homebrew is a package manager for macOS. For Linux you can use Snapcraft or Flatpak and Chocolatey for windows. Make sure you have a package manager installed before continuing.
```bash
brew install zsh
```
Then set zsh as default shell:
```bash
chsh -s /bin/zsh
```
Zsh uses ~ / 5 startup files. These will be visible after we install Prezto.
```bash
$ZDOTDIR/.zshenv
$ZDOTDIR/.zprofile
$ZDOTDIR/.zshrc
$ZDOTDIR/.zlogin
$ZDOTDIR/.zlogout
```
> From your terminal you can echo $ ZDOTDIR to see what it refers to. By default it refers to $HOME.

 - The .zshenv is used every time you start zsh. This is for your environment variables like $PATH, $EDITOR, $VISUAL, $PAGER, $LANG.
 - The .zprofile is an alternative to .zlogin and these two are not intended to be used together.
 - The .zshrc is where we add our aliases, functions and other customizations. In this tutorial we will mainly discuss the .zshrc.
 - The .zlogin is started when you log in your shell but after your .zshrc.
 - The .zlogout is used when you close your shell.

We'll come back to the .zshrc later, for now we'll leave it as is. Let's start by installing Prezto.
```bash
git clone --recursive https://github.com/sorin-ionescu/prezto.git "${ZDOTDIR:-$HOME}/.zprezto"
```

Then copy-paste the following into your terminal:
```bash
setopt EXTENDED_GLOB
for rcfile in "${ZDOTDIR:-$HOME}"/.zprezto/runcoms/^README.md(.N); do
  ln -s "$rcfile" "${ZDOTDIR:-$HOME}/.${rcfile:t}"
done
```
After we have installed Prezto you will find a .zpreztorc file in your $HOME folder next to the zsh startup files. Here we edit our Prezto options.

{{<figure src="iterm-prezto-options.png" alt="Iterm prezto options" >}}

To install plugins in zsh we need a plugin manager, I prefer zplug but feel free to use any of the other plugin managers. Zplug also makes it possible to install Oh-My-Zsh plugins without any hassle.
To install zplug run the following:

```bash
curl -sL --proto-redir -all,https https://raw.githubusercontent.com/zplug/installer/master/installer.zsh | zsh
```
We will be installing the following
 - Git - a tab-completion library for Git
 - Osx - provides a few utilities for macOS
 - Zsh auto-suggestions - Fish-like fast/unobtrusive auto-suggestions for zsh.
 - Fasd - offers quick access to files and directories for POSIX shells
 - Enhancd - A next-generation cd command with an interactive filter
 - Fzf - A general-purpose command-line fuzzy finder.
 - Lsdeluxe - colorizes the ls output with color and icons
 - Yarn - adds common aliases and completion for the Yarn package manager
 - Powerlevel10k - Powerlevel10k theme for zsh with configuration wizard.

These are my essential plugins that I use, this will be different for everyone so take the time to see which plugins suit you. For more plugins see: awesome zsh plugins.
Add the following to your ~/.zshrc file.
```bash
# Customize to your needs...
source ~/.zplug/init.zsh

# Plugins
zplug "plugins/git",   from:oh-my-zsh
zplug "plugins/osx",   from:oh-my-zsh
zplug "zsh-users/zsh-autosuggestions"
zplug "clvv/fasd"
zplug "b4b4r07/enhancd"
zplug "junegunn/fzf"
zplug "Peltoche/lsd"
zplug "g-plane/zsh-yarn-autocompletions"
zplug "romkatv/powerlevel10k", as:theme, depth:1
```
Then we run:
```bash
source ~/.zshrc && zplug install
```

## Add a font
Let's start by installing a Nerd font, this ensures that all glyphs and icons in our theme are supported.
Go to downloads on the Nerd font website and download the font of your choice, I prefer Hack.
Then set your font in your terminal:

{{<figure src="iterm-font.png" alt="Iterm font" >}}

## Adding a theme
Prezto comes standard with a number of themes that you can use.Once you type _prompt -l_ you will see a list of currently available themes. For example, use prompt -s paradox to set a theme.

{{<figure src="iterm-zsh-prompt.png" alt="Iterm zsh prompt" >}}

If you are satisfied with your theme, you can save this in your .zpreztorc file as follows. In this case we go for powerlevel10k, perhaps one of the most well-known themes out there.
To set this up we go to "themes" in our preztorc file.
```bash
# Set the prompt theme to load.
# Setting it to 'random' loads a random theme.
# Auto set to 'off' on dumb terminals.
zstyle ':prezto:module:prompt' theme 'powerlevel10k'
```
Then start with the p10k setup wizard, which you start as follows:
```bash
p10k configure
```
After following the setup wizard your powerlevel10k is set. This is my result

{{<figure src="iterm-zsh-theme.png" alt="Iterm zsh theme" >}}

## Aliases
The common alias plugin gives us access to a list of aliases we can use. However, if you want to add your own aliases, this can be done in your .zshrc file.
Examples of particular aliases
```bash
# Aliases
alias untar='tar -zxvf' # Unpack .tar file
alias wget='wget -c' # Download and resume
alias getpass='openssl rand -base64 20' # Generate password
alias sha='shasum -a 256' # Check shasum
alias ping='ping -c 5' # Limit ping to 5'
alias www='php -S localhost:8000' # Run local web server
```
## Custom functions
For some complex commands, an alias is insufficient and long commands are difficult to maintain. We can use custom functions for this. There are several ways to create a custom function in zsh. I will give two examples that are most common.
One is to define your functions as follows in your _~/.zshrc_:

```bash
# Custom functions
hello_world () {
    echo "Hello world"
}
```
After that use the source command to load your function and run it as follows.

```bash
source ~/.zshrc && hello_world
```

The second way is to define your custom functions.

This will keep your zsh configuration more organized rather than putting everything in your ~/.zshrc file.

When you echo $fpath in your console you can see a set of directories being defined, which contain files that can be marked to be loaded automatically. To add a folder to your $fpath we need to add the following in your ~/.zshrc.

```bash
fpath=( ~/.zfunc "${fpath[@]}" )
```

Then we check to see that zfunc is being added to $fpath.

```bash
source ~/.zshrc && echo $fpath | grep -o zfunc
```

Finally we need to create a directory ~/.zfunc with a file "hello" - you can name it however you like but this is how I prefer it.

```bash
mkdir ~/.zfunc && touch ~/.zfunc/hello
```

As an example I created a simple hello function.
```bash
hello () {
    echo "Hello $@"
}
hello
```

After you need to mark the function to be automatically loaded, the U option suppress alias expansion and z stands for to use zsh instead of kash style:

```bash
autoload -Uz hello
```

Remember to source it every time you update your function like this:

```bash
source ~/.zshrc/hello
```

After you can run:
```bash
hello John
```

We have reached the end of this blog. I hope I've provided you with enough information to create your own zsh configuration. If there are any questions or suggestions, let me know in the comments.