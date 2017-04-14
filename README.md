minpac: A minimal package manager for Vim 8 (and NeoVim)
========================================================

Overview
--------

Minpac is a minimal package manager for Vim 8 (and NeoVim). This uses the
[packages](http://vim-jp.org/vimdoc-en/repeat.html#packages) feature and
the [jobs](http://vim-jp.org/vimdoc-en/channel.html#job-channel-overview)
feature which have been newly added on Vim 8.

Concept
-------

* Utilize Vim 8's packages feature.
* Parallel install/update using Vim 8's jobs feature.
* Simple.
* Fast.


Requirements
------------

* Vim 8.0.0050 or later  
  (Hopefully minpac will also work on NeoVim.)
* Git 1.9 or later
* OS  
  Windows: tested  
  Linux: tested  
  macOS: not tested


Installation
------------

Minpac should be installed under `pack/minpac/opt/` in the first directory
in the `'packpath'` option.
Plugins installed under `pack/*/start/` are automatically added to the `'runtimepath'` after `.vimrc` is sourced. However, minpac needs to be loaded before that. Therefore, minpac should be installed under "opt" directory, and should be loaded using `packadd minpac`.

### Windows

```cmd
cd /d %USERPROFILE%
mkdir vimfiles\pack\minpac\opt
cd vimfiles\pack\minpac\opt
git clone https://github.com/k-takata/minpac.git
```

### Linux, macOS

```sh
mkdir -p ~/.vim/pack/minpac/opt
cd ~/.vim/pack/minpac/opt
git clone https://github.com/k-takata/minpac.git
```

### Sample .vimrc

#### Basic sample

```vim
packadd minpac

call minpac#init()

" minpac must have {'type': 'opt'} so that it can be loaded with `packadd`.
call minpac#add('k-takata/minpac', {'type': 'opt'})

" Add other plugins here.
call minpac#add('vim-jp/syntax-vim-ex')
...

" Load the plugins right now. (optional)
"packloadall
```

#### Customizing 'packpath'

If you want to use `.vim` directory instead of `vimfiles` even on Windows,
you should add `~/.vim` on top of `'packpath'`:

```vim
set packpath^=~/.vim
packadd minpac

call minpac#init()
...
```

#### Advanced sample

You can write a .vimrc which can be also used even if minpac is not installed.

```vim
" Try to load minpac.
silent! packadd minpac

if !exists('*minpac#init')
  " minpac is not available.

  " Settings for plugin-less environment.
  ...
else
  " minpac is available.
  call minpac#init()
  call minpac#add('k-takata/minpac', {'type': 'opt'})

  " Additional plugins here.
  ...

  " Plugin settings here.
  ...
endif

" Common settings here.
...
```

#### Load minpac on demand

Very interestingly, minpac doesn't need to be loaded every time. Unlike other plugin managers, it is needed only when updating, installing or cleaning the plugins. This is because minpac itself doesn't handle the runtime path.
You can define a user command to load minpac, reload .vimrc to register the information of plugins, then call `minpac#update()` or `minpac#clean()`.

```vim
" For a paranoia.
" Normally `:set nocp` is not needed, because it is done automatically
" when .vimrc is found.
if &compatible
  " `:set nocp` has many side effects. Therefore this should be done
  " only when 'compatible' is set.
  set nocompatible
endif

if exists('*minpac#init')
  " minpac is loaded.
  call minpac#init()
  call minpac#add('k-takata/minpac', {'type': 'opt'})

  " Additional plugins here.
  call minpac#add('vim-jp/syntax-vim-ex')
  ...
endif

" Plugin settings here.
...

" Define user commands for updating/cleaning the plugins.
" Each of them loads minpac, reloads .vimrc to register the
" information of plugins, then performs the task.
command! PackUpdate packadd minpac | source $MYVIMRC | call minpac#update()
command! PackClean  packadd minpac | source $MYVIMRC | call minpac#clean()
```

Note that your .vimrc must be reloadable to use this. E.g.:

* `:set nocompatible` should not be executed twice to avoid side effects.
* `:function!` should be used to define a user function.
* `:command!` should be used to define a user command.
* `:augroup!` should be used properly to avoid the same autogroups are defined twice.


Usage
-----

### Commands

Minpac doesn't provide any commands. Use the `:call` command to call minpac
functions. E.g.:

```vim
" To install or update plugins:
call minpac#update()

" To uninstall unused plugins:
call minpac#clean()
```


### Functions

#### minpac#init([{opt}])

Initialize minpac.

`{opt}` is a Dictionary which specifies options.

| option    | description |
|-----------|-------------|
| `'dir'`   | Base directory. Default: the first directory of the `'packpath'` option. |
| `'package_name'` | Package name. Default: `'minpac'` |
| `'git'`   | Git command. Default: `'git'` |
| `'depth'` | Default clone depth. Default: 1 |
| `'jobs'`  | Maximum job numbers. Default: 8 |
| `'verbose'` | If > 0, show verbose log. Default: 0 |

All plugins will be installed under the following directories:

    "start" plugins: <dir>/pack/<package_name>/start/<plugin_name>
    "opt" plugins:   <dir>/pack/<package_name>/opt/<plugin_name>


"start" plugins will be automatically loaded after processing your `.vimrc`, or you can load them explicitly using `:packloadall` command.
"opt" plugins can be loaded with `:packadd` command.
See `:help packages` for detail.

#### minpac#add({url}, [{opt}])

Register a plugin.

`{url}` is a URL of a plugin. It can be a short form (`'<github-account>/<repository>'`) or a valid git URL.

`{opt}` is a Dictionary which specifies options.

| option     | description |
|------------|-------------|
| `'name'`   | Unique name of the plugin (`plugin_name`). Also used as a local directory name. Default: derived from the repository name. |
| `'type'`   | Type of the plugin. `'start'` or `'opt'`. Default: `'start'` |
| `'frozen'` | If 1, the plugin will not be updated automatically. Default: 0 |
| `'depth'`  | If >= 1, it is used as a depth to be cloned. Default: 1 or specified value by `minpac#init()`. |
| `'branch'` | Used as a branch name to be cloned. Default: empty |
| `'do'`     | Post-update hook. See [Post-update hooks](#post-update-hooks). Default: empty |

#### minpac#update([{name}])

Install or update all plugins or the specified plugin.

`{name}` is a unique name of a plugin (`plugin_name`).

If `{name}` is omitted, all plugins will be installed or updated. Frozen plugins will be installed, but it will not be updated.

If `{name}` is specified, only specified plugin will be installed or updated. Frozen plugin will be also updated.
`{name}` can also be a list of plugin names.

You can check the results with `:message` command.

Note: This resets the 'more' option temporarily to avoid jobs being interrupted.

#### minpac#clean([{name}])

Remove all plugins which are not registered, or remove the specified plugin.

`{name}` is a name of a plugin. It can be a unique plugin name (`plugin_name`) or a plugin name with wildcards.

If `{name}` is omitted, all plugins under the `minpac` directory will be checked. If unregistered plugins are found, they are listed and a prompt is shown. If you type `y`, they will be removed.

If `{name}` is specified, matched plugins are listed (even they are registered with `minpac#add()`) and a prompt is shown. If you type `y`, they will be removed.
`{name}` can also be a list of plugin names.

#### minpac#getpluginfo({name})

Get information of specified plugin.

`{name}` is a unique name of a plugin (`plugin_name`).
A dictionary with following items will be returned:

| item       | description |
|------------|-------------|
| `'url'`    | URL of the plugin repository.  |
| `'dir'`    | Local directory of the plugin. |
| `'frozen'` | If 1, the plugin is frozen. |
| `'type'`   | Type of the plugin. |
| `'branch'` | Branch name to be cloned. |
| `'depth'`  | Depth to be cloned. |


### Hooks

Currently, minpac supports only one type of hook: Post-update hooks.


#### Post-update hooks

If a plugin requires extra works (e.g. building a native module), you can use the post-update hooks.

You can specify the hook with the `'do'` item in the option of the `minpac#add()` function. It can be a String or a Funcref.
If a String is specified, it is executed as an Ex command.
If a Funcref is specified, it is called with two arguments; `hooktype` and `name`.

| item       | description |
|------------|-------------|
| `hooktype` | Type of the hook. `'post-update'` for post-update hooks.  |
| `name`     | Unique name of the plugin. (`plugin_name`) |

The current directory is set to the directory of the plugin, when the hook is invoked.

E.g.:

```vim
" Execute an Ex command as a hook.
call minpac#add('Shougo/vimproc.vim', {'do': 'silent! !make'})

" Execute a lambda function as a hook.
" Parameters for a lambda can be omitted, if you don't need them.
call minpac#add('Shougo/vimproc.vim', {'do': {-> system('make')}})

" Of course, you can also use a normal user function as a hook.
function! s:hook(hooktype, name)
  echom a:hooktype
  " You can use `minpac#getpluginfo()` to get the information about
  " the plugin.
  echom 'Directory:' minpac#getpluginfo(a:name).dir
  call system('make')
endfunction
call minpac#add('Shougo/vimproc.vim', {'do': function('s:hook')})
```

The above examples execute the "make" command synchronously. If you want to execute an external command asynchronously, you should use the `job_start()` function on Vim 8 or the `jobstart()` function on NeoVim.
You may also want to use the `minpac#job#start()` function, but this is mainly for internal use and the specification is subject to change without notice.


Credit
------

Prabir Shrestha (as the author of [async.vim](https://github.com/prabirshrestha/async.vim))


License
-------

VIM License

(`autoload/minpac/job.vim` is the MIT License.)
