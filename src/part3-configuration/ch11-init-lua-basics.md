# Chapter 11: init.lua 시작하기

Part 1과 Part 2에서 Neovim의 편집 기능을 깊이 익혔다. 이제 Neovim을 **나만의 편집기**로 만드는 과정에 들어간다. 그 첫 단계가 바로 설정 파일(init.lua)이다.

설정 파일은 Neovim이 시작될 때 자동으로 실행되는 스크립트다. 여기에 원하는 옵션, 키맵, 플러그인 설정을 작성하면 Neovim이 매번 그 상태로 열린다.

## init.vim vs init.lua: 왜 Lua인가

전통적으로 Vim과 Neovim은 Vimscript라는 자체 언어로 설정했다. 설정 파일 이름도 `init.vim`이었다. Neovim 0.5부터 **Lua**를 공식 설정 언어로 지원하면서 `init.lua`가 등장했다.

### Vimscript의 한계

```vim
" Vimscript 예시
set number
set tabstop=4
nnoremap <leader>ff :Telescope find_files<CR>

" 조건문이 직관적이지 않다
if has('termguicolors')
  set termguicolors
endif

" 함수 정의가 장황하다
function! ToggleNumber()
  if &number
    set nonumber
  else
    set number
  endif
endfunction
```

Vimscript는 편집기 설정에 특화된 언어지만, 범용 프로그래밍 언어로서는 한계가 명확하다. 복잡한 로직을 작성하기 어렵고, 디버깅이 까다로우며, 학습 자료도 제한적이다.

### Lua의 장점

```lua
-- Lua 예시: 같은 작업이 더 명확하다
vim.opt.number = true
vim.opt.tabstop = 4
vim.keymap.set('n', '<leader>ff', ':Telescope find_files<CR>')

-- 조건문이 자연스럽다
if vim.fn.has('termguicolors') == 1 then
  vim.opt.termguicolors = true
end

-- 함수 정의가 간결하다
local function toggle_number()
  vim.opt.number = not vim.opt.number:get()
end
```

Lua를 선택해야 하는 이유:

| 항목 | Vimscript | Lua |
|------|-----------|-----|
| 학습 난이도 | Vim 전용 문법 | 범용 언어, 간결한 문법 |
| 성능 | 인터프리터 방식 | LuaJIT으로 빠른 실행 |
| 생태계 | Vim 플러그인 | 대부분의 최신 플러그인이 Lua 기반 |
| 디버깅 | 제한적 | `vim.inspect`, `print` 등 활용 가능 |
| API 접근 | 간접적 | Neovim API에 직접 접근 |

> **팁**: 기존 Vimscript 설정이 있다면 한꺼번에 전환할 필요 없다. Lua 안에서 `vim.cmd()`를 통해 Vimscript를 실행할 수 있으므로 점진적으로 마이그레이션하면 된다.

## 설정 파일 위치

Neovim은 다음 경로에서 설정 파일을 찾는다:

| 운영체제 | 경로 |
|----------|------|
| Linux / macOS | `~/.config/nvim/init.lua` |
| Windows | `~/AppData/Local/nvim/init.lua` |

`init.vim`과 `init.lua`는 **동시에 사용할 수 없다**. 둘 다 존재하면 에러가 발생한다. 하나만 선택해야 한다.

설정 디렉토리를 처음 만드는 경우:

```bash
# macOS / Linux
mkdir -p ~/.config/nvim
touch ~/.config/nvim/init.lua
```

설정 디렉토리 전체를 확인하려면:

```bash
# Neovim의 설정 경로 확인
nvim --headless -c 'echo stdpath("config")' -c 'quit'
```

Neovim 안에서도 확인할 수 있다:

```vim
:echo stdpath('config')
```

## 설정 디렉토리 구조

설정이 복잡해지면 모든 것을 `init.lua` 하나에 넣는 것은 비효율적이다. Neovim은 `lua/` 디렉토리를 통한 **모듈 시스템**을 제공한다.

