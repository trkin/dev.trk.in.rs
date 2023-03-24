---
layout: post
---


## First steps on new Ubuntu machine

```
# git is used to code collaboration
sudo apt install -y git curl
# necessary if you want to commit to any project
git config --global user.email "you@example.com"
git config --global user.name "Your Name"
# nice editor
sudo apt install vim
# if you want to change default editor from nano to something else
sudo update-alternatives --config editor
# enable ssh to localhost
sudo apt install openssh-server
# generate ~/ssh/id_rsa and ~/ssh/id_rsa.pub to add to
ssh-keygen

# network tool to find all open ports and ip addresses, eg  nmap -F 192.168.1.-
sudo apt install nmap

# for building ruby
sudo apt install -y build-essential libz-dev
```

Install google chrome from <https://www.google.com/chrome/>
Right click on .deb file in Downloads, select Properties and in 'Open With' tab select 'Software Install' and click 'Set as default'. Exit and double click on the .deb file to install Chrome.

## Brew

Use brew instead of manually install packages so all users can use the same libs
<https://brew.sh/>

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add to your `.bashrc` and reopen terminal. Typing `reset` won't work

```
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/$USER/.bashrc
cat ~/.bashrc # you should see above line
```
Now you can use `brew` every time you open terminal

## Install rbenv

```
brew install rbenv ruby-build libyaml openssl@1.1
```

Add this to your `~/.bashrc` so you can access rbenv commands

```
echo export PATH="$HOME/.rbenv/bin:$PATH" >> ~/.bashrc
echo eval "$(rbenv init -)" >> ~/.bashrc
```

## Installing Ruby using rbenv

To change ruby version you can use `.ruby-version` file in current or any parent
folder. For example you can set default version with `.ruby-version` in your
home folder `~/`
```
echo 3.2.0 > ~/.ruby-version
```

You can also change ruby version with this command or env variable
```
rbenv shell 3.2.0

# this is the same as export RBENV_VERSION=3.2.0
```

### Ruby 3.2.0

For compiling Ruby 3.2.0 you should use openssl@3
Check [rbenv/ruby-build](https://github.com/rbenv/ruby-build/wiki) wiki for additional info

```
brew install openssl@3

rbenv install 3.2.0
rbenv versions # check all installed versions
```

If you need to have openssl@3 first in your PATH, run:
```
echo 'export PATH="/home/linuxbrew/.linuxbrew/opt/openssl@3/bin:$PATH"' >> /home/$USER/.bashrc
```

Export these variables, **don't** save them in your `.bashrc` it won't work for every Ruby version so to be on the safe side just export them every time you want to install new Ruby version

For compilers to find openssl@3
```
export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/openssl@3/lib"
export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/openssl@3/include"
```

For pkg-config to find openssl@3
```
export PKG_CONFIG_PATH="/home/linuxbrew/.linuxbrew/opt/openssl@3/lib/pkgconfig"
```

Note: **Do not restart terminal** if you are exporting variables, it won't persist in the next session

### Installing Ruby version that uses openssl@1.1

To install version of ruby that uses openssl@1.1 we need to use RUBY_CONFIGURE_OPTS

```
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl@1.1`"
```

Then run
```
rbenv install ruby-version-you-need
```

For compilers to find openssl@1.1 you may need to export (this is needed for rvm):

```
export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/openssl@1.1/lib"
export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/openssl@1.1/include"
export PKG_CONFIG_PATH="/home/linuxbrew/.linuxbrew/opt/openssl@1.1/lib/pkgconfig"
```

## Jekyll

When you install new gems, sometimes you need to rehash to create new shims
[https://github.com/rbenv/rbenv#rbenv-rehash](https://github.com/rbenv/rbenv#rbenv-rehash)

```
gem install jekyll
Command 'jekyll' not found, but can be installed with:

rbenv rehash
jekyll # now it works
```

## Rails

If you notice an error when you are using `rails s` command
```
LoadError: libssl.so.1.1: cannot open shared object file: No such file or directory - /home/dule/.rbenv/versions/3.2.0/lib/ruby/gems/3.2.0/gems/puma-5.6.5/lib/puma/puma_http11.so
```
you should remove puma gem and install with correct LD and CPP flags pointing to
openssl 3.
```
gem uninstall puma

export LDFLAGS="-L$(brew --prefix openssl@3)/lib"
export CPPFLAGS="-I$(brew --prefix openssl@3)/include"

bundle

# or you can install specific version using arguments to `gem install`
gem install puma -v 5.6.5 -- --with-cppflags=-I`brew --prefix openssl@3`/include --with-ldflags=-L`brew --prefix openssl@3`/lib
```

Alternative solution is to disable SSL when installing puma
```
gem uninstall puma
DISABLE_SSL=1 bundle
```

## NVM node yarn

Install nvm <https://github.com/nvm-sh/nvm>

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
```
than restart shell (ie open new shell) install node
```
nvm install 14
npm install -g yarn
```

Old node-sass has some issues with node 16 (it can not find python even python2
is installed)
[stackoverflow](https://stackoverflow.com/questions/67241196/error-no-template-named-remove-cv-t-in-namespace-std-did-you-mean-remove)

## Wall

Enable tty group so user can send messages
<https://unix.stackexchange.com/a/313558/150895>

## Python

You can install python using pyenv https://github.com/pyenv/pyenv similar to
rbenv
```
brew install pyenv
pyenv install 2.7.18
# pyenv global 2.7.18
pyenv shell 2.7.18
pyenv versions

# Add to .bashrc
# PATH=$(pyenv root)/shims:$PATH
# eval "$(pyenv init -)"
```
