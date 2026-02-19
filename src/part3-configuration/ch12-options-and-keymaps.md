# Chapter 12: 옵션과 키맵

Chapter 11에서 init.lua의 기본 구조를 만들었다. 이 챕터에서는 Neovim의 두 가지 핵심 설정 요소인 **옵션(options)**과 **키맵(keymaps)**을 깊이 다룬다. 옵션은 Neovim의 동작 방식을 결정하고, 키맵은 나만의 단축키를 정의한다.

## vim.opt 사용법

Lua에서 Neovim 옵션을 설정하는 가장 일반적인 방법은 `vim.opt`다.

```lua
-- boolean 옵션
vim.opt.number = true          -- :set number
vim.opt.relativenumber = true  -- :set relativenumber
vim.opt.wrap = false           -- :set nowrap

-- 숫자 옵션
vim.opt.tabstop = 4            -- :set tabstop=4
vim.opt.scrolloff = 8          -- :set scrolloff=8

-- 문자열 옵션
vim.opt.signcolumn = 'yes'     -- :set signcolumn=yes
vim.opt.mouse = 'a'            -- :set mouse=a
```

### 리스트, 맵, 셋 옵션

일부 옵션은 여러 값을 가진다. `vim.opt`는 이런 옵션을 편리하게 다룰 수 있다:

```lua
-- 리스트 옵션: append, prepend, remove
vim.opt.wildignore:append({ '*.o', '*.pyc' })  -- 값 추가
vim.opt.wildignore:remove({ '*.o' })            -- 값 제거

-- 셋(set) 옵션
vim.opt.completeopt = { 'menu', 'menuone', 'noselect' }

-- 맵(map) 옵션
vim.opt.listchars = { tab = '» ', trail = '·', nbsp = '␣' }
```

### 옵션 값 읽기

```lua
-- :get()으로 현재 값을 가져온다
local tabstop = vim.opt.tabstop:get()     -- 숫자 반환: 4
local number = vim.opt.number:get()        -- boolean 반환: true
local completeopt = vim.opt.completeopt:get()  -- 테이블 반환

-- 디버깅 시 유용
print(vim.inspect(vim.opt.completeopt:get()))
```

## vim.opt vs vim.o vs vim.bo vs vim.wo

Neovim은 옵션을 설정하는 여러 API를 제공한다. 차이를 이해해야 올바르게 사용할 수 있다.

| API | 설명 | Vimscript 대응 | 특징 |
|-----|------|----------------|------|
| `vim.opt` | 범용 옵션 설정 | `:set` | 리스트/맵 조작 메서드 제공 |
| `vim.o` | 전역 옵션 | `:set` | 값을 문자열로 다룸 |
| `vim.bo` | 버퍼 로컬 옵션 | `:setlocal` (버퍼) | 특정 버퍼에만 적용 |
| `vim.wo` | 윈도우 로컬 옵션 | `:setlocal` (윈도우) | 특정 윈도우에만 적용 |
| `vim.go` | 전역 옵션만 | `:setglobal` | 전역 값만 변경 |

```lua
-- vim.opt: 가장 일반적. `:set`처럼 전역 + 로컬 모두 설정
vim.opt.tabstop = 4

-- vim.o: vim.opt와 비슷하지만 값이 항상 문자열/숫자/boolean
vim.o.tabstop = 4

-- vim.bo: 현재 버퍼에만 적용 (버퍼 로컬 옵션)
vim.bo.filetype = 'lua'

-- vim.wo: 현재 윈도우에만 적용 (윈도우 로컬 옵션)
vim.wo.number = true

-- 특정 버퍼/윈도우 지정
vim.bo[bufnr].tabstop = 2       -- 특정 버퍼 번호의 옵션
vim.wo[winnr].cursorline = true  -- 특정 윈도우 번호의 옵션
```

> **팁**: 대부분의 경우 `vim.opt`를 사용하면 충분하다. `vim.bo`와 `vim.wo`는 autocommand나 플러그인 개발에서 특정 버퍼/윈도우의 옵션을 제어할 때 필요하다.

## 주요 옵션 카테고리별 설명

### 화면 표시 옵션

