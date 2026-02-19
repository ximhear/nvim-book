# Chapter 14: Lua 스크립팅

Chapter 11~13에서 init.lua, 옵션, 키맵, 자동 명령을 설정했다. 이 챕터에서는 Lua 언어 자체와 Neovim의 Lua API를 깊이 다룬다. Lua를 제대로 이해하면 단순한 설정을 넘어 **자신만의 기능을 직접 만들 수 있다**.

Neovim에 내장된 Lua는 LuaJIT(Lua 5.1 호환)이다. 문법이 간결하고 배우기 쉬워서, 프로그래밍 경험이 적더라도 빠르게 익힐 수 있다.

## Lua 기초 문법

### 변수

```lua
-- 로컬 변수 (권장: 항상 local을 붙인다)
local name = 'Neovim'
local version = 0.10
local is_active = true
local nothing = nil  -- 값이 없음

-- 전역 변수 (local 없이 선언 → 권장하지 않음)
global_var = 'avoid this'
```

> **팁**: Lua에서 `local` 없이 선언한 변수는 전역 변수가 된다. 전역 변수는 다른 모듈과 충돌할 수 있으므로, 항상 `local`을 붙이는 습관을 들이자.

### 자료형

```lua
local str = 'hello'          -- 문자열 (string)
local num = 42               -- 숫자 (number)
local float_num = 3.14       -- 소수점 숫자
local bool = true             -- 부울 (boolean)
local empty = nil             -- nil
local tbl = {}                -- 테이블 (table)
local fn = function() end    -- 함수 (function)

-- 자료형 확인
print(type(str))     -- "string"
print(type(num))     -- "number"
print(type(tbl))     -- "table"
```

Lua에는 이 외에 별도의 배열, 딕셔너리, 클래스 자료형이 없다. **테이블 하나로 모든 것을 표현**한다.

### 문자열

```lua
-- 문자열 선언
local s1 = 'single quotes'
local s2 = "double quotes"
local s3 = [[
  여러 줄
  문자열
  (long string)
]]

-- 문자열 연결: .. 연산자
local greeting = 'Hello' .. ' ' .. 'World'  -- "Hello World"

-- 문자열 길이
print(#greeting)  -- 11

-- 문자열 포맷
local formatted = string.format('이름: %s, 나이: %d', 'Kim', 30)

-- 주요 문자열 함수
string.upper('hello')       -- "HELLO"
string.lower('HELLO')       -- "hello"
string.sub('hello', 1, 3)   -- "hel" (1번째부터 3번째까지)
string.find('hello', 'llo') -- 3, 5 (시작 위치, 끝 위치)
string.gsub('hello', 'l', 'r')  -- "herro", 2 (치환 결과, 치환 횟수)
string.match('2024-01-15', '(%d+)-(%d+)-(%d+)')  -- "2024", "01", "15"
```

### 테이블 (Table)

테이블은 Lua의 유일한 복합 자료형이다. 배열, 딕셔너리, 객체 역할을 모두 한다.

```lua
-- 배열처럼 사용 (인덱스가 1부터 시작!)
local fruits = { 'apple', 'banana', 'cherry' }
print(fruits[1])    -- "apple" (0이 아니라 1부터!)
print(#fruits)      -- 3 (길이)

-- 딕셔너리(맵)처럼 사용
local config = {
  theme = 'dark',
  font_size = 14,
  show_line_numbers = true,
}
print(config.theme)        -- "dark"
print(config['font_size']) -- 14

-- 혼합 사용
local mixed = {
  'first',           -- [1] = 'first'
  'second',          -- [2] = 'second'
  name = 'example',  -- name = 'example'
}

-- 테이블 수정
fruits[4] = 'date'              -- 요소 추가
table.insert(fruits, 'elderberry')  -- 끝에 추가
table.remove(fruits, 2)         -- 2번째 요소 제거

-- 중첩 테이블
local nested = {
  user = {
    name = 'Kim',
    settings = {
      theme = 'dark',
    },
  },
}
print(nested.user.settings.theme)  -- "dark"
```

### 조건문

```lua
local x = 10

-- 기본 if-else
if x > 0 then
  print('양수')
elseif x < 0 then
  print('음수')
else
  print('영')
end

-- Lua에서 "거짓"으로 평가되는 값: nil과 false만
-- 0, '', {} 등은 모두 "참"이다!
if 0 then print('0은 참이다!') end         -- 출력됨
if '' then print('빈 문자열도 참이다!') end  -- 출력됨

-- and / or 논리 연산자
local a = true and 'yes' or 'no'  -- 삼항 연산자 패턴: "yes"
local b = nil or 'default'        -- nil이면 기본값: "default"
```