### lua/ 디렉토리와 require

`~/.config/nvim/lua/` 디렉토리에 있는 Lua 파일은 `require()`로 불러올 수 있다:

```lua
-- ~/.config/nvim/lua/config/options.lua 파일을 불러온다
require('config.options')

-- 디렉토리 구분자는 '.'을 사용한다 ('/'가 아님)
-- .lua 확장자는 생략한다
```

`require()`의 동작 방식:

1. `require('config.options')` 호출
2. `~/.config/nvim/lua/config/options.lua` 파일을 찾음
3. 또는 `~/.config/nvim/lua/config/options/init.lua` 파일을 찾음
4. 파일을 실행하고 반환값을 캐싱

> **팁**: `require()`는 한 번 실행되면 결과를 캐싱한다. 설정을 수정한 뒤 반영하려면 Neovim을 재시작하거나, 캐시를 비워야 한다. 개발 중에는 `:source %`로 현재 파일을 다시 실행할 수 있지만, 모듈 캐싱 때문에 완전한 재적용은 Neovim 재시작이 확실하다.

### 추천 디렉토리 구조

```
~/.config/nvim/
├── init.lua                 -- 진입점: 각 모듈을 require
└── lua/
    ├── config/
    │   ├── options.lua      -- vim.opt 설정 (옵션)
    │   ├── keymaps.lua      -- vim.keymap.set 설정 (키맵)
    │   └── autocmds.lua     -- 자동 명령 (autocommands)
    └── plugins/
        └── init.lua         -- 플러그인 매니저 설정 (Part 4)
```

이 구조의 장점:

- **관심사 분리**: 옵션, 키맵, 자동 명령이 각각의 파일에 있어 찾기 쉽다
- **유지보수 용이**: 특정 설정을 수정할 때 해당 파일만 열면 된다
- **확장성**: 플러그인 설정이 늘어나도 디렉토리 구조로 관리할 수 있다
- **버전 관리**: Git으로 추적할 때 변경 이력이 명확하다

## 기본 init.lua 작성

이제 실제로 `init.lua`를 작성해 보자.

### 최소한의 init.lua

```lua
-- ~/.config/nvim/init.lua

-- 기본 옵션
vim.opt.number = true          -- 줄 번호 표시
vim.opt.relativenumber = true  -- 상대 줄 번호
vim.opt.tabstop = 4            -- 탭 너비 4칸
vim.opt.shiftwidth = 4         -- 들여쓰기 너비 4칸
vim.opt.expandtab = true       -- 탭을 공백으로 변환
vim.opt.mouse = 'a'            -- 마우스 사용
vim.opt.clipboard = 'unnamedplus'  -- 시스템 클립보드 연동

-- 검색 설정
vim.opt.ignorecase = true      -- 검색 시 대소문자 무시
vim.opt.smartcase = true       -- 대문자 입력 시 대소문자 구분

-- Leader 키 설정 (다른 키맵보다 먼저 설정해야 한다)
vim.g.mapleader = ' '
vim.g.maplocalleader = ' '
```

이 파일 하나만으로도 Neovim이 훨씬 사용하기 편해진다.

### 모듈로 분리한 init.lua

설정이 늘어나면 모듈로 분리한다:

```lua
-- ~/.config/nvim/init.lua

-- Leader 키는 다른 모듈보다 먼저 설정
vim.g.mapleader = ' '
vim.g.maplocalleader = ' '

-- 각 설정 모듈 로딩
require('config.options')    -- 옵션 설정
require('config.keymaps')    -- 키맵 설정
require('config.autocmds')   -- 자동 명령
```

```lua
-- ~/.config/nvim/lua/config/options.lua

local opt = vim.opt

-- 화면 표시
opt.number = true
opt.relativenumber = true
opt.signcolumn = 'yes'
opt.cursorline = true

-- 편집
opt.tabstop = 4
opt.shiftwidth = 4
opt.expandtab = true
opt.smartindent = true

-- 검색
opt.ignorecase = true
opt.smartcase = true
opt.hlsearch = true
opt.incsearch = true
```

