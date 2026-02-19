# Chapter 13: 자동 명령 (Autocommands)

자동 명령(autocommand)은 **특정 이벤트가 발생했을 때 자동으로 실행되는 명령**이다. 파일을 열 때, 저장할 때, 특정 파일 타입을 감지했을 때 등 다양한 시점에 원하는 동작을 실행할 수 있다.

Chapter 12에서 설정한 옵션과 키맵은 "항상" 적용된다. 자동 명령은 "특정 상황에서만" 동작하는 설정을 가능하게 한다. 예를 들어 Python 파일을 열 때만 탭 크기를 4로 바꾸거나, 파일을 저장할 때마다 불필요한 공백을 제거하는 것이다.

## Autocommand의 구조

자동 명령은 세 가지 요소로 구성된다:

1. **이벤트(Event)**: 언제 실행할 것인가 (예: 파일 열기, 저장 등)
2. **패턴(Pattern)**: 어떤 파일에 적용할 것인가 (예: `*.py`, `*.lua`)
3. **동작(Action)**: 무엇을 실행할 것인가 (명령 또는 Lua 함수)

```
이벤트 발생 → 패턴 일치 확인 → 동작 실행
```

## vim.api.nvim_create_autocmd()

Lua에서 자동 명령을 만드는 함수다.

### 기본 문법

```lua
vim.api.nvim_create_autocmd(event, {
  pattern = pattern,     -- 파일 패턴 (선택)
  group = group,         -- 그룹 (권장)
  callback = function()  -- 실행할 Lua 함수
    -- 동작
  end,
  -- 또는
  command = 'vim command',  -- 실행할 Vim 명령 (callback 대신)
})
```

### 간단한 예시

```lua
-- 파일 저장 시 메시지 표시
vim.api.nvim_create_autocmd('BufWritePost', {
  pattern = '*',
  callback = function()
    print('파일이 저장되었습니다!')
  end,
})

-- Lua 파일을 열 때 탭 크기를 2로 설정
vim.api.nvim_create_autocmd('FileType', {
  pattern = 'lua',
  callback = function()
    vim.opt_local.tabstop = 2
    vim.opt_local.shiftwidth = 2
  end,
})
```

### 여러 이벤트 지정

```lua
-- 테이블로 여러 이벤트를 한 번에 지정할 수 있다
vim.api.nvim_create_autocmd({ 'BufEnter', 'BufWinEnter' }, {
  pattern = '*.lua',
  callback = function()
    vim.opt_local.colorcolumn = '80'
  end,
})
```

### callback의 인자

`callback` 함수는 이벤트 정보를 담은 테이블을 인자로 받는다:

```lua
vim.api.nvim_create_autocmd('BufEnter', {
  pattern = '*',
  callback = function(args)
    -- args에 포함된 정보:
    -- args.event  : 발생한 이벤트 이름
    -- args.buf    : 버퍼 번호
    -- args.file   : 파일 경로
    -- args.match  : 패턴과 일치한 문자열
    print('열린 파일: ' .. args.file)
  end,
})
```

## vim.api.nvim_create_augroup()으로 그룹 관리

자동 명령을 그룹으로 묶으면 **관리와 정리가 쉬워진다**. 특히 설정 파일을 다시 로딩할 때 중복 등록을 방지하는 데 필수적이다.

### 왜 그룹이 필요한가

그룹 없이 자동 명령을 만들면, 설정 파일을 `:source`로 다시 로딩할 때마다 **같은 자동 명령이 중복 등록**된다. 파일을 저장할 때마다 메시지가 1번 → 2번 → 3번... 점점 늘어나는 것이다.

```lua
-- 나쁜 예: 그룹 없이 생성 → :source 할 때마다 중복
vim.api.nvim_create_autocmd('BufWritePost', {
  callback = function()
    print('저장!')  -- :source 3번 하면 "저장!"이 3번 출력된다
  end,
})
```

### 그룹으로 중복 방지

```lua
-- 좋은 예: 그룹으로 관리 → :source 해도 중복 없음
local group = vim.api.nvim_create_augroup('MyAutoGroup', { clear = true })

vim.api.nvim_create_autocmd('BufWritePost', {
  group = group,
  callback = function()
    print('저장!')  -- 항상 1번만 실행
  end,
})
```