> **팁**: Lua에는 삼항 연산자(`? :`)가 없다. 대신 `condition and value_if_true or value_if_false` 패턴을 사용한다. 단, `value_if_true`가 `nil`이나 `false`면 올바르게 동작하지 않으니 주의하자.

### 반복문

```lua
-- 숫자 for
for i = 1, 5 do
  print(i)  -- 1, 2, 3, 4, 5
end

for i = 10, 1, -2 do
  print(i)  -- 10, 8, 6, 4, 2
end

-- ipairs: 배열 순회 (순서 보장, 인덱스 + 값)
local fruits = { 'apple', 'banana', 'cherry' }
for i, fruit in ipairs(fruits) do
  print(i, fruit)  -- 1 apple, 2 banana, 3 cherry
end

-- pairs: 테이블 순회 (순서 보장 안 됨, 키 + 값)
local config = { theme = 'dark', size = 14 }
for key, value in pairs(config) do
  print(key, value)
end

-- while
local count = 0
while count < 3 do
  count = count + 1
  print(count)
end

-- repeat-until (do-while과 유사)
local n = 0
repeat
  n = n + 1
until n >= 3
```

### 함수

```lua
-- 기본 함수 선언
local function greet(name)
  return 'Hello, ' .. name
end
print(greet('Vim'))  -- "Hello, Vim"

-- 익명 함수 (anonymous function)
local square = function(x)
  return x * x
end

-- 여러 값 반환
local function get_position()
  return 10, 20
end
local x, y = get_position()

-- 가변 인자
local function sum(...)
  local total = 0
  for _, v in ipairs({...}) do
    total = total + v
  end
  return total
end
print(sum(1, 2, 3, 4))  -- 10

-- 테이블을 인자로 받는 패턴 (옵션 전달에 자주 사용)
local function setup(opts)
  opts = opts or {}
  local theme = opts.theme or 'dark'
  local size = opts.size or 14
  print(theme, size)
end

setup({ theme = 'light', size = 16 })
setup()  -- 기본값 사용
```

## Neovim Lua API: vim.api

`vim.api`는 Neovim의 내부 기능에 직접 접근하는 API다. `nvim_`으로 시작하는 함수들로 구성되어 있다.

### 버퍼 관련 API

```lua
-- 현재 버퍼 번호
local buf = vim.api.nvim_get_current_buf()

-- 버퍼의 줄 가져오기 (0-indexed)
local lines = vim.api.nvim_buf_get_lines(buf, 0, -1, false)
-- 0: 시작 줄, -1: 마지막 줄까지, false: 범위 밖 허용

-- 특정 범위의 줄 가져오기 (3번째~5번째 줄)
local some_lines = vim.api.nvim_buf_get_lines(buf, 2, 5, false)

-- 줄 설정하기
vim.api.nvim_buf_set_lines(buf, 0, 0, false, { '새 첫 번째 줄' })
-- 0번 줄 앞에 삽입

-- 버퍼 이름(파일 경로) 가져오기
local name = vim.api.nvim_buf_get_name(buf)

-- 버퍼의 줄 수
local line_count = vim.api.nvim_buf_line_count(buf)

-- 버퍼 옵션 가져오기/설정하기
local ft = vim.api.nvim_buf_get_option(buf, 'filetype')
vim.api.nvim_buf_set_option(buf, 'modifiable', false)
```

### 윈도우 관련 API

```lua
-- 현재 윈도우 번호
local win = vim.api.nvim_get_current_win()

-- 커서 위치 가져오기 (1-indexed 줄, 0-indexed 열)
local cursor = vim.api.nvim_win_get_cursor(win)
local row = cursor[1]  -- 줄 번호 (1부터)
local col = cursor[2]  -- 열 번호 (0부터)

-- 커서 위치 설정
vim.api.nvim_win_set_cursor(win, { 5, 0 })  -- 5번째 줄 첫 번째 열

-- 윈도우 크기
local width = vim.api.nvim_win_get_width(win)
local height = vim.api.nvim_win_get_height(win)
```

### 전역 API

