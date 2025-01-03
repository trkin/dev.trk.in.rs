---
layout: post
---


# Sudo commands

First steps on new Ubuntu machine.
This commands are for amins with sudo priviledges

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

## Install Brew

Use brew so all users can use the same libs from `/home/linuxbrew/`
<https://brew.sh/>
Note that we will install only for one user and other will ssh to it
```
# on macos go to System Settings > Users & Groups > Add user bbrew with admin rights
# use strong password and Allow bbrew to administer this computer
# on ubuntu follow those commands
sudo groupadd brew
sudo useradd -m -g brew -s /bin/bash brew
sudo passwd brew
sudo mkdir -p /home/linuxbrew/.linuxbrew/
sudo chown -R brew:brew /home/linuxbrew/.linuxbrew/
```
Install as brew only for brew user since not we do not need sudo
```
ssh brew@localhost
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Add brew to shell
cat >> ~/.bashrc << 'HERE_DOC'
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
HERE_DOC
```

That is.

For other users you need to load brew but also change brew command
All users should be able to `ssh brew@localhost` so you need to copy keys

```
ssh-copy-id brew@localhost
```

and add this function to .bashrc or .zprofile
```
# .bashrc or .zprofile
if [ -d /home/linuxbrew/.linuxbrew/bin ]; then
  eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
elif [ -d /opt/homebrew/bin ]; then
  eval "$(/opt/homebrew/bin/brew shellenv)"
fi

function brew() {
  if [ -d /home/linuxbrew/.linuxbrew/bin ]; then
    ssh brew@localhost 'bash  --login -c "export PATH=/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:$PATH && brew '"$@"'"'
  elif [ -d /opt/homebrew/bin ]; then
    ssh brew@localhost 'bash  --login -c "export PATH=/opt/homebrew/bin:$PATH && brew '"$@"'"'
  else
    echo 'Can not find /home/linuxbrew/ or /opt/homebrew. Please install brew as brew user'
  fi
}
```
usage is like
```
bbrew
Example usage:
  brew search TEXT|/REGEX/
```

Here are the reasons why we can not share the brew

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
Set `umask` if not already allow group flags (value `0002`)
```
# ~/.bashrc
umask 0002
```
Add group and set sticky bit
```
sudo groupadd brew
compgen -u # list all users
sudo usermod -aG brew dule
sudo usermod -aG brew mile
sudo usermod -aG brew newuser

sudo chgrp -R brew /home/linuxbrew/.linuxbrew/
sudo chmod -R g+w /home/linuxbrew/.linuxbrew/
# Set the sticky bit to ensure new files and directories inherit the group
# ownership: https://en.wikipedia.org/wiki/Setuid#When_set_on_a_directory
find /home/linuxbrew/.linuxbrew -type d -exec sudo chmod g+s {} +

# check that group has rw-
ls -la /home/linuxbrew/.linuxbrew/Homebrew/README.md
-rw-rw-r-- 1 dule brew 8495 феб  9 08:40 /home/linuxbrew/.linuxbrew/Homebrew/README.md

# check setgid, it should start with 2
stat -c "%a %A" $(brew --prefix)
2775 drwxrwsr-x

# also https://zenn.dev/megeton/articles/f9f17d184fead6
git config --global --add safe.directory $(brew --prefix)
# otherwise you can not update
==> Updating Homebrew...
Warning: No remote 'origin' in /home/linuxbrew/.linuxbrew/Homebrew/Library/Taps/homebrew/homebrew-bundle, skipping update!
Error: Your Homebrew is too outdated for `brew bundle`. Please run `brew update`!
```
On macos
```
sudo dseditgroup -o create homebrew
sudo dseditgroup -o edit -a dule -t user homebrew
sudo dseditgroup -o edit -a mile -t user homebrew

sudo chown -R :homebrew /opt/homebrew
sudo chmod -R g+w /opt/homebrew
```

Log out and login so the group has an effect and try to run `brew upgrade` as
different users.

```
brew update && brew upgrade ruby-build

Error: Failure while executing; `/usr/bin/env cp -pR /tmp/homebrew-unpack20240628-559339-n6br4d/expat/. /home/linuxbrew/.linuxbrew/Cellar/expat` exited with 1. Here's the output:
cp: preserving times for '/home/linuxbrew/.linuxbrew/Cellar/expat/.': Operation not permitted
```
so I tried to override cp with --no-preserve=timestamps but than it does not
detect that it has to upgrade

## Other sudo

Also for postgresql you need to add new user ability to create databases
```
sudo -u postgres psql -d postgres -c "CREATE USER newuser WITH SUPERUSER;"
```

# Regular user comands

## Initialize ssh to brew user

All users should be able to `ssh brew@localhost` so you need to copy keys

