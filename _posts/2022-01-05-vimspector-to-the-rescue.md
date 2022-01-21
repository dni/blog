---
layout: post
title:  "vimspector to the rescue, i finally have a debugger!"
---
---

introduction
============
i just wanna share my experience here, i finally found a debugger in vim i like to use

---

vimspector
=============
its vimspector, i feel like it that kind of debugger i always wanted, with some additional keybind to my vimrc
i'm finally can have the debugging experience i always dreamed of :), im gonna share my binds with you

---

php debugging
=============
as a webdeveloper in my work time i almost exclusively work on php, and php debugging with xdebug is very possible.
i encountered some problems with ubuntu to and i went for compiling vim with latest python 3.8 support. here are my
scripts for using it in a ubuntu dev enviroment. also some stuff on the vimspector side wasn't working mostly due to
me not having node installed, it was also missing from the docs, so did a [PR](https://github.com/puremourning/vimspector/commit/c03d8f9ee137bbf2727ffdb37b8f942c97989638). which was merged :).
here is the php configuration i use for my typo3 projects, .vimspector.json
```
{
  "configurations": {
    "Listen for XDebug": {
      "adapter": "vscode-php-debug",
      "siletypes": [ "php" ],
      "configuration": {
        "name": "Listen for XDebug",
        "type": "php",
        "request": "launch",
        "port": 9003,
        "stopOnEntry": false,
        "pathMappings": {
          "/var/www/project": "${workspaceRoot}/public"
        }
      }
    }
  }
}

```
also before using that i had to use this vim command
```
:VimspectorInstall vscode-php-debug
```

---

python debugging
=============
python debugging for my [lnbits extension]() was a breeze to i just had to use following .vimspector.json configuration
```
{
  "configurations": {
    "Python Attach": {
      "adapter": "multi-session",
      "filetypes": [ "python" ],
      "configuration": {
        "request": "attach",
        "pathMappings": [{
            "localRoot": "${workspaceRoot}",
            "remoteRoot": "/app"
          }]
      }
    }
  }
}
```

---

compiling vim with latest python3.8 support on ubuntu
=====================================================
i sadly had to compile vim and my development ubuntu machines :( and arch it was no problem at all.
* [vim compile script](https://github.com/dni/scripts/blob/main/server/ubuntu/devtools.sh#L9)

---

vim key binds
=========
:) nice binds, i also used maximizer plugin to fullscreen debugging windows
```
" DEBUGGER
nnoremap <leader>dd :call vimspector#Launch()<CR>
nnoremap <leader>dq :call vimspector#Reset()<CR>
nnoremap <leader>d<space> :call vimspector#Continue()<CR>
nnoremap <leader>dc :call win_gotoid(g:vimspector_session_windows.code)<CR>
nnoremap <leader>dt :call win_gotoid(g:vimspector_session_windows.tagpage)<CR>
nnoremap <leader>dv :call win_gotoid(g:vimspector_session_windows.variables)<CR>
nnoremap <leader>dw :call win_gotoid(g:vimspector_session_windows.watches)<CR>
nnoremap <leader>ds :call win_gotoid(g:vimspector_session_windows.stack_trace)<CR>
nnoremap <leader>do :call win_gotoid(g:vimspector_session_windows.output)<CR>
nmap <leader>dh <Plug>VimspectorStepOver
nmap <leader>dj <Plug>VimspectorStepInto
nmap <leader>dk <Plug>VimspectorStepOut
nmap <leader>dl <Plug>VimspectorRunToCursor
nmap <leader>dr <Plug>VimspectorRestart
nmap <leader>db <Plug>VimspectorToggleBreakpoint
nmap <leader>dbc <Plug>VimspectorToggleConditionalBreakpoint

" maximizer
nnoremap <leader>m :MaximizerToggle<CR>
"vnoremap <leader>m :MaximizerToggle<CR>gv
"inoremap <leader>m <C-o>:MaximizerToggle<CR>

```

---

links
=====

* [.vimrc](https://github.com/dni/.dotfiles/blob/main/vim/.vimrc)
* [dotfiles](https://github.com/dni/.dotfiles)
* [ubuntu vim compile script](https://github.com/dni/scripts/blob/main/server/ubuntu/devtools.sh#L9)
* [vimspector](https://github.com/puremourning/vimspector)
* [PR in vimspector](https://github.com/puremourning/vimspector/commit/c03d8f9ee137bbf2727ffdb37b8f942c97989638)
* [lnbits blog](https://blog.dnilabs.com/2022/01/21/bitcoin-lightning-lnbits-sendmail-extension.html)