```lua
-- 모든 버퍼 목록
local buffers = vim.api.nvim_list_bufs()

-- 모든 윈도우 목록
local windows = vim.api.nvim_list_wins()

-- 하이라이트 설정
vim.api.nvim_set_hl(0, 'MyHighlight', {
  fg = '#ff0000',
  bg = '#000000',
  bold = true,
})

-- 사용자 명령 만들기
vim.api.nvim_create_user_command('Hello', function(opts)
  print('Hello, ' .. opts.args)
end, {
  nargs = 1,     -- 인자 1개 필요
  desc = '인사 명령',
})
-- 사용: :Hello World → "Hello, World"
```

## vim.fn: Vimscript 함수 호출

`vim.fn`을 통해 Vimscript의 내장 함수를 Lua에서 호출할 수 있다. Vimscript에는 수백 가지 유용한 내장 함수가 있으며, Lua API로 대체할 수 없는 것도 많다.

```lua
-- 파일 경로 관련
local home = vim.fn.expand('~')                  -- 홈 디렉토리
local current_file = vim.fn.expand('%')           -- 현재 파일명
local full_path = vim.fn.expand('%:p')            -- 현재 파일 전체 경로
local dir = vim.fn.expand('%:p:h')                -- 현재 파일의 디렉토리
local filename = vim.fn.expand('%:t')             -- 파일명만
local ext = vim.fn.expand('%:e')                  -- 확장자만

-- 파일 시스템
local exists = vim.fn.filereadable('/path/to/file')  -- 파일 존재 여부 (1/0)
local is_dir = vim.fn.isdirectory('/path/to/dir')    -- 디렉토리 여부 (1/0)
local files = vim.fn.glob('*.lua', false, true)       -- 패턴 매칭 (테이블 반환)
vim.fn.mkdir('/path/to/dir', 'p')                     -- 디렉토리 생성

-- 시스템 명령 실행
local result = vim.fn.system('echo hello')  -- "hello\n"
local lines = vim.fn.systemlist('ls')       -- 줄별 테이블

-- 입력 받기
local answer = vim.fn.input('이름을 입력하세요: ')
local confirmed = vim.fn.confirm('저장할까요?', '&Yes\n&No', 1)

-- 기타 유용한 함수
local os_name = vim.fn.has('mac')      -- macOS인지 (1/0)
local has_cmd = vim.fn.executable('git')  -- 명령어 존재 여부 (1/0)
local line_nr = vim.fn.line('.')       -- 현재 줄 번호
local col_nr = vim.fn.col('.')        -- 현재 열 번호
```

> **팁**: `vim.fn`의 함수들은 Vimscript 규칙을 따른다. boolean 대신 `0`/`1`을 반환하는 경우가 많으므로, 조건문에서 `== 1`로 비교하는 것이 안전하다.

## vim.cmd: Vimscript 명령 실행

Chapter 11에서 소개한 `vim.cmd`를 더 깊이 다룬다.

```lua
-- 단일 명령
vim.cmd('write')
vim.cmd('colorscheme habamax')
vim.cmd('echo "hello"')

-- 여러 줄 명령
vim.cmd([[
  highlight Normal guibg=#1e1e2e
  highlight Comment guifg=#6c7086
]])

-- 함수 호출 스타일
vim.cmd.write()
vim.cmd.colorscheme('habamax')
vim.cmd.edit('~/.config/nvim/init.lua')

-- Ex 명령 범위 지정
vim.cmd('%s/old/new/g')          -- 전체 치환
vim.cmd('1,10 delete')           -- 1~10줄 삭제
vim.cmd('normal! gg=G')          -- gg=G 실행 (전체 정렬)
```

## vim.notify: 사용자 알림

`print()`보다 시각적으로 나은 알림을 보여준다. 나중에 nvim-notify 같은 플러그인과 결합하면 더 멋진 알림을 표시할 수 있다.

```lua
-- 기본 알림
vim.notify('파일이 저장되었습니다')

-- 알림 레벨 지정
vim.notify('작업 완료', vim.log.levels.INFO)
vim.notify('주의: 변경사항이 있습니다', vim.log.levels.WARN)
vim.notify('오류가 발생했습니다', vim.log.levels.ERROR)
vim.notify('디버그 정보', vim.log.levels.DEBUG)

-- 알림 레벨
-- vim.log.levels.TRACE (0)
-- vim.log.levels.DEBUG (1)
-- vim.log.levels.INFO  (2)
-- vim.log.levels.WARN  (3)
-- vim.log.levels.ERROR (4)
```

## 모듈 패턴: local M = {} ... return M

