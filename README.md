vim-clurin
=====================

[![Build Status](https://travis-ci.org/uplus/vim-clurin.svg?branch=master)](https://travis-ci.org/uplus/vim-clurin)

# difference from original
- Support multiple filetype key  
    `'c cpp': {'def': [ ... ]}`
- Support `'use_default'` for each filetype  
    You can disable default config of each filetype  
    `'c': {'use_default': 0, 'def': [ ... ]}`
- Support `'quit'` option  
    You can quit substitution in `'replace'` function  
    `'ruby': {'quit': 1, 'group': ['pattern': '...', 'replace': function(RepFunc)]}`
- Support multiple subexpression substitution  
    The original can use only `'\1'` in `'replace'`  
    This fork can use all

# usage

```vim
nmap + <Plug>(clurin-next)
nmap - <Plug>(clurin-prev)
vmap + <Plug>(clurin-next)
vmap - <Plug>(clurin-prev)
```

# customize

```
g:clurin: Dictionary of Dictionary. key: - or 'filetype' or 'filetype1 filetype2 ...'
	def			(List of GROUP, required)
	use_default		(Bool, 1)
	use_default_user	(Bool, 1)
	nomatch  (Funcref({cnt}), none)
	jump     (Bool, 0)  0: under the cursor, otherwise: after the cursor

GROUP:: Dictionary or group
	cyclic		(Bool, 1)
  quit			(Bool, 0) Quit replace in function
	group			(List of Dictionary or Strings)
		Dictionary:
			pattern		(String)
			replace		(String or Funcref({str},{cnt},{def}))
				{str}: matched text
				{cnt}: count
				{def}: normalized def.
			String {s}
				string. NOTE: cannot use regexp .
				 -> {'pattern': '\<\({s}\)\>', 'replace' '{s}'} ({s}='^\k\+$') or
				    {'pattern': '\({s}\)', 'replace' '{s}'}
  
b:clurin: Buffer local List of GROUP
```

## example

```vim
function! g:CountUp(strs, cnt, def) abort
  " a:strs: matched text list
  " a:cnt: non zero.
  " a:def: definition
  return str2nr(a:strs[0]) + a:cnt
endfunction

function! g:CtrlAX(cnt) abort
	if a:cnt >= 0
		execute 'normal!' a:cnt . "\<C-A>"
	else
		execute 'normal!' (-a:cnt) . "\<C-X>"
	endif
endfunction

" for RSpec
au BufRead *_spec.rb let b:clurin = [
      \   ['be_truthy', 'be_falsey'],
      \   [{'pattern': '\.to_not ', 'replace': '.to '}]
      \ ]

let g:clurin = {
\ '-': { 'def': [[
\       {'pattern': '\(-\?\d\+\)', 'replace': function('g:CountUp')},
\     ], [
\       {'pattern': '\<true\>', 'replace': 'true'},
\       {'pattern': '\<false\>', 'replace': 'false'},
\     ], [
\       'on', 'off'
\     ]]},
\ 'vim': {'def': [[
\       {'pattern': '''\(\k\+\)''', 'replace': '''\1'''},
\       {'pattern': '"\(\k\+\)"', 'replace': '"\1"'},
\     ]]},
\ 'c cpp' : {'def': [
\     [ '&&', '||' ],
\     [
\       {'pattern': '\(\k\+\)\.', 'replace': '\1.'},
\       {'pattern': '\(\k\+\)->', 'replace': '\1->'},
\     ]], 'nomatch': function('g:CtrlAX')},
\}
```


## receipt

ruby string: 'string' -> "string" -> :string
```vim
\     [
\       {'pattern': '''\(\k\+\)''', 'replace': '''\1'''},
\       {'pattern': '"\(\k\+\)"', 'replace': '"\1"'},
\       {'pattern': ':\(\k\+\)"', 'replace': ':\1'},
\     ]
```

closure string: 'string -> "string" -> :string
```vim
\     [
\       {'pattern': '''\(\k\+\)', 'replace': '''\1'},
\       {'pattern': '"\(\k\+\)"', 'replace': '"\1"'},
\       {'pattern': ':\(\k\+\)"', 'replace': ':\1'},
\     ]
```

vim dictionary: ['key'] -> ["key"] -> .key
```vim
\     [
\       {'pattern': '\[''\(\k\+\)''\]', 'replace': '[''\1'']'},
\       {'pattern': '\["\(\k\+\)"\]',   'replace': '["\1"]'},
\       {'pattern': '\.\(\k\+\)',       'replace': '.\1'},
\     ]
```

### more 
- [gist](https://gist.github.com/uplus/0739fec00b425d4a19bdcbf3cd3c9ca1)


# FAQ
Q: I want to new line.  
A: vim-clurin call setline() in internal. It cannot new line. You need to define function and to call it via `'replace'`.  

```vim
function! g:NewLine(strs, cnt, def) abort 
  call append('.', '')  
endfunction

let g:clurin = {'-': {'def': [
      \   {'quit': 1, 'group': [{'pattern': 'abc', 'replace': function('g:NewLine')}]},
      \ ]}
      \ }
```

Q: I want to multiple line substitution.  
A: Please look for ruby's config of [gist](https://gist.github.com/uplus/0739fec00b425d4a19bdcbf3cd3c9ca1)  

Q: I want to undo at once  
A: I recommend to use [vim-submode](https://github.com/kana/vim-submode)  

```vim
let keymap = '!'
nnoremap <silent><Plug>(undojoin) :<c-u>undojoin<cr>
nmap <silent><Plug>(clurin-undojoin) <Plug>(undojoin)<Plug>(clurin-next)
call submode#enter_with('clurin', 'n', 'r', keymap, '<Plug>(clurin-next)')
call submode#map('clurin', 'n', 'r', keymap, '<Plug>(clurin-undojoin)')
```


# Similar work

- [switch.vim](https://github.com/AndrewRadev/switch.vim)
- [toggle.vim](http://www.vim.org/scripts/script.php?script_id=895)
- [cycle.vim](https://github.com/zef/vim-cycle)

