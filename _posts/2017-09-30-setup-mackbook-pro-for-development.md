---
layout: post
title: Set up a new Macbook pro for development
---

Yesterday I replaced my old Macbook Pro with a new one. I got the new Retina
MBP with the TouchBar, which meant that I have to take a bit of a risk, because
the new TouchBar and the new keyboard were a bit of an unknown for me. 

Why did I replace it? It seems that at my current workplace I will be able to 
use my own notebook due to an introduction of a BYOD (Bring Your Own Device)
policy and my old one just doesn't have the capability to run the environment
I need to run there. I am working on reducing those requirements to a level 
where an 8GB notebook is enough, but it is probably still some months away.
Also, the thing is 4 years old! Actually, I am very happy about that, and I do
hope that this one will repeat that experience. I do hope that I'll have 4
or 5 good years with this.

But anyhow, not it sits on my desk (and I am writing this post on it), and I
wanted to share the setup process I am using to configure my primary 
development environment, so here it is.

## First impressions

My first impressions on the new machine is that it is a very nice machine. 
There are some differences compared to what I am used to, but after a few 
hours it seems totally OK. I was worried about the keyboard, but after using
it form a few hours, my worries are gone. About the TouchBar ... eh, it needs
some getting used to. I have no big trouble hitting the esc keys, but it's not
as firm as I'd want to. I kinda miss the feedback for hitting it right now. 
We'll see how it works out in the end.

## Setting it up

After the initial configuration (you know: iCloud, Touch ID, disc encryption, 
etc), and the update to High Sierra, I started to set up this as my primary 
develoment machine. Which means that I need a couple of things:

- Command line tooling:
    - Git
    - Ruby for jekyll
    - PHP for the occasions
    - Java tooling: JDK, Maven, Gradle
- Applications:
    - Visual Studio Code
    - Jetbrains Toolbox
    - Spotify
    - Slack
    - Docker
    - iTerm2
- Fonts - because which font I use matters
    - [Hack](http://sourcefoundry.org/hack/)
    - [Source Code Pro](http://adobe-fonts.github.io/source-code-pro/)
    - [Fira Code](https://github.com/tonsky/FiraCode)

### Basics: SSH key

When I set up a new computer I always start with a new SSH key. I use mine to
log in to [Github] and [Gitlab], but I do not use PGP, so this method is 
suitable for me for now. Generating one is as simple as:

```bash
ssh-keygen -t rsa -b 4096 -C "my.email@address.com"
```

Afterwards I just update these keys on the respective sites and I am good to 
go.

### Basics: git + iterm2

I still need to automate some things, but it's getting better. I still have to
manually download iTerm2, and configure git with:

```bash
git --version
``` 

so that XCode doesn't get in my way anymore. But afterwards I can just use 
[my dotfiles repository] with the following command:

```bash
git clone --recursive git@github.com:sandor-nemeth/dotfiles.git
```

The `--recursive` part is important because that checks out all submodules 
which I have in the repository, therefore I get `Vundle`, `oh-my-zsh` and 
`tpm` (tmux plugin manager) right away. Afterwards I just need to link a few
files to the home directory:

```bash
ln -s ~/dotfiles/zsh ~/.oh-my-zsh
ln -s ~/dotfiles/.zshrc ~/.zshrc
ln -s ~/dotfiles/zsh-theme/gitster.zsh-theme ~/dotfiles/zsh/themes/
ln -s ~/dotfiles/.tmux.conf ~/.tmux.conf
ln -s ~/dotfiles/tmux ~/.tmux
ln -s ~/dotfiles/vim ~/.vim
ln -s ~/dotfiles/.vimrc ~/.vimrc
```

and my command line is all set up. 

### Brewing the Mac

Next step is to set up the applications I'll need. For this I normally use 
[homebrew], so first I install that:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

And then there is something I didn't know before, but I added it now:

I can write a file called `Brewfile` residing in my home directory, and there 
I can define all the things I want to have installed from brew, which is pretty
neat. Here is how my Brewfile looks like at the moment:

```bash
tap 'caskroom/cask'
tap 'caskroom/fonts'
tap 'homebrew/php'

brew 'zsh'
brew 'zsh-completions'
brew 'tmux'
brew 'git'
brew 'npm'
brew 'php71'

cask 'java'
brew 'maven'
brew 'gradle'

cask 'google-chrome'
cask 'spotify'
cask 'slack'
cask 'docker'
cask 'vlc'
cask 'alfred'

cask 'font-fira-code'
cask 'font-source-code-pro'
cask 'font-hack'

cask 'visual-studio-code'
cask 'jetbrains-toolbox'
```

This is pretty neat. With this file I practically installed everything I want 
to have, without any further configuration. It may be that later a few things
will be added to the file, but initially this is a good start. 

And with this, the setup is practically completed! All I need now is to 
style the environments how I like them, and that's it. But that is for 
another post.

[my dotfiles repository]: http://github.com/sandor-nemeth/dotfiles
[Github]: https://github.com
[Gitlab]: https://gitlab.com
[homebrew]: https://brew.sh