Lua에서 재사용 가능한 코드를 만드는 표준 패턴이다. Neovim 플러그인과 설정에서 매우 자주 사용된다.

### 기본 패턴

```lua
-- ~/.config/nvim/lua/utils.lua

local M = {}

function M.greet(name)
  return 'Hello, ' .. name
end

function M.is_mac()
  return vim.fn.has('mac') == 1
end

function M.get_visual_selection()
  local _, ls, cs = unpack(vim.fn.getpos('v'))
  local _, le, ce = unpack(vim.fn.getpos('.'))
  local lines = vim.api.nvim_buf_get_lines(0, ls - 1, le, false)
  if #lines == 0 then return '' end
  lines[#lines] = string.sub(lines[#lines], 1, ce)
  lines[1] = string.sub(lines[1], cs)
  return table.concat(lines, '\n')
end

return M
```

### 모듈 사용

```lua
-- 다른 파일에서 require로 불러오기
local utils = require('utils')

print(utils.greet('Vim'))  -- "Hello, Vim"

if utils.is_mac() then
  -- macOS 전용 설정
end
```

### setup 패턴

플러그인에서 흔히 볼 수 있는 패턴이다:

```lua
-- ~/.config/nvim/lua/my_module.lua

local M = {}

-- 기본 설정
local defaults = {
  enabled = true,
  theme = 'dark',
  width = 80,
}

-- 실제 설정 (기본값과 사용자 설정 병합)
M.config = {}

function M.setup(opts)
  M.config = vim.tbl_deep_extend('force', defaults, opts or {})
  -- 초기화 로직
  if M.config.enabled then
    vim.notify('모듈이 활성화되었습니다: 테마=' .. M.config.theme)
  end
end

function M.get_width()
  return M.config.width
end

return M
```

```lua
-- 사용하는 쪽
local my_module = require('my_module')
my_module.setup({
  theme = 'light',  -- 기본값 덮어쓰기
  -- width는 기본값 80 사용
})
```

`vim.tbl_deep_extend('force', defaults, opts)`는 두 테이블을 깊은 병합(deep merge)한다. 사용자가 지정하지 않은 값은 기본값이 유지된다.

## 실전 유틸리티 함수 예제

### 현재 파일 경로 클립보드에 복사

```lua
-- ~/.config/nvim/lua/utils.lua 에 추가

local M = {}

-- 현재 파일의 절대 경로를 클립보드에 복사
function M.copy_file_path()
  local path = vim.fn.expand('%:p')
  if path == '' then
    vim.notify('파일이 저장되지 않았습니다', vim.log.levels.WARN)
    return
  end
  vim.fn.setreg('+', path)
  vim.notify('경로 복사: ' .. path)
end

-- 현재 파일의 상대 경로를 클립보드에 복사
function M.copy_relative_path()
  local path = vim.fn.expand('%')
  if path == '' then
    vim.notify('파일이 저장되지 않았습니다', vim.log.levels.WARN)
    return
  end
  vim.fn.setreg('+', path)
  vim.notify('경로 복사: ' .. path)
end

-- 현재 파일명 + 줄 번호를 클립보드에 복사
function M.copy_file_location()
  local path = vim.fn.expand('%')
  local line = vim.fn.line('.')
  local location = path .. ':' .. line
  vim.fn.setreg('+', location)
  vim.notify('위치 복사: ' .. location)
end

return M
```

키맵에 연결:

```lua
-- keymaps.lua에 추가
local utils = require('utils')
vim.keymap.set('n', '<leader>cp', utils.copy_file_path, { desc = '파일 경로 복사' })
vim.keymap.set('n', '<leader>cr', utils.copy_relative_path, { desc = '상대 경로 복사' })
vim.keymap.set('n', '<leader>cl', utils.copy_file_location, { desc = '파일 위치 복사' })
```

### 버퍼 내용 기반 간단한 통계

```lua
function M.buffer_stats()
  local buf = vim.api.nvim_get_current_buf()
  local lines = vim.api.nvim_buf_get_lines(buf, 0, -1, false)

  local total_lines = #lines
  local blank_lines = 0
  local total_words = 0
  local total_chars = 0

  for _, line in ipairs(lines) do
    if line:match('^%s*$') then
      blank_lines = blank_lines + 1
    end
    for _ in line:gmatch('%S+') do
      total_words = total_words + 1
    end
    total_chars = total_chars + #line
  end

  local stats = string.format(
    '줄: %d | 빈 줄: %d | 단어: %d | 문자: %d',
    total_lines, blank_lines, total_words, total_chars
  )
  vim.notify(stats, vim.log.levels.INFO)
end
```