`{ clear = true }` 옵션이 핵심이다. 같은 이름의 그룹이 이미 있으면 기존 자동 명령을 **모두 제거**하고 새로 만든다. 따라서 `:source`를 반복해도 중복이 발생하지 않는다.

### 그룹 사용 패턴

```lua
-- 파일 상단에 그룹을 먼저 정의
local augroup = function(name)
  return vim.api.nvim_create_augroup('my_' .. name, { clear = true })
end

-- 자동 명령에서 그룹 지정
vim.api.nvim_create_autocmd('TextYankPost', {
  group = augroup('yank_highlight'),
  callback = function()
    vim.highlight.on_yank()
  end,
})

vim.api.nvim_create_autocmd('BufWritePre', {
  group = augroup('trim_whitespace'),
  pattern = '*',
  callback = function()
    -- trailing whitespace 제거
  end,
})
```

## 주요 이벤트

Neovim에는 수십 가지 이벤트가 있다. 가장 자주 사용하는 이벤트를 카테고리별로 정리했다.

### 버퍼 관련 이벤트

| 이벤트 | 발생 시점 |
|--------|-----------|
| `BufEnter` | 버퍼에 진입할 때 |
| `BufLeave` | 버퍼를 떠날 때 |
| `BufRead` / `BufReadPost` | 파일을 읽은 후 |
| `BufNewFile` | 새 파일을 만들 때 |
| `BufWritePre` | 파일 저장 직전 |
| `BufWritePost` | 파일 저장 직후 |
| `BufDelete` | 버퍼가 삭제될 때 |

### 파일 타입 이벤트

| 이벤트 | 발생 시점 |
|--------|-----------|
| `FileType` | 파일 타입이 설정될 때 |

`FileType`은 가장 많이 쓰는 이벤트 중 하나다. 파일 타입별로 다른 설정을 적용할 때 사용한다.

### 윈도우/에디터 이벤트

| 이벤트 | 발생 시점 |
|--------|-----------|
| `WinEnter` | 윈도우에 진입할 때 |
| `WinLeave` | 윈도우를 떠날 때 |
| `VimEnter` | Neovim이 완전히 시작된 후 |
| `VimLeavePre` | Neovim 종료 직전 |
| `FocusGained` | 터미널 포커스를 얻었을 때 |
| `FocusLost` | 터미널 포커스를 잃었을 때 |

### 편집 이벤트

| 이벤트 | 발생 시점 |
|--------|-----------|
| `TextYankPost` | 텍스트를 yank(복사)한 후 |
| `InsertEnter` | Insert 모드 진입 시 |
| `InsertLeave` | Insert 모드 탈출 시 |
| `CursorHold` | 커서가 일정 시간 멈춰 있을 때 |
| `CursorMoved` | 커서가 이동할 때 |
| `TextChanged` | 텍스트가 변경될 때 (Normal 모드) |
| `TextChangedI` | 텍스트가 변경될 때 (Insert 모드) |

### 터미널 이벤트

| 이벤트 | 발생 시점 |
|--------|-----------|
| `TermOpen` | 터미널 버퍼가 열릴 때 |
| `TermClose` | 터미널 버퍼가 닫힐 때 |

> **팁**: 전체 이벤트 목록은 `:help autocmd-events`에서 확인할 수 있다.

## 실전 자동 명령 패턴

실제 설정 파일에서 가장 많이 사용되는 자동 명령 패턴들이다.

### 저장 시 trailing whitespace 제거

줄 끝의 불필요한 공백을 자동으로 제거한다:

```lua
vim.api.nvim_create_autocmd('BufWritePre', {
  group = vim.api.nvim_create_augroup('trim_whitespace', { clear = true }),
  pattern = '*',
  callback = function()
    -- 현재 커서 위치 저장
    local cursor_pos = vim.api.nvim_win_get_cursor(0)
    -- trailing whitespace 제거
    vim.cmd([[%s/\s\+$//e]])
    -- 커서 위치 복원
    vim.api.nvim_win_set_cursor(0, cursor_pos)
  end,
})
```

`%s/\s\+$//e`에서 `e` 플래그는 일치하는 패턴이 없을 때 에러를 무시한다. 커서 위치를 저장하고 복원하는 것은 치환 후 커서가 엉뚱한 곳으로 이동하는 것을 방지한다.

