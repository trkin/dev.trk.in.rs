---
layout: post
---

# Brew

Use brew instead of manually install packages so all users can use the same libs
https://brew.sh/


```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

```

echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/$USER/.bashrc

```

# Rbenv

Install rbenv and openssl 1.1
```
brew install rbenv ruby-build openssl@1.1
```
Remove rvm and install rbenv https://gist.github.com/akdetrick/7604130
Make sure rbenv is a local installation, visible from terminal!
```
# follow steps
rbenv init
# rbenv to be able to find bin from gems you need to add
# .bashrc
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init -)"
```
When you install new gems, sometimes you need to rehash to create new shims
https://github.com/rbenv/rbenv#rbenv-rehash
```
gem install jekyll
rbenv rehash
jekyll
```

Install and change to that version
```
rbenv install 3.2.0
rbenv shell 3.2.0 # this export RBENV_VERSION=3.2.0
```

For compilers to find openssl@1.1 you may need to set (this is needed for rvm):
```
export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/openssl@1.1/lib"
export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/openssl@1.1/include"
export PKG_CONFIG_PATH="/home/linuxbrew/.linuxbrew/opt/openssl@1.1/lib/pkgconfig"

# To install version of ruby that uses openssl 1.1 we need to use RUBY_CONFIGURE_OPTS
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl@1.1`"
```


# Wall

Enable tty group so user can send messages
https://unix.stackexchange.com/a/313558/150895