키맵에 연결:

```lua
vim.keymap.set('n', '<leader>bs', require('utils').buffer_stats, { desc = '버퍼 통계' })
```

### 토글 함수 만들기

```lua
-- 줄 번호 토글
function M.toggle_number()
  if vim.opt_local.number:get() then
    vim.opt_local.number = false
    vim.opt_local.relativenumber = false
    vim.notify('줄 번호: OFF')
  else
    vim.opt_local.number = true
    vim.opt_local.relativenumber = true
    vim.notify('줄 번호: ON')
  end
end

-- 줄바꿈 토글
function M.toggle_wrap()
  local wrap = not vim.opt_local.wrap:get()
  vim.opt_local.wrap = wrap
  vim.notify('줄바꿈: ' .. (wrap and 'ON' or 'OFF'))
end

-- 진단(diagnostic) 토글
function M.toggle_diagnostics()
  local enabled = vim.diagnostic.is_enabled()
  vim.diagnostic.enable(not enabled)
  vim.notify('진단: ' .. (enabled and 'OFF' or 'ON'))
end

-- 범용 토글 함수
function M.toggle_option(option)
  local current = vim.opt_local[option]:get()
  vim.opt_local[option] = not current
  vim.notify(option .. ': ' .. (current and 'OFF' or 'ON'))
end
```

키맵에 연결:

```lua
local utils = require('utils')
vim.keymap.set('n', '<leader>tn', utils.toggle_number, { desc = '줄 번호 토글' })
vim.keymap.set('n', '<leader>tw', utils.toggle_wrap, { desc = '줄바꿈 토글' })
vim.keymap.set('n', '<leader>td', utils.toggle_diagnostics, { desc = '진단 토글' })
vim.keymap.set('n', '<leader>ts', function()
  utils.toggle_option('spell')
end, { desc = '맞춤법 검사 토글' })
```

## vim.inspect로 디버깅

`vim.inspect()`는 Lua 값을 사람이 읽기 쉬운 문자열로 변환한다. 테이블의 내용을 확인하거나 API 반환값을 디버깅할 때 필수적이다.

### 기본 사용법

```lua
-- 테이블 내용 확인
local t = { 1, 2, 3, name = 'test', nested = { a = 1 } }
print(vim.inspect(t))
-- 출력:
-- { 1, 2, 3,
--   name = "test",
--   nested = {
--     a = 1
--   }
-- }
```

### 디버깅 패턴

```lua
-- 현재 버퍼 정보 확인
:lua print(vim.inspect(vim.api.nvim_buf_get_option(0, 'filetype')))

-- 키맵 정보 확인
:lua print(vim.inspect(vim.api.nvim_get_keymap('n')[1]))

-- vim.opt 값 확인 (테이블인 경우 유용)
:lua print(vim.inspect(vim.opt.completeopt:get()))

-- 모듈의 내용 확인
:lua print(vim.inspect(require('utils')))
```

### 디버그용 유틸리티 함수

```lua
-- utils.lua에 추가

-- 값을 알림으로 표시 (빠른 디버깅용)
function M.inspect(value, title)
  local output = vim.inspect(value)
  if title then
    output = title .. ':\n' .. output
  end
  vim.notify(output, vim.log.levels.DEBUG)
  return value  -- 체이닝 가능
end

-- 사용 예시:
-- require('utils').inspect(vim.opt.completeopt:get(), 'completeopt')
```

### Neovim 명령줄에서 Lua 실행

디버깅할 때 매번 파일을 수정할 필요 없이, 명령줄에서 바로 Lua를 실행할 수 있다:

```vim
:lua print('hello')
:lua print(vim.fn.expand('%'))
:lua print(vim.inspect(vim.opt.tabstop:get()))

" 여러 줄 실행
:lua << EOF
local buf = vim.api.nvim_get_current_buf()
local lines = vim.api.nvim_buf_get_lines(buf, 0, 5, false)
for i, line in ipairs(lines) do
  print(i, line)
end
EOF
```

또는 `=` 단축 접두사를 사용할 수 있다:

```vim
:= vim.fn.expand('%')
:= vim.opt.tabstop:get()
:= 1 + 2
```

## 유용한 Neovim 내장 유틸리티

### vim.tbl_* 함수들