### 파일 타입별 설정

파일 타입에 따라 다른 옵션을 적용한다:

```lua
vim.api.nvim_create_autocmd('FileType', {
  group = vim.api.nvim_create_augroup('filetype_settings', { clear = true }),
  pattern = { 'lua', 'javascript', 'typescript', 'json', 'yaml', 'html', 'css' },
  callback = function()
    vim.opt_local.tabstop = 2
    vim.opt_local.shiftwidth = 2
  end,
})

vim.api.nvim_create_autocmd('FileType', {
  group = vim.api.nvim_create_augroup('filetype_python', { clear = true }),
  pattern = 'python',
  callback = function()
    vim.opt_local.tabstop = 4
    vim.opt_local.shiftwidth = 4
    vim.opt_local.colorcolumn = '88'  -- Black 포매터 기준
  end,
})

vim.api.nvim_create_autocmd('FileType', {
  group = vim.api.nvim_create_augroup('filetype_go', { clear = true }),
  pattern = 'go',
  callback = function()
    vim.opt_local.tabstop = 4
    vim.opt_local.shiftwidth = 4
    vim.opt_local.expandtab = false  -- Go는 탭 문자 사용
  end,
})
```

> **팁**: `vim.opt_local`은 현재 버퍼/윈도우에만 옵션을 적용한다. 파일 타입별 설정에서는 반드시 `vim.opt_local`을 사용해야 다른 버퍼에 영향을 주지 않는다.

### 복사 시 하이라이트 효과

텍스트를 복사(yank)하면 해당 영역이 잠깐 하이라이트되어 어떤 부분이 복사되었는지 시각적으로 확인할 수 있다:

```lua
vim.api.nvim_create_autocmd('TextYankPost', {
  group = vim.api.nvim_create_augroup('yank_highlight', { clear = true }),
  callback = function()
    vim.highlight.on_yank({
      higroup = 'IncSearch',  -- 하이라이트 색상 그룹
      timeout = 200,          -- 지속 시간 (밀리초)
    })
  end,
})
```

이 설정은 거의 모든 Neovim 사용자가 넣는 "필수" 자동 명령이다.

### 마지막 편집 위치 복원

파일을 다시 열었을 때 마지막으로 편집하던 위치로 커서를 이동한다:

```lua
vim.api.nvim_create_autocmd('BufReadPost', {
  group = vim.api.nvim_create_augroup('restore_cursor', { clear = true }),
  callback = function(args)
    local mark = vim.api.nvim_buf_get_mark(args.buf, '"')
    local line_count = vim.api.nvim_buf_line_count(args.buf)
    if mark[1] > 0 and mark[1] <= line_count then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
})
```

`'"'` 마크는 Neovim이 자동으로 기록하는 "마지막 편집 위치"다. `pcall`은 에러가 발생해도 프로그램이 중단되지 않도록 보호하는 Lua 함수다.

### 특정 파일 열 때 자동 명령 실행

```lua
-- Markdown 파일에서 줄바꿈 활성화
vim.api.nvim_create_autocmd('FileType', {
  group = vim.api.nvim_create_augroup('markdown_settings', { clear = true }),
  pattern = 'markdown',
  callback = function()
    vim.opt_local.wrap = true
    vim.opt_local.linebreak = true
    vim.opt_local.spell = true
    vim.opt_local.spelllang = 'en,cjk'  -- 영어 맞춤법 + CJK 무시
  end,
})

-- 터미널 열 때 Insert 모드로 시작
vim.api.nvim_create_autocmd('TermOpen', {
  group = vim.api.nvim_create_augroup('terminal_settings', { clear = true }),
  callback = function()
    vim.opt_local.number = false
    vim.opt_local.relativenumber = false
    vim.cmd('startinsert')
  end,
})
```

### 포커스 복귀 시 파일 변경 감지

다른 프로그램에서 파일을 수정한 경우, Neovim으로 돌아왔을 때 자동으로 반영한다:

```lua
vim.api.nvim_create_autocmd({ 'FocusGained', 'TermClose', 'TermLeave' }, {
  group = vim.api.nvim_create_augroup('checktime', { clear = true }),
  callback = function()
    if vim.o.buftype ~= 'nofile' then
      vim.cmd('checktime')
    end
  end,
})
```

