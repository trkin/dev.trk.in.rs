---
layout: post
---

# Keyboard remap

## xremap

https://github.com/k0kubun/xremap

Download from releases https://github.com/k0kubun/xremap/releases and put in
your path.
For specific device use `xremap --device 'Name'`
`modmap` allow key-to-key remap
`keymap` for remapping a sequence of key combinations with modifiers
* `SHIFT-`, `CTRL-`, `ALT-`, `SUPER-`

Key codes https://github.com/emberian/evdev/blob/1d020f11b283b0648427a2844b6b980f1a268221/src/scancodes.rs#L26-L572
but you can see them with
```
sudo RUST_LOG=debug ~/config/bin/xremap ~/config/bashrc/xremap.config.yml
```

Problem is that I can not remap semicolon to colon ie it still show semicolon
```
keymap:
  - name: Try
    remap:
      KEY_SEMICOLON: SHIFT-KEY_SEMICOLON
      a: SHIFT-b
```

so I switch to keyd.

Before GNOME 14 we chould get WMCLASS with
```
gdbus call --session \
  --dest org.gnome.Shell \
  --object-path /org/gnome/Shell \
  --method org.gnome.Shell.Eval "global.display.focus_window.get_wm_class()"
```
but now we can not use eval.
Another way to get WMCLASS is using Gnome extension
https://extensions.gnome.org/extension/5060/xremap/ and run
```
sleep 3 && busctl --user call org.gnome.Shell /com/k0kubun/Xremap com.k0kubun.Xremap WMClass
```

## Keyd

man page https://github.com/rvaiya/keyd/blob/master/docs/keyd.scdoc

Install with https://github.com/rvaiya/keyd#from-source
```
git clone https://github.com/rvaiya/keyd
cd keyd
make && sudo make install
sudo systemctl enable keyd && sudo systemctl start keyd

# stop
sudo systemctl stop keyd && sudo systemctl disable keyd
```

Usefull commands

```
# reload configuration
sudo keyd reload

# config errors can be found on
sudo journalctl -eu keyd

# Use `<backspace>+<escape>+<enter>` to stop keyd if you can not use keyboard

# List all keycode
keyd list-keys
# See key codes while typing
sudo keyd monitor
```

Main config file must start with `[ids]`
```
# /etc/keyd/default.conf
[ids]
*

[layer-name:modifier]
key alias = key/macro/action
```
modifier is one of:
* `C` control
* `M` meta/super
* `A` alt
* `S` shift
* `G` AltGr

Example capslock is activating nav layer

```
capslock = layer(nav)

[nav]
h = left
l = right
```

You can remap whole layer with modifier
[capslock:C]
TODO: 


App specific remaps
https://github.com/rvaiya/keyd#application-specific-remapping-experimental
```
sudo usermod -aG keyd dule

# ~/.config/keyd/app.conf
[chromium]

alt.[ = C-S-tab
alt.] = macro(C-tab)

keyd-application-mapper
Gnome detected
keyd extension not found, installing...
Success! Please restart Gnome and run this script one more time.
```

after reboot
