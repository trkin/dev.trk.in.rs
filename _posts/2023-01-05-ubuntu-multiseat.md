---
layout: post
---

We started with multiseat



Use brew instead of manually install packages so all users can use the same libshttps://brew.sh/

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# run those three commands, just replace 'dule' with your username
echo '# Set PATH, MANPATH, etc., for Homebrew.' >> /home/dule/.profile
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> /home/dule/.profile
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```
