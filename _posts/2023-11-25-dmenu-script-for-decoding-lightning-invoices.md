---
layout: post
title:  "Dmenu script for decoding lightning (bolt11) invoices"
youtubeId: zJefEQVEs8Q
---

Here is a helper script i quite often use to quickly inspect a decoded bolt11 invoice.
It uses the `bolt11` python module from LNbits and also `jq` for parsing the json and `xclip` for handling the clipboard.


### Function
The script checks if there is a string starting with `ln` or `LN` in your clipboard else it will send a notification `not a lightning invoice`.
Then it will run the decoder and preview the beautified json inside a dmenu window. This allows you to search for a key and easily copy it. If you hit enter on the `{` key it will copy the whole json. If you hit ESC, it will not copy anything.


### Dependencies
```console
pacman -S dmenu jq xclip python-pipx
pipx install bolt11
```


### Code
```sh
clipboard=$(xclip -selection clipboard -o)
prefix=$(echo "$clipboard" | cut -c 1-2 | tr '[:upper:]' '[:lower:]')
if [[ "$prefix" != "ln" ]]; then
    notify-send "not a lightning invoice"
    exit 1
fi
decoded=$(bolt11 decode "$clipboard")
[[ -z "$decoded" ]] && notify-send "decoding bolt11 failed" && exit 1
selected=$(echo "$decoded" | jq -r | dmenu -l 30 || exit 1)
[[ -z "$selected" ]] && exit 1
if [[ "$selected" == "{"  ]]; then
    echo $decoded | jq -r | xclip -selection clipboard
else
    echo $selected | xclip -selection clipboard
fi
```


Video
=====
in this video i show using the decoder in my setup, i bound it to `super-c + i`
{% include youtube.html id=page.youtubeId %}


Links:
=====
* [youtube](https://youtu.be/zJefEQVEs8Q)
* [dmenu script](https://github.com/dni/scripts/tree/main/userspace/dmenu.sh#L44)
* [dotfiles](https://github.com/dni/.dotfiles)
* [LNbits bolt11 repository](https://github.com/lnbits/bolt11)