```lua
local opt = vim.opt

-- 줄 번호
opt.number = true           -- 절대 줄 번호 표시
opt.relativenumber = true   -- 상대 줄 번호 표시 (현재 줄은 절대 번호)

-- 커서 강조
opt.cursorline = true       -- 현재 줄 하이라이트
-- opt.cursorcolumn = true  -- 현재 열 하이라이트 (성능 영향 있음)

-- 사이드 컬럼
opt.signcolumn = 'yes'      -- 사인 컬럼 항상 표시 (Git, 진단 표시용)
opt.numberwidth = 4         -- 줄 번호 컬럼 최소 너비

-- 줄바꿈
opt.wrap = false            -- 긴 줄 자동 줄바꿈 끄기
opt.linebreak = true        -- 줄바꿈 시 단어 단위로 (wrap이 true일 때)
opt.breakindent = true      -- 줄바꿈된 줄에 들여쓰기 유지

-- 스크롤 여백
opt.scrolloff = 8           -- 커서 위아래로 최소 8줄 여백 유지
opt.sidescrolloff = 8       -- 커서 좌우로 최소 8칸 여백 유지

-- 색상
opt.termguicolors = true    -- 24비트 색상 지원 (대부분의 터미널에서 필요)

-- 보이지 않는 문자 표시
opt.list = true
opt.listchars = { tab = '» ', trail = '·', nbsp = '␣' }

-- 상태줄
opt.laststatus = 3          -- 전역 상태줄 (하나만 표시)
opt.showmode = false        -- 모드 표시 끄기 (상태줄 플러그인 사용 시)

-- 분할
opt.splitright = true       -- 수직 분할 시 오른쪽에
opt.splitbelow = true       -- 수평 분할 시 아래에
```

### 편집 옵션

```lua
-- 클립보드
opt.clipboard = 'unnamedplus'  -- 시스템 클립보드와 연동

-- 마우스
opt.mouse = 'a'             -- 모든 모드에서 마우스 사용

-- 백스페이스
opt.backspace = { 'indent', 'eol', 'start' }  -- 자유로운 백스페이스

-- Undo
opt.undofile = true         -- undo 히스토리를 파일로 저장
opt.undolevels = 10000      -- undo 최대 횟수

-- 스왑 파일
opt.swapfile = false        -- 스왑 파일 생성 끄기
opt.backup = false          -- 백업 파일 생성 끄기

-- 자동 읽기
opt.autoread = true         -- 외부에서 파일 변경 시 자동 반영

-- 파일 인코딩
opt.fileencoding = 'utf-8'  -- 파일 인코딩 UTF-8

-- 업데이트 시간
opt.updatetime = 250        -- CursorHold 이벤트 대기 시간 (ms)
opt.timeoutlen = 300        -- 키 시퀀스 대기 시간 (ms)
```

### 검색 옵션

```lua
-- 대소문자
opt.ignorecase = true       -- 검색 시 대소문자 무시
opt.smartcase = true        -- 대문자 포함 시 대소문자 구분

-- 하이라이트
opt.hlsearch = true         -- 검색 결과 하이라이트
opt.incsearch = true        -- 입력 중 실시간 검색

-- 정규식
opt.magic = true            -- 정규식에서 특수문자 기본 활성화
```

### 들여쓰기 옵션

```lua
-- 탭과 공백
opt.tabstop = 4             -- 탭 문자의 화면 표시 너비
opt.softtabstop = 4         -- 편집 시 탭 키의 동작 너비
opt.shiftwidth = 4          -- 자동 들여쓰기 너비
opt.expandtab = true        -- 탭 키를 공백으로 변환

-- 자동 들여쓰기
opt.smartindent = true      -- 문법 기반 자동 들여쓰기
opt.autoindent = true       -- 이전 줄의 들여쓰기 유지
opt.shiftround = true       -- 들여쓰기를 shiftwidth의 배수로 맞춤
```

> **팁**: `tabstop`, `softtabstop`, `shiftwidth`는 보통 같은 값으로 설정한다. 프로젝트 규칙에 따라 2 또는 4가 일반적이다.

### 기타 유용한 옵션

```lua
-- 자동 완성
opt.completeopt = { 'menu', 'menuone', 'noselect' }
opt.pumheight = 10          -- 자동 완성 팝업 최대 높이

-- 검색 경로
opt.path:append('**')       -- :find 명령에서 하위 디렉토리 검색

-- 숨김 버퍼
opt.hidden = true           -- 저장하지 않은 버퍼를 숨길 수 있음

-- 명령줄 자동 완성
opt.wildmode = 'longest:full,full'
opt.wildignore:append({ '*.o', '*.obj', '*.pyc', 'node_modules/**' })

-- 접기(folding) 기본 설정
opt.foldmethod = 'indent'   -- 들여쓰기 기반 접기
opt.foldlevel = 99          -- 시작 시 모든 접기를 펼침
```