### 빈 줄에서 자동 들여쓰기 제거

빈 줄을 만들고 다른 줄로 이동하면 불필요한 공백이 남는 문제를 해결한다:

```lua
vim.api.nvim_create_autocmd('BufWritePre', {
  group = vim.api.nvim_create_augroup('remove_blank_indent', { clear = true }),
  pattern = '*',
  callback = function()
    local cursor_pos = vim.api.nvim_win_get_cursor(0)
    vim.cmd([[%s/^\s\+$//e]])
    vim.api.nvim_win_set_cursor(0, cursor_pos)
  end,
})
```

### 큰 파일에서 성능 최적화

파일이 너무 크면 특정 기능을 끄서 성능을 유지한다:

```lua
vim.api.nvim_create_autocmd('BufReadPre', {
  group = vim.api.nvim_create_augroup('large_file', { clear = true }),
  callback = function(args)
    local ok, stats = pcall(vim.loop.fs_stat, vim.api.nvim_buf_get_name(args.buf))
    if ok and stats and stats.size > 1024 * 1024 then  -- 1MB 이상
      vim.opt_local.syntax = 'off'
      vim.opt_local.swapfile = false
      vim.opt_local.undofile = false
      vim.opt_local.foldmethod = 'manual'
      vim.notify('대용량 파일: 일부 기능이 비활성화되었습니다', vim.log.levels.WARN)
    end
  end,
})
```

## 파일 타입별 설정: ftplugin 디렉토리

자동 명령 외에 파일 타입별 설정을 관리하는 또 다른 방법이 있다. `ftplugin` 디렉토리를 사용하는 것이다.

### ftplugin 디렉토리 구조

```
~/.config/nvim/
├── init.lua
├── ftplugin/
│   ├── lua.lua          -- Lua 파일용 설정
│   ├── python.lua       -- Python 파일용 설정
│   ├── go.lua           -- Go 파일용 설정
│   ├── markdown.lua     -- Markdown 파일용 설정
│   └── javascript.lua   -- JavaScript 파일용 설정
└── lua/
    └── config/
```

### ftplugin 파일 예시

```lua
-- ~/.config/nvim/ftplugin/python.lua
-- Python 파일을 열 때 자동으로 실행된다

vim.opt_local.tabstop = 4
vim.opt_local.shiftwidth = 4
vim.opt_local.expandtab = true
vim.opt_local.colorcolumn = '88'

-- Python 전용 키맵
vim.keymap.set('n', '<leader>rr', ':!python3 %<CR>', {
  buffer = true,  -- 현재 버퍼에만 적용
  desc = 'Python 파일 실행',
})
```

```lua
-- ~/.config/nvim/ftplugin/lua.lua
vim.opt_local.tabstop = 2
vim.opt_local.shiftwidth = 2
vim.opt_local.expandtab = true
```

```lua
-- ~/.config/nvim/ftplugin/markdown.lua
vim.opt_local.wrap = true
vim.opt_local.linebreak = true
vim.opt_local.spell = true
vim.opt_local.conceallevel = 2
```

### ftplugin vs autocommand

| 항목 | ftplugin | autocommand (FileType) |
|------|----------|------------------------|
| 파일 위치 | `ftplugin/{filetype}.lua` | `autocmds.lua` 안에 |
| 중복 방지 | 자동 (파일당 1회 실행) | `augroup` + `clear` 필요 |
| 코드 분리 | 파일 타입별 독립 파일 | 한 파일에 모여 있음 |
| 버퍼 로컬 | 자동으로 버퍼 로컬 | `vim.opt_local` 명시 필요 |
| 키맵 | `buffer = true` 명시 필요 | `buffer = true` 명시 필요 |

파일 타입별 설정이 많다면 `ftplugin` 디렉토리가 깔끔하다. 몇 가지 간단한 설정이라면 `autocmds.lua`에 모아두는 것이 편리하다.

## 자동 명령 확인과 디버깅

### 등록된 자동 명령 확인

```vim
:autocmd                    " 모든 자동 명령 표시
:autocmd BufWritePre        " 특정 이벤트의 자동 명령 표시
:autocmd MyGroup            " 특정 그룹의 자동 명령 표시
```

### 자동 명령 삭제