```
ssh-copy-id brew@localhost
```

and add this function to .bashrc or .zprofile
```
# .bashrc or .zprofile
function bbrew() {
  if [ -d /home/linuxbrew/.linuxbrew/bin ]; then
    ssh brew@localhost 'bash  --login -c "export PATH=/home/linuxbrew/.linuxbrew/bin:/home/linuxbrew/.linuxbrew/sbin:$PATH && brew '"$@"'"'
  elif [ -d /opt/homebrew/bin ]; then
    ssh brew@localhost 'bash  --login -c "export PATH=/opt/homebrew/bin:$PATH && brew '"$@"'"'
  else
    echo 'Can not find /home/linuxbrew/ or /opt/homebrew. Please install brew as brew user'
  fi
}
```
usage is like
```
bbrew
Example usage:
  brew search TEXT|/REGEX/
```

Additional tips:

If you need to read from Brewfile, than youo need to copy to brew
```
scp Brewfile brew@localhost
bbrew bundle --no-upgrade --no-lock
```

Add to your `.bashrc` and reopen terminal. Typing `reset` won't work

```
cat >> ~/.bashrc << 'HERE_DOC'
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
HERE_DOC

cat ~/.bashrc
# you should see above line: eval ....
```
Now you can use `brew` every time you open terminal


## Install rbenv

```
brew install rbenv
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
# to undo you can use
rbenv shell --unset

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

## Errors

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

For ruby 2.7.8 and mini_racer error
```
linking shared-object mini_racer_extension.so
g++: error: /home/ubuntu/xceednet/shared/bundle/ruby/2.7.0/gems/libv8-node-16.10.0.0-x86_64-linux-musl/vendor/v8/x86_64-linux/libv8/obj/libv8_monolith.a: No such file or
directory
make: *** [Makefile:263: mini_racer_extension.so] Error 1
```

solution is to link hardcoded paths
https://github.com/rubyjs/mini_racer/issues/243#issuecomment-2439906407
```
cd /home/ubuntu/xceednet/shared/bundle/ruby/2.7.0/gems/libv8-node-16.10.0.0-x86_64-linux-musl/vendor/v8/
ln -s x86_64-linux-musl x86_64-linux
```

Installing libv8-node-16 is sometimes not supported with error in bit-field.h
```
../deps/v8/src/base/bit-field.h:43:29: error: integer value 7 is outside the valid range of values [0, 3] for the enumeration type 'Kind'
[-Wenum-constexpr-conversion]
1 error generated.
make: ***
[/Users/dule/.asdf/installs/ruby/2.6.6/lib/ruby/gems/2.6.0/gems/libv8-node-16.10.0.0/src/node-v16.10.0/out/Release/obj.target/v8_init/deps/v8/src/init/setup-isolate-full.o]
Error 1
1 error generated.
m
```
so the solution is to install the gem using system node
```
gem install libv8 -v '16.10.0.0' -- --with-system-v8
bundle install
```
but issue could be that on mac darwin we can not compile node
```
uname -r
24.1.0
```
Similar logs on
https://bugs.freebsd.org/bugzilla/show_bug.cgi?id=272013
I tried to upgrade to latest miniracer which is supported by ruby 2.6.6 but that
is 0.6.5 which depends on libv8-node 16.20.0.0
https://github.com/rubyjs/mini_racer/blob/stable-0.6/lib/mini_racer/version.rb


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

Error installing ruby 3.0.x on new mac m1
```
bigdecimal.c:1560:5: error: 'maybe_unused' attribute cannot be applied to types
 1560 |     ENTER(3);
```
can be solved with patch https://bugs.ruby-lang.org/issues/20760#note-4
https://github.com/rbenv/ruby-build?tab=readme-ov-file#applying-patches
https://github.com/asdf-vm/asdf-ruby?tab=readme-ov-file#use

```
curl -sSL https://github.com/ruby/ruby/commit/1dfe75b0beb7171b8154ff0856d5149be0207724.patch -o ruby-3.0.x.patch
RUBY_APPLY_PATCHES=$'./ruby-3.0.x.patch' asdf install ruby 3.0.6
```

When you install new gems, sometimes you need to rehash to create new shims
[https://github.com/rbenv/rbenv#rbenv-rehash](https://github.com/rbenv/rbenv#rbenv-rehash)

```
gem install jekyll
Command 'jekyll' not found, but can be installed with:

rbenv rehash
jekyll # now it works
```


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

## node yarn

--Install nvm <https://github.com/nvm-sh/nvm>--

we will use https://github.com/nodenv/nodenv
```
brew install nodenv

cat >> ~/.bashrc << 'HERE_DOC'
eval "$(nodenv init - bash)"
HERE_DOC
```

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