## vim.keymap.set 사용법

키맵은 특정 키 조합에 원하는 동작을 연결하는 것이다. Lua에서는 `vim.keymap.set()`을 사용한다.

### 기본 문법

```lua
vim.keymap.set(mode, lhs, rhs, opts)
```

| 매개변수 | 설명 | 예시 |
|----------|------|------|
| `mode` | 동작할 모드 | `'n'`, `'i'`, `'v'`, `'x'`, `'t'`, `{'n', 'v'}` |
| `lhs` | 입력할 키 조합 | `'<leader>w'`, `'jk'`, `'<C-h>'` |
| `rhs` | 실행할 동작 | `':w<CR>'`, Lua 함수 |
| `opts` | 옵션 테이블 | `{ desc = '설명', silent = true }` |

### 모드 문자

| 문자 | 모드 |
|------|------|
| `'n'` | Normal |
| `'i'` | Insert |
| `'v'` | Visual + Select |
| `'x'` | Visual만 |
| `'s'` | Select만 |
| `'c'` | Command-line |
| `'t'` | Terminal |
| `'o'` | Operator-pending |
| `''` (빈 문자열) | Normal + Visual + Operator-pending |

### 옵션 (opts)

```lua
vim.keymap.set('n', '<leader>w', ':w<CR>', {
  noremap = true,   -- 재귀적 매핑 방지 (기본값 true)
  silent = true,    -- 명령줄에 실행 내용 표시 안 함
  desc = '파일 저장', -- 키맵 설명 (which-key 등에서 표시)
  buffer = 0,       -- 현재 버퍼에만 적용 (생략 시 전역)
  expr = false,     -- rhs를 표현식으로 평가 (기본값 false)
})
```

> **팁**: `vim.keymap.set()`은 기본적으로 `noremap = true`이므로, 별도로 지정하지 않아도 비재귀적 매핑이 된다. Vimscript의 `nnoremap`과 동일한 동작이다.

### 문자열 vs 함수

`rhs`에는 문자열 또는 Lua 함수를 전달할 수 있다:

```lua
-- 문자열: Vimscript 명령이나 키 시퀀스
vim.keymap.set('n', '<leader>w', ':w<CR>')

-- Lua 함수: 복잡한 로직 실행
vim.keymap.set('n', '<leader>w', function()
  vim.cmd('write')
  vim.notify('파일이 저장되었습니다')
end, { desc = '파일 저장 + 알림' })
```

함수를 사용하면 조건 분기, 다른 API 호출 등 복잡한 동작을 구현할 수 있다.

## Leader 키 설정

Leader 키는 사용자 정의 키맵의 접두사(prefix)다. `<leader>`를 포함한 키맵은 Leader 키를 먼저 누르고 후속 키를 눌러 실행한다.

```lua
-- 스페이스 바를 Leader 키로 설정
vim.g.mapleader = ' '

-- Local Leader (파일 타입별 키맵용)
vim.g.maplocalleader = ' '
```

Leader 키 설정은 **반드시 키맵 정의보다 먼저** 해야 한다. 그래야 `<leader>`가 포함된 키맵이 올바르게 등록된다.

### Leader 키로 많이 쓰는 키

| 키 | 장점 | 단점 |
|----|------|------|
| `<Space>` | 양손 모두 접근 가능, 넓은 키 | Normal 모드의 기본 동작(커서 이동) 덮어씀 |
| `\` | Vim 기본 Leader | 위치가 불편, 키보드마다 다름 |
| `,` | 접근성 좋음 | `f`/`t` 역방향 반복 키를 덮어씀 |

`<Space>`가 가장 인기 있는 선택이다. 양손 엄지로 쉽게 누를 수 있고, 기존 Vim 동작 중 잃는 것이 거의 없다.

## 실용적인 키맵 모음

아래는 많은 Neovim 사용자가 설정하는 유용한 키맵이다. 모든 것을 넣을 필요는 없다. 필요한 것만 선택하자.

### 기본 편의 키맵

```lua
local keymap = vim.keymap.set

-- 검색 하이라이트 끄기
keymap('n', '<Esc>', '<cmd>nohlsearch<CR>', { desc = '검색 하이라이트 끄기' })

-- 파일 저장
keymap('n', '<leader>w', '<cmd>write<CR>', { desc = '파일 저장' })

-- 파일 저장 후 종료
keymap('n', '<leader>q', '<cmd>quit<CR>', { desc = '종료' })

