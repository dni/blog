---
layout: post
title:  "a sxmo inspired rethinking of my dotfiles and also a proof of concept for a new structuring of sxmo-utils"
---

#### Introduction
After sieving through the sxmo-utils, i started thinking about the structure of sxmo and how the scripts are executed and loaded, which lead me into a journey of rethinking my own dotfiles, i'll talk about that in detail in this blogpost. i've rewritten the filepaths and -names to fit in the context of sxmo.

why? to simplify customizing, unclutter the code with unnecessary calls and configs, more reusability and the ease of just being able to overwrite any function by just putting it in `~/.functions` or maybe even a `.sxmorc`?

also to make sxmo-utils readable, configurable and make packaging easier.

everything should be a function! :D

___


#### table of contents
* **part i:** profile
* **part ii:** the magic run_function
* **part iii:** conclusion
* **part iv:** But WHY? why is this even relevant, what are the benefits of it?
* whats next?

___


## **PART I:** /etc/profiles.d/ and the .profile
### the one place the configure everything
to have systemwide access to the functions i decided to go for /etc/profile.d/. when you put a script in there it gets sourced on login, but only once, so if you change anything there you have to relogin.
```sh
-> % cat /etc/profile.d/sxmo.sh
#!/usr/bin/env sh
. /usr/share/sxmo/.profile
```
the .profile consists of 3 files and are sourced from the sxmo dir and also from the home dir if they exist
```sh
-> % cat dotfiles/.profile
#!/usr/bin/env sh
# .env should hold enviroment variable
# .functions should hold functions
# .alias should hold aliases
# also we source potential user configuration, so overwriting is possible
while read -r x; do
  . "$DOTFILES"/"$x"
  [ -e "$HOME"/"$x" ] && . "$HOME"/"$x"
done <<EOF
.env
.aliases
.functions
EOF
```
in the `.functions` file furthers functions are loaded
```sh
#!/usr/bin/env sh
. "$DOTFILES"/scripts/audio.sh
. "$DOTFILES"/scripts/devtools/typo3.sh
. "$DOTFILES"/scripts/devtools/magento2.sh
. "$DOTFILES"/scripts/devtools/webserver.sh
. "$DOTFILES"/scripts/devtools/mysql.sh
. "$DOTFILES"/scripts/dmenu.sh
. "$DOTFILES"/scripts/status.sh
. "$DOTFILES"/scripts/utils.sh
```
one very nice aspect of this is it is possible to group together functions into 1 file in a sensical way,
which makes it for easier to understand and also less code. for example
[audio.sh](https://github.com/dni/dotfiles/blob/sxmo/scripts/audio.sh) which is used in sxhkd config:
[sxhkdrc](https://github.com/dni/dotfiles/blob/sxmo/.config/sxhkd/sxhkdrc#L11).

I did a small example what that could mean for sxmo, as seen here [sxmo_audio.sh](https://github.com/dni/dotfiles/blob/sxmo/scripts/sxmo_audio.sh).
NOTE: also i put the `audiostatus()` also in this file, this was originally inside sxmo_status.sh [here](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_statusbar.sh#L102) and could be just replace with audiostatus there. Now all audio things are neatly tied together into 1 file and can be overwritten anywhere.
Now all of the following files would merge into 1 file:
* [sxmo_audiocurrentdevice.sh](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_audiocurrentdevice.sh)
* [sxmo_audioout.sh](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_audioout.sh)
* [sxmo_vol.sh](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_vol.sh)


also note that we dont need to `. "$(dirname "$0")/sxmo_common.sh"` [here](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_vol.sh#L5)
anymore and also no need to call the function in the and like `"$@"` [here](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_vol.sh#L41)

___


## **PART II:** the magic run_function
### so our functions are sourced via .profile everything should just work, right? no!
a problem i immediatly encountered is that the `.profile` isnt sourced in a non-interactive shell.
which means, running following commands won't work without sourcing the profile again.
to illustrate this point better and also for debugging i introduced the
[hello](https://github.com/dni/dotfiles/blob/sxmo/scripts/utils.sh#L4) function.

```sh
-> % which hello
hello () {
  echo "hello world! you are running a"
  case $- in
    (*i*) echo "interactive" ;;
    (*) echo "non-interactive" ;;
  esac
  tty | grep -q "tty" \
    && echo "tty / (?)login shell" \
    || echo "non-login shell"
}

-> % hello
hello world! you are running a
interactive
non-login shell

-> % sh -c hello
sh: row 1: hello: command not found

-> % sh -c ". $DOTFILES/.functions; hello"
hello world! you are running a
non-interactive
non-login shell

-> % sh -ic "hello"
hello world! you are running a
interactive
non-login shell
```
as you see in the example above `sh -c hello` throws a command not found error, because it is a non-interactive shell.
the command only works when you source the `.functions` again before running the command `sh -c ". $DOTFILES/.functions; hello"`.
similar `sh -ic hello` runs as expected because -i forces interactive shell.  this is a problem because in some cases like
.xinitrc, dmenu, sxhkd and probably many more, it is not clean to force an interactive shell or source our `.functions`
again in each of these cases, so our function will fail in this context.

i do not like sourcing files over and over again, so i came up with, what i think is, a rather clever solution.
first of all i created a bin directory in /usr/share/sxmo which i put into the $PATH in `.profile` so binaries there are found.
binaries can be executed also in a non-interactive context and with this magical
[run_function](https://github.com/dni/dotfiles/blob/sxmo/scripts/run_function) script and symlinks to it, it is
possible to simply run any of our defined functions in a non-interactive way and sourcing is done inside the run_function.

#### [run_function](https://github.com/dni/dotfiles/blob/sxmo/scripts/run_function)
```sh
#!/usr/bin/env sh

# this file is useful for running functions in a non-interactive shell
# (dmenu, xinit, sxhkd). so we need to source our functions here and
# run them, else we end in an infinite loop. functions are not sourced
# in a non-interactive shell so we create symlinks to this script
# in the $PATH and source the profile.

# source .profile with functions and eval
# symlinked binaries in the $PATH are overwritten by the function calls
. "$DOTFILES"/.profile
eval "${0##*/}"
```

#### [create_binaries](https://github.com/dni/dotfiles/blob/sxmo/scripts/utils.sh#L18)
the symlinks are regenerated by calling the function
```sh
# populate ./bin/ with symlink to the magic run_function script
create_binaries() {
  rm "$DOTFILES"/bin/*
  get_functions | while read -r x; do
    ln -sf "$DOTFILES"/scripts/run_function "$DOTFILES"/bin/"$x"
  done
}
```
now after running `create_binaries` function and sourcing, our non working command now runs just fine everywhere, everytime, hopefully :).

```sh
-> % sh -c hello
hello world! you are running a
non-interactive
non-login shell
```

___


## **PART III:** Conclusion
### Now? everything is fine? right?
Well, very probably, i still have some concerns i need to evaluate and im hoping for a feedback from the sxmo people.

Obviously, there is a reason for a non-interactive and an interactive enviroment and why the `.profile` is not
sourced in a non-interactive enviroment. That is, to have a clean enviroment when running in a non-interactive shell
and to not run the risk to somehow break the clean non-interactive enviroment with our .profile. still it is unclear to
me what the risk really is someone would need to assess it, probably, my gut feeling is even when it goes wrong it is not that bad.

that said i still think the PROs heavily outweights the CONs. Especially because the whole concept of sxmo is to
run everything with shell scripts so there is a very good reason to also be able to run functions in a non-interactive way.

___


## **PART IV:** But WHY? why is this even relevant, what are the benefits of it?
Glad you asked :D

### less stuff the package installer has to do
sourcing .env, .aliases and .functions through .profile interactivly and in a non-login/login shell, and non-interactivly through $PATH=sxmo/bin,
helps a lot with packaging because the `MAKEFILE` does not need to take care of putting the sxmoscripts into /usr/bin directory,
they are just linked via the $PATH variable and then run with symlinks via the magic run_function.
Only 1 file, `.profile` in /etc/profile.d/ is needed for bootstrapping.


### cleaner configuration
there is no need anymore for hooks and default configuration like seen [here](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_xinit.sh#L84)
and [envvars](https://git.sr.ht/~mil/sxmo-utils/tree/master/item/scripts/core/sxmo_xinit.sh#L4) in the sxmo_xinit script.
i think if i dig deeper i will find even more things.

### overwriting of functions, now every smxo script could just be a function
this is very useful for multiple reasons. the first and most important reason is, that user now can easily overwrite any function from the sxmo utils without a hassle you would just have to put it in `~/.functions` and it gets sourced and overwrites core function on the fly.

also with this approach if we would do something like a sxmo-extra package with nice to have but not essential tools we could also just overwrite those functions easily in between the user config. so its possible to extend functionality for different functions in a meta package, could be very useful for something like the [sxmo_urlhandler](https://github.com/proycon/sxmo-utils/blob/master/scripts/core/sxmo_urlhandler.sh#L35) and youtube-dl in there, which i see as a non essential-tool and i would put it into sxmo-extra.

### cleaner and simpler code
for me this is a big plus too, because we now can run every of our scripts just by a function call. we could logically group our functions into a file and all of our scripts now do not need the first lines, sourcing sxmo_common, and last lines, calling the function, anymore. i already talked about that in PART I and the [sxmo_audio.sh](https://github.com/dni/dotfiles/blob/sxmo/scripts/sxmo_audio.sh).

___


### proof of concept and dotfiles reworking done
## **what's next?**
First of all i created a new [ticket](https://todo.sr.ht/~mil/sxmo-tickets/297) on the sxmo-tickets page,
please leave a feedback there if you have time.  I want to wait for feedback and how this idea and structure
is received, before i move on trying the port this to the sxmo-utils repository.

In the meantime im still working on simplifying the sxmo-utils package itself, and i think my next blogpost going to be a deep dive into the
packaging of sxmo-utils and also how we could split them up in multiple packages to make it easier to understand, more readable and better structured. my [last blogpost](http://blog.dnilabs.com/2021/06/21/adventures-with-arch-arm-sxmo-pinephone.html) also talks about that.

___


# Links

* [ticket](https://todo.sr.ht/~mil/sxmo-tickets/297)
* [dotfiles](https://github.com/dni/dotfiles/tree/sxmo)
* [run_function](https://github.com/dni/dotfiles/blob/sxmo/scripts/run_function)
* [hello](https://github.com/dni/dotfiles/blob/sxmo/scripts/utils.sh#L4) function
* [sxmo_audio.sh](https://github.com/dni/dotfiles/blob/sxmo/scripts/sxmo_audio.sh).
* [arch-sxmo blogpost](http://blog.dnilabs.com/2021/06/21/adventures-with-arch-arm-sxmo-pinephone.html)
* IRC: #sxmo on irc.oftc.net