```lua
-- 그룹 단위로 삭제 (clear = true로 재생성하면 됨)
vim.api.nvim_create_augroup('MyGroup', { clear = true })

-- 특정 자동 명령 삭제 (ID 필요)
local id = vim.api.nvim_create_autocmd('BufEnter', {
  callback = function() end,
})
vim.api.nvim_del_autocmd(id)
```

### 이벤트 수동 발생

```vim
:doautocmd BufWritePre      " BufWritePre 이벤트를 수동으로 발생
:doautocmd FileType python  " FileType python 이벤트를 수동으로 발생
```

디버깅할 때 자동 명령이 올바르게 동작하는지 이벤트를 수동으로 발생시켜 테스트할 수 있다.

## 실습: autocmds.lua 완성하기

아래 순서로 `autocmds.lua`를 작성해 보자.

```lua
-- ~/.config/nvim/lua/config/autocmds.lua

-- 헬퍼 함수
local function augroup(name)
  return vim.api.nvim_create_augroup('my_' .. name, { clear = true })
end

-- 1. 복사 시 하이라이트
vim.api.nvim_create_autocmd('TextYankPost', {
  group = augroup('yank_highlight'),
  callback = function()
    vim.highlight.on_yank({ timeout = 200 })
  end,
})

-- 2. 마지막 편집 위치 복원
vim.api.nvim_create_autocmd('BufReadPost', {
  group = augroup('restore_cursor'),
  callback = function(args)
    local mark = vim.api.nvim_buf_get_mark(args.buf, '"')
    local line_count = vim.api.nvim_buf_line_count(args.buf)
    if mark[1] > 0 and mark[1] <= line_count then
      pcall(vim.api.nvim_win_set_cursor, 0, mark)
    end
  end,
})

-- 3. 저장 시 trailing whitespace 제거
vim.api.nvim_create_autocmd('BufWritePre', {
  group = augroup('trim_whitespace'),
  pattern = '*',
  callback = function()
    local cursor_pos = vim.api.nvim_win_get_cursor(0)
    vim.cmd([[%s/\s\+$//e]])
    vim.api.nvim_win_set_cursor(0, cursor_pos)
  end,
})

-- 4. 파일 변경 감지
vim.api.nvim_create_autocmd({ 'FocusGained', 'TermClose', 'TermLeave' }, {
  group = augroup('checktime'),
  callback = function()
    if vim.o.buftype ~= 'nofile' then
      vim.cmd('checktime')
    end
  end,
})

-- 5. 여기에 자신만의 자동 명령을 추가해 보자!
```

연습 과제:
1. 위의 `autocmds.lua`를 작성하고 Neovim을 재시작하라
2. 텍스트를 복사(yank)해서 하이라이트가 나타나는지 확인하라
3. 파일을 열고, 다른 위치로 이동한 뒤 저장하고, 다시 열어서 커서 위치가 복원되는지 확인하라
4. 줄 끝에 공백을 추가한 뒤 저장하고, 공백이 제거되었는지 확인하라
5. `ftplugin/` 디렉토리에 사용하는 언어의 설정 파일을 하나 만들어 보라
6. `:autocmd my_yank_highlight`을 실행하여 자동 명령이 등록되었는지 확인하라
7. Python 파일과 Lua 파일을 각각 열어 `tabstop` 값이 다른지 `:set tabstop?`으로 확인하라

## 핵심 정리

- 자동 명령은 **이벤트 + 패턴 + 동작**으로 구성된다
- **`vim.api.nvim_create_autocmd()`**로 생성하고, **callback**에 Lua 함수를 전달한다
- **`vim.api.nvim_create_augroup()`**과 **`clear = true`**로 중복 등록을 방지한다 (필수 패턴)
- 가장 많이 쓰는 이벤트: **`FileType`**, **`BufWritePre`**, **`TextYankPost`**, **`BufReadPost`**
- **`vim.opt_local`**을 사용해 파일 타입별 설정이 다른 버퍼에 영향을 주지 않도록 한다
- **`ftplugin/`** 디렉토리로 파일 타입별 설정을 독립 파일로 관리할 수 있다
- 자동 명령의 `callback`에서 **`args`** 매개변수로 이벤트 정보(버퍼 번호, 파일 경로 등)에 접근한다