-- 모든 버퍼 저장
keymap('n', '<leader>W', '<cmd>wall<CR>', { desc = '모든 파일 저장' })

-- 강제 종료
keymap('n', '<leader>Q', '<cmd>quit!<CR>', { desc = '강제 종료' })
```

### 창 이동

```lua
-- 창 간 이동: Ctrl + h/j/k/l
keymap('n', '<C-h>', '<C-w>h', { desc = '왼쪽 창으로 이동' })
keymap('n', '<C-j>', '<C-w>j', { desc = '아래 창으로 이동' })
keymap('n', '<C-k>', '<C-w>k', { desc = '위 창으로 이동' })
keymap('n', '<C-l>', '<C-w>l', { desc = '오른쪽 창으로 이동' })

-- 창 크기 조절
keymap('n', '<C-Up>', '<cmd>resize +2<CR>', { desc = '창 높이 증가' })
keymap('n', '<C-Down>', '<cmd>resize -2<CR>', { desc = '창 높이 감소' })
keymap('n', '<C-Left>', '<cmd>vertical resize -2<CR>', { desc = '창 너비 감소' })
keymap('n', '<C-Right>', '<cmd>vertical resize +2<CR>', { desc = '창 너비 증가' })
```

### 버퍼 이동

```lua
-- 버퍼 간 이동
keymap('n', '<S-h>', '<cmd>bprevious<CR>', { desc = '이전 버퍼' })
keymap('n', '<S-l>', '<cmd>bnext<CR>', { desc = '다음 버퍼' })

-- 버퍼 삭제
keymap('n', '<leader>bd', '<cmd>bdelete<CR>', { desc = '버퍼 삭제' })
```

### 줄 이동

```lua
-- Visual 모드에서 선택한 줄 이동
keymap('v', 'J', ":move '>+1<CR>gv=gv", { desc = '선택 줄 아래로 이동' })
keymap('v', 'K', ":move '<-2<CR>gv=gv", { desc = '선택 줄 위로 이동' })
```

### 들여쓰기 유지

```lua
-- Visual 모드에서 들여쓰기 후 선택 유지
keymap('v', '<', '<gv', { desc = '내어쓰기 (선택 유지)' })
keymap('v', '>', '>gv', { desc = '들여쓰기 (선택 유지)' })
```

### 검색과 이동 개선

```lua
-- 검색 시 결과를 화면 중앙에
keymap('n', 'n', 'nzzzv', { desc = '다음 검색 결과 (중앙)' })
keymap('n', 'N', 'Nzzzv', { desc = '이전 검색 결과 (중앙)' })

-- 페이지 이동 시 커서를 화면 중앙에
keymap('n', '<C-d>', '<C-d>zz', { desc = '반 페이지 아래 (중앙)' })
keymap('n', '<C-u>', '<C-u>zz', { desc = '반 페이지 위 (중앙)' })

-- J로 줄 합칠 때 커서 위치 유지
keymap('n', 'J', 'mzJ`z', { desc = '줄 합치기 (커서 유지)' })
```

### 붙여넣기 개선

```lua
-- Visual 모드에서 붙여넣기 시 레지스터 유지
keymap('x', '<leader>p', '"_dP', { desc = '붙여넣기 (레지스터 유지)' })

-- 삭제 시 블랙홀 레지스터 사용 (레지스터 오염 방지)
keymap({ 'n', 'v' }, '<leader>d', '"_d', { desc = '삭제 (레지스터 보존)' })
```

### Insert 모드 편의 키맵

```lua
-- jk로 Insert 모드 탈출
keymap('i', 'jk', '<Esc>', { desc = 'Insert 모드 탈출' })

-- Insert 모드에서 이동
keymap('i', '<C-h>', '<Left>', { desc = '왼쪽 이동' })
keymap('i', '<C-l>', '<Right>', { desc = '오른쪽 이동' })
keymap('i', '<C-j>', '<Down>', { desc = '아래 이동' })
keymap('i', '<C-k>', '<Up>', { desc = '위 이동' })
```

### 설정 파일 관련

```lua
-- init.lua 열기
keymap('n', '<leader>vc', '<cmd>edit $MYVIMRC<CR>', { desc = 'init.lua 열기' })

-- 설정 다시 로딩
keymap('n', '<leader>vr', '<cmd>source $MYVIMRC<CR>', { desc = '설정 다시 로딩' })
```

### 유틸리티

```lua
-- 현재 줄 복제
keymap('n', '<leader>yy', 'yyp', { desc = '현재 줄 복제' })