```lua
-- ~/.config/nvim/lua/config/keymaps.lua

local keymap = vim.keymap.set

-- 기본 키맵 예시
keymap('n', '<Esc>', ':nohlsearch<CR>', { desc = '검색 하이라이트 끄기' })
keymap('n', '<leader>w', ':w<CR>', { desc = '파일 저장' })
```

```lua
-- ~/.config/nvim/lua/config/autocmds.lua

-- 복사 시 하이라이트
vim.api.nvim_create_autocmd('TextYankPost', {
  callback = function()
    vim.highlight.on_yank()
  end,
})
```

> **팁**: 각 모듈 파일에서 자주 쓰는 API를 로컬 변수에 할당하면 코드가 간결해진다. `local opt = vim.opt`, `local keymap = vim.keymap.set` 같은 패턴은 Neovim 설정에서 매우 흔하다.

## Vimscript와 Lua 혼용: vim.cmd()

기존 Vimscript 설정이나 명령을 Lua 안에서 실행해야 할 때 `vim.cmd()`를 사용한다.

### 단일 명령 실행

```lua
-- 컬러스킴 적용
vim.cmd('colorscheme desert')

-- Ex 명령 실행
vim.cmd('syntax on')

-- 약어(abbreviation) 설정
vim.cmd('cnoreabbrev W w')
vim.cmd('cnoreabbrev Q q')
```

### 여러 줄의 Vimscript 실행

```lua
-- [[ ]] 문법으로 여러 줄 작성
vim.cmd([[
  augroup MyGroup
    autocmd!
    autocmd FileType python setlocal tabstop=4
    autocmd FileType lua setlocal tabstop=2
  augroup END
]])
```

### vim.cmd와 Lua API 비교

대부분의 Vimscript 설정은 Lua API로 대체할 수 있다:

| Vimscript (vim.cmd) | Lua API |
|---------------------|---------|
| `vim.cmd('set number')` | `vim.opt.number = true` |
| `vim.cmd('nnoremap ...')` | `vim.keymap.set('n', ...)` |
| `vim.cmd('autocmd ...')` | `vim.api.nvim_create_autocmd(...)` |
| `vim.cmd('highlight ...')` | `vim.api.nvim_set_hl(...)` |
| `vim.cmd('colorscheme ...')` | `vim.cmd.colorscheme('...')` (Lua 스타일) |

가능하면 Lua API를 사용하는 것이 좋다. 타입 검사, 자동 완성, 에러 메시지가 더 명확하기 때문이다. 하지만 복잡한 하이라이트 설정이나 레거시 플러그인 호출 등에서는 `vim.cmd()`가 더 편리한 경우도 있다.

### vim.cmd의 함수 호출 스타일

Neovim은 `vim.cmd`를 함수 호출 스타일로도 사용할 수 있다:

```lua
-- 문자열 스타일
vim.cmd('colorscheme desert')

-- 함수 호출 스타일 (동일한 동작)
vim.cmd.colorscheme('desert')
vim.cmd.write()          -- :write
vim.cmd.quit()           -- :quit
vim.cmd.edit('file.lua') -- :edit file.lua
```

## 설정 적용과 확인

### 설정 파일 열기

Neovim 안에서 설정 파일을 빠르게 열 수 있다:

```vim
:edit $MYVIMRC
```

또는 Lua에서:

```lua
-- init.lua에 추가하면 <leader>vc로 설정 파일을 열 수 있다
vim.keymap.set('n', '<leader>vc', ':edit $MYVIMRC<CR>', { desc = 'init.lua 열기' })
```

### 설정 다시 로딩

설정을 수정한 뒤 적용하는 방법:

```vim
:source $MYVIMRC        " init.lua 다시 실행
:source %               " 현재 파일 다시 실행
:lua require('config.options')  " 특정 모듈 실행 (캐시 주의)
```

