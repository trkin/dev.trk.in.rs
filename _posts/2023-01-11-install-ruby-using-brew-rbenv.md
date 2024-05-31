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

If you use multiseat than you need to add new group and attach each user to it
https://apple.stackexchange.com/a/45003/449651
Error is
```
brew update && brew upgrade ruby-build
Error: /home/linuxbrew/.linuxbrew/Homebrew is not writable. You should change the
ownership and permissions of /home/linuxbrew/.linuxbrew/Homebrew back to your
user account:
  sudo chown -R $(whoami) /home/linuxbrew/.linuxbrew/Homebrew
```
Add group
```
sudo groupadd brew
compgen -u # list all users
sudo usermod -aG brew dule
sudo usermod -aG brew mile
sudo usermod -aG brew newuser

sudo chgrp -R brew /home/linuxbrew/.linuxbrew/
sudo chmod -R g+w /home/linuxbrew/.linuxbrew/

# check that group has rw-
ls -la /home/linuxbrew/.linuxbrew/Homebrew/README.md
-rw-rw-r-- 1 dule brew 8495 феб  9 08:40 /home/linuxbrew/.linuxbrew/Homebrew/README.md

# also https://zenn.dev/megeton/articles/f9f17d184fead6
git config --global --add safe.directory /usr/local/Homebrew
```
On macos
```
sudo dseditgroup -o create homebrew
sudo dseditgroup -o edit -a dule -t user homebrew
sudo dseditgroup -o edit -a mile -t user homebrew

sudo chown -R :homebrew /opt/homebrew
sudo chmod -R g+w /opt/homebrew
```

Also for postgresql you need to add new user ability to create databases
```
sudo -u postgres psql -d postgres -c "CREATE USER newuser WITH SUPERUSER;"
```

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

# check all installed versions
rbenv versions
```

### Ruby 3.2.0

For compiling Ruby 3.2.0 you should use openssl@3
Check [rbenv/ruby-build](https://github.com/rbenv/ruby-build/wiki) wiki for
additional info

```
brew install openssl@3

export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl@3`"

rbenv install 3.2.0
```

Export these variables, **don't** save them in your `.bashrc` it won't work for
every Ruby version so to be on the safe side just export them every time you
want to install new Ruby version

When you install gems that need to be compiled locally than you might need to
export PATH, LDFLAGS, CPPFLAGS or PKG_CONFIG_PATH
```
export PATH="/home/linuxbrew/.linuxbrew/opt/openssl@3/bin:$PATH"
export LDFLAGS="-L`brew --prefix openssl@3`/lib"
export CPPFLAGS="-I`brew --prefix openssl@3`/include"
export PKG_CONFIG_PATH="`brew --prefix openssl@3`/lib/pkgconfig"
```

Note: **Do not restart terminal** if you are exporting variables, it won't persist in the next session

For example Installing Ruby version that uses openssl@1.1

To install version of ruby that uses openssl@1.1 we need to use RUBY_CONFIGURE_OPTS
```
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl@1.1`"
ruby install 2.6.7

export LDFLAGS="-L`brew --prefix openssl@1.1`/lib"
export CPPFLAGS="-I`brew --prefix openssl@1.1`/include"
export PKG_CONFIG_PATH="`brew --prefix openssl@1.1`/lib/pkgconfig"

bundle
```

For ruby 2.6.7 on mac for error
```
vm.c:2295:9: error: call to undeclared function 'rb_native_mutex_destroy'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
        rb_native_mutex_destroy(&vm->waitpid_lock);
        ^
```
you might need
```
CFLAGS="-Wno-error=implicit-function-declaration" rbenv install 2.6.7
```

For ruby 2.6.6 on mac you might see error
```
error closure.c:264:14: error: call to undeclared function 'ffi_prep_closure'; ISO C99 and later do not support implicit function declarations [-Wimplicit-function-declaration]
```
so you need to configure libffi
```
RUBY_CFLAGS=-DUSE_FFI_CLOSURE_ALLOC rbenv install 2.6.6
```

For ruby 2.7.6 on ubuntu and error
```
crypto/comp/c_zlib.c:35:11: fatal error: zlib.h: No such file or directory
```
solution is to install zlib
```
sudo apt-get install zlib1g-dev
```

For Ruby 3.2.1 on ubuntu you might need to use define readline location
```
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl` --with-readline-dir=`brew --prefix readline`"
```
On macOS you, ruby 3.2.1 can be installed `rbenv install 3.2.1` but make sure
you are not using Rosseta (Finder > Menu Go > Utilities > Terminal > right click
Get Info > uncheck Open using Rosetta) so the output is arm
```
uname -m
arm64
```

For Ruby 3.0.1 on M1 I got this error
```
 error: use of undeclared identifier 'RSA_SSLV23_PADDING'
```
solution is to use old ssl so either uninstall new ssl 
https://github.com/postmodern/ruby-install/issues/409#issuecomment-1560601024
```
brew uninstall --ignore-dependencies openssl
brew install openssl@1.1
```
or export variables
```
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@1.1)"
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=$(brew --prefix openssl@1.1)"
export PATH="/opt/homebrew/opt/openssl@1.1/bin:$PATH"

export LDFLAGS="-L/opt/homebrew/opt/openssl@1.1/lib"
export CPPFLAGS="-I/opt/homebrew/opt/openssl@1.1/include"

export PKG_CONFIG_PATH="/opt/homebrew/opt/openssl@1.1/lib/pkgconfig"

rbenv install
```
or you can try with env
```
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