-- 전체 파일 선택
keymap('n', '<C-a>', 'ggVG', { desc = '전체 선택' })

-- 단어 검색 및 치환 (커서 아래 단어)
keymap('n', '<leader>sr', ':%s/\\<<C-r><C-w>\\>/<C-r><C-w>/gI<Left><Left><Left>',
  { desc = '단어 치환' })

-- 현재 파일에 실행 권한 부여
keymap('n', '<leader>cx', '<cmd>!chmod +x %<CR>', { desc = '실행 권한 부여', silent = true })
```

## 키맵 확인과 디버깅

### 현재 키맵 확인

```vim
:map              " 모든 키맵 표시
:nmap             " Normal 모드 키맵 표시
:imap             " Insert 모드 키맵 표시
:nmap <leader>    " Leader로 시작하는 Normal 모드 키맵
```

Lua에서도 확인할 수 있다:

```lua
-- 특정 키맵의 매핑 정보
:lua print(vim.inspect(vim.api.nvim_get_keymap('n')))
```

### 키맵 삭제

```lua
vim.keymap.del('n', '<leader>w')  -- 키맵 삭제
```

## which-key 맛보기

키맵이 늘어나면 어떤 키에 무엇이 매핑되어 있는지 기억하기 어려워진다. **which-key** 플러그인은 키를 누르면 사용 가능한 후속 키맵을 팝업으로 보여준다.

예를 들어 `<leader>`를 누르면:

```
┌─────────────────────────────┐
│  w → 파일 저장               │
│  q → 종료                    │
│  W → 모든 파일 저장           │
│  bd → 버퍼 삭제              │
│  vc → init.lua 열기          │
│  sr → 단어 치환              │
│  ...                         │
└─────────────────────────────┘
```

이 팝업이 나타나 어떤 키를 눌러야 하는지 실시간으로 안내한다. which-key의 설치와 상세 설정은 Part 4에서 다룬다. 지금은 `desc` 옵션을 키맵에 넣어두는 습관이 중요하다는 것만 기억하자. `desc`가 없으면 which-key에서도 설명이 표시되지 않는다.

## 실습: 나만의 키맵 구성하기

아래 순서로 `keymaps.lua`를 완성해 보자.

### 단계 1: 기본 구조 만들기

```lua
-- ~/.config/nvim/lua/config/keymaps.lua

local keymap = vim.keymap.set

-- ── 기본 ──
-- 여기에 키맵 추가

-- ── 창 관리 ──
-- 여기에 키맵 추가

-- ── 버퍼 관리 ──
-- 여기에 키맵 추가

-- ── 편집 개선 ──
-- 여기에 키맵 추가
```

### 단계 2: 가장 많이 쓸 키맵부터 추가

연습 과제:
1. 위의 "실용적인 키맵 모음"에서 5개를 골라 `keymaps.lua`에 추가하라
2. `<leader>w`로 파일을 저장하고, 실제로 동작하는지 확인하라
3. Visual 모드에서 `J`/`K`로 줄이 이동하는지 테스트하라
4. `n`과 `N`으로 검색 결과가 화면 중앙에 오는지 확인하라
5. `:nmap <leader>`를 실행하여 등록된 키맵 목록을 확인하라
6. 자주 쓰는 명령이 있다면 자신만의 키맵을 하나 만들어 보라
7. `<C-h>`/`<C-j>`/`<C-k>`/`<C-l>`로 창 간 이동을 테스트하라 (`:vsplit`으로 분할 후)

## 핵심 정리

- **`vim.opt`**로 대부분의 옵션을 설정한다. 리스트/맵 조작 메서드(`:append`, `:remove`)를 활용할 수 있다
- **`vim.o`**: 전역 옵션, **`vim.bo`**: 버퍼 로컬, **`vim.wo`**: 윈도우 로컬 옵션
- **`vim.keymap.set(mode, lhs, rhs, opts)`**로 키맵을 정의한다
- **Leader 키**는 키맵 정의보다 **먼저** 설정해야 한다 (`vim.g.mapleader`)
- `rhs`에 **Lua 함수**를 전달하면 복잡한 동작도 구현할 수 있다
- **`desc`** 옵션을 항상 넣자 — which-key 등에서 키맵 설명을 표시할 때 필요하다
- 키맵은 한 번에 다 외울 필요 없다. **자주 쓰는 것부터** 추가하고 점차 늘려라
