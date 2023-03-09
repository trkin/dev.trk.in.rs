---
layout: post
---


# First steps on new Ubuntu machine

```
# git is used to code collaboration
sudo apt install -y git curl
# nice editor
sudo apt install vim
# enable ssh to localhost
sudo apt install openssh-server
# generate ~/ssh/id_rsa and ~/ssh/id_rsa.pub to add to
ssh-keygen

# network tool to find all open ports and ip addresses, eg  nmap -F 192.168.1.-
sudo apt install nmap

# for building ruby
sudo apt install -y build-essential libz-dev
```

Install google chrome from https://www.google.com/chrome/
Right click on .deb file in Downloads, select Properties and in 'Open With' tab select 'Software Install' and click 'Set as default'. Exit and double click on the .deb file to install Chrome.

# Brew

Use brew instead of manually install packages so all users can use the same libs
https://brew.sh/


```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Add to your `.bashrc` and restart terminal
```
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/$USER/.bashrc
cat ~/.bashrc # you should see above line
```
Install rbenv and openssl 1.1
```
brew install rbenv ruby-build openssl@1.1
```

# Rbenv

Rbenv is using `.ruby-version` file in current or any parent folder, so set to your ruby, also add those two kubes to .bashrc
```
echo 3.0.1 > ~/.ruby-version

echo export PATH="$HOME/.rbenv/bin:$PATH" >> ~/.bashrc
echo eval "$(rbenv init -)" >> ~/.bashrc
```
When you install new gems, sometimes you need to rehash to create new shims
[https://github.com/rbenv/rbenv#rbenv-rehash](https://github.com/rbenv/rbenv#rbenv-rehash)
```
gem install jekyll
Command 'jekyll' not found, but can be installed with:

rbenv rehash
jekyll # now it works
```

Install and change to that version
```
rbenv install 3.2.0
rbenv shell 3.2.0 # this will the same as export RBENV_VERSION=3.2.0
```

For compilers to find openssl@1.1 you may need to set (this is needed for rvm):
```
export LDFLAGS="-L/home/linuxbrew/.linuxbrew/opt/openssl@1.1/lib"
export CPPFLAGS="-I/home/linuxbrew/.linuxbrew/opt/openssl@1.1/include"
export PKG_CONFIG_PATH="/home/linuxbrew/.linuxbrew/opt/openssl@1.1/lib/pkgconfig"

# To install version of ruby that uses openssl 1.1 we need to use RUBY_CONFIGURE_OPTS
export RUBY_CONFIGURE_OPTS="--with-openssl-dir=`brew --prefix openssl@1.1`"
```


# NVM node yarn

Install nvm https://github.com/nvm-sh/nvm

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

# Wall

Enable tty group so user can send messages
https://unix.stackexchange.com/a/313558/150895