> **팁**: 가장 확실한 방법은 Neovim을 종료했다 다시 여는 것이다. 설정 개발 초기에는 이 방법이 가장 안전하다.

### 현재 옵션 값 확인

```vim
:set number?            " number 옵션의 현재 값 확인
:set tabstop?           " tabstop 값 확인
:lua print(vim.opt.number:get())  " Lua에서 확인
:lua print(vim.inspect(vim.opt.completeopt:get()))  " 테이블 값 확인
```

## 실습: 첫 번째 init.lua 만들기

단계별로 init.lua를 만들어 보자.

### 단계 1: 디렉토리 생성

```bash
mkdir -p ~/.config/nvim/lua/config
```

### 단계 2: init.lua 작성

```lua
-- ~/.config/nvim/init.lua

-- Leader 키 설정 (가장 먼저!)
vim.g.mapleader = ' '
vim.g.maplocalleader = ' '

-- 모듈 로딩
require('config.options')
require('config.keymaps')
require('config.autocmds')
```

### 단계 3: options.lua 작성

```lua
-- ~/.config/nvim/lua/config/options.lua

local opt = vim.opt

-- 줄 번호
opt.number = true
opt.relativenumber = true

-- 들여쓰기
opt.tabstop = 4
opt.shiftwidth = 4
opt.expandtab = true
opt.smartindent = true

-- 검색
opt.ignorecase = true
opt.smartcase = true
opt.hlsearch = true

-- 화면
opt.termguicolors = true
opt.signcolumn = 'yes'
opt.cursorline = true
opt.scrolloff = 8

-- 기타
opt.mouse = 'a'
opt.clipboard = 'unnamedplus'
opt.undofile = true
opt.swapfile = false
```

### 단계 4: keymaps.lua 작성

```lua
-- ~/.config/nvim/lua/config/keymaps.lua

local keymap = vim.keymap.set
local opts = { noremap = true, silent = true }

-- 검색 하이라이트 끄기
keymap('n', '<Esc>', ':nohlsearch<CR>', opts)

-- 파일 저장
keymap('n', '<leader>w', ':w<CR>', { desc = '파일 저장' })
```

### 단계 5: autocmds.lua 작성

```lua
-- ~/.config/nvim/lua/config/autocmds.lua

-- 복사 시 하이라이트
vim.api.nvim_create_autocmd('TextYankPost', {
  callback = function()
    vim.highlight.on_yank({ timeout = 200 })
  end,
})
```

### 단계 6: Neovim을 열고 확인

```bash
nvim
```

에러 없이 열리면 성공이다. `:set number?`로 옵션이 적용되었는지 확인하고, `y`로 텍스트를 복사해서 하이라이트가 나타나는지 테스트해 보자.

연습 과제:
1. `options.lua`에 `wrap = false` 옵션을 추가하고 효과를 확인하라
2. `keymaps.lua`에 `<leader>q`를 `:q<CR>`로 매핑하라
3. `:edit $MYVIMRC`로 설정 파일을 열고, `:source %`로 다시 로딩하라
4. `vim.cmd('colorscheme')`을 사용해 다른 컬러스킴을 적용하라 (예: `vim.cmd('colorscheme habamax')`)
5. 현재 설정 디렉토리 구조를 터미널에서 `tree` 또는 `find`로 확인하라

## 핵심 정리

- Neovim의 설정 파일은 **`~/.config/nvim/init.lua`**에 위치한다
- **Lua**는 Vimscript보다 간결하고, 빠르며, 최신 플러그인 생태계와 호환된다
- `lua/` 디렉토리와 **`require()`**를 사용해 설정을 모듈로 분리한다
- 추천 구조: `config/options.lua`, `config/keymaps.lua`, `config/autocmds.lua`
- **`vim.cmd()`**로 Vimscript를 Lua 안에서 실행할 수 있다 (점진적 마이그레이션 가능)
- **Leader 키 설정**은 다른 키맵보다 먼저 해야 한다
- 설정 수정 후 적용: `:source %` 또는 Neovim 재시작