```lua
-- 테이블 병합
local defaults = { a = 1, b = 2, c = { d = 3 } }
local override = { b = 20, c = { e = 4 } }

-- 얕은 병합: 첫 번째 수준만 덮어쓰기
local shallow = vim.tbl_extend('force', defaults, override)
-- { a = 1, b = 20, c = { e = 4 } }  -- c.d가 사라짐

-- 깊은 병합: 중첩 테이블도 병합
local deep = vim.tbl_deep_extend('force', defaults, override)
-- { a = 1, b = 20, c = { d = 3, e = 4 } }  -- c.d 유지

-- 테이블 필터
local numbers = { 1, 2, 3, 4, 5, 6 }
local even = vim.tbl_filter(function(v) return v % 2 == 0 end, numbers)
-- { 2, 4, 6 }

-- 테이블 맵
local doubled = vim.tbl_map(function(v) return v * 2 end, numbers)
-- { 2, 4, 6, 8, 10, 12 }

-- 키 목록
local keys = vim.tbl_keys(defaults)  -- { 'a', 'b', 'c' }

-- 값 목록
local values = vim.tbl_values(defaults)

-- 테이블에 값이 포함되어 있는지
local has = vim.tbl_contains(fruits, 'apple')  -- true
```

### vim.keycode

특수 키 코드를 변환한다:

```lua
local esc = vim.keycode('<Esc>')
local cr = vim.keycode('<CR>')
-- feedkeys와 함께 사용할 때 유용
vim.api.nvim_feedkeys(vim.keycode('<Esc>'), 'n', false)
```

### vim.schedule

비동기 작업 후 안전하게 API를 호출할 때 사용한다:

```lua
vim.schedule(function()
  -- 이 안의 코드는 안전한 컨텍스트에서 실행된다
  vim.api.nvim_buf_set_lines(0, 0, 0, false, { 'scheduled line' })
end)
```

## 실습: 유틸리티 모듈 만들기

아래 순서로 `utils.lua` 모듈을 만들고 활용해 보자.

### 단계 1: 모듈 파일 생성

```lua
-- ~/.config/nvim/lua/utils.lua

local M = {}

-- 여기에 함수를 추가한다

return M
```

### 단계 2: 함수 구현

연습 과제:
1. `M.copy_file_path()` 함수를 구현하고, `<leader>cp`에 매핑하라. 파일을 열고 해당 키맵을 눌러 클립보드에 경로가 복사되는지 확인하라
2. `M.buffer_stats()` 함수를 구현하고, `<leader>bs`에 매핑하라. 다양한 파일에서 통계를 확인해 보라
3. `M.toggle_number()` 함수를 구현하고, `<leader>tn`에 매핑하라. 줄 번호가 토글되는지 확인하라
4. Neovim 명령줄에서 `:lua print(vim.inspect(require('utils')))`를 실행하여 모듈 내용을 확인하라
5. `:lua print(vim.inspect(vim.api.nvim_buf_get_lines(0, 0, 5, false)))`로 현재 버퍼의 처음 5줄을 확인하라
6. 자신만의 유틸리티 함수를 하나 만들어 보라. 예를 들어 "현재 단어를 대문자로 변환"하는 함수는 어떨까?
7. `vim.tbl_filter`를 사용하여 현재 버퍼의 빈 줄이 아닌 줄만 필터링하는 코드를 `:lua` 명령줄에서 작성해 보라

## 핵심 정리

- Lua는 **간결한 문법**의 범용 언어다. 변수(`local`), 함수, 테이블(배열+딕셔너리), 조건문, 반복문이 핵심이다
- **테이블**이 Lua의 유일한 복합 자료형이다. 배열, 딕셔너리, 객체 역할을 모두 한다 (인덱스는 **1부터** 시작)
- **`vim.api`**: Neovim 내부 API (`nvim_buf_get_lines`, `nvim_win_get_cursor` 등)
- **`vim.fn`**: Vimscript 내장 함수 호출 (`expand`, `glob`, `system` 등)
- **`vim.cmd`**: Vimscript 명령 실행
- **`vim.notify`**: `print`보다 나은 사용자 알림
- **모듈 패턴**: `local M = {} ... return M`으로 재사용 가능한 코드를 만든다
- **`vim.inspect()`**은 Lua 디버깅의 핵심 도구다. 테이블 내용을 한눈에 확인할 수 있다
- **`vim.tbl_deep_extend`**, **`vim.tbl_filter`** 등 Neovim 내장 유틸리티를 활용하자
