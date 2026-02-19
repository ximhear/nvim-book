# Chapter 21: 실전 워크플로우

지금까지 Vim의 편집 언어, 설정, 플러그인을 하나씩 배워왔다. 이 챕터에서는 그 모든 조각을 **하나의 흐름으로 연결**한다. 실제 개발에서 마주치는 상황별로 어떤 도구를 어떤 순서로 사용하는지, 손에 익을 때까지 반복할 수 있는 워크플로우를 제시한다.

## 프로젝트 탐색 워크플로우

코드를 작성하는 시간보다 **읽고 찾는 시간**이 더 길다. Telescope, LSP, Treesitter를 조합하면 프로젝트 안에서 원하는 코드에 빠르게 도달할 수 있다.

### 파일 찾기 → 심볼 검색 → 정의 이동 → 참조 찾기

전형적인 탐색 흐름은 다음과 같다:

```
1. 파일 찾기         <leader>ff    (Telescope find_files)
2. 텍스트 검색       <leader>fg    (Telescope live_grep)
3. 심볼 검색         <leader>fs    (Telescope lsp_document_symbols)
4. 정의로 이동       gd            (LSP go to definition)
5. 참조 찾기         gr            (LSP references)
6. 이전 위치로 복귀  <C-o>         (jump list)
```

#### 실전 시나리오: "이 함수를 누가 호출하고 있지?"

1. `<leader>ff`로 파일을 열거나, `<leader>fg`로 함수 이름을 검색한다
2. 함수 정의 위에 커서를 놓는다
3. `gr`로 참조 목록을 확인한다 (Telescope 또는 quickfix list로 표시)
4. 목록에서 원하는 위치를 선택하여 이동한다
5. `<C-o>`로 원래 위치로 돌아온다

#### 실전 시나리오: "이 타입의 구조를 파악하고 싶다"

1. `<leader>fs`로 현재 파일의 심볼 목록을 연다
2. 또는 `<leader>fS`로 워크스페이스 전체 심볼을 검색한다
3. 타입 정의로 이동한 뒤 `K`로 문서를 확인한다
4. `gd`로 관련 타입의 정의를 따라간다

### 탐색 키맵 정리

```lua
-- init.lua 또는 키맵 설정 파일
local builtin = require('telescope.builtin')

-- 파일 탐색
vim.keymap.set('n', '<leader>ff', builtin.find_files, { desc = '파일 찾기' })
vim.keymap.set('n', '<leader>fg', builtin.live_grep, { desc = '텍스트 검색' })
vim.keymap.set('n', '<leader>fb', builtin.buffers, { desc = '열린 버퍼 목록' })
vim.keymap.set('n', '<leader>fr', builtin.oldfiles, { desc = '최근 파일' })

-- LSP 심볼 탐색
vim.keymap.set('n', '<leader>fs', builtin.lsp_document_symbols, { desc = '문서 심볼' })
vim.keymap.set('n', '<leader>fS', builtin.lsp_workspace_symbols, { desc = '워크스페이스 심볼' })

-- LSP 네비게이션
vim.keymap.set('n', 'gd', vim.lsp.buf.definition, { desc = '정의로 이동' })
vim.keymap.set('n', 'gr', builtin.lsp_references, { desc = '참조 찾기' })
vim.keymap.set('n', 'gI', vim.lsp.buf.implementation, { desc = '구현으로 이동' })
vim.keymap.set('n', 'K', vim.lsp.buf.hover, { desc = '문서 보기' })
```

## 코드 편집 워크플로우

효율적인 편집은 "올바른 도구를 올바른 상황에 사용하는 것"이다.

### 빠른 변수명 변경: LSP rename vs ciw + n.

변수명을 변경하는 방법은 크게 두 가지다:

**방법 1: LSP rename (의미 기반)**

```lua
vim.keymap.set('n', '<leader>rn', vim.lsp.buf.rename, { desc = '이름 변경' })
```

커서를 변수 위에 놓고 `<leader>rn` → 새 이름 입력 → `<CR>`. LSP가 프로젝트 전체에서 해당 심볼의 모든 참조를 정확하게 변경한다. 단순한 텍스트 치환이 아니라 **의미를 이해한** 변경이다.

**방법 2: ciw + n. (수동, 선택적)**

1. `*`로 현재 단어를 검색한다
2. `ciw`로 단어를 변경하고 새 이름을 입력한다
3. `<Esc>` → `n`으로 다음 일치로 이동
4. `.`으로 변경을 반복하거나, `n`으로 건너뛴다

LSP rename이 더 정확하지만, **선택적으로 변경하고 싶을 때**는 `ciw` + `n.` 패턴이 유용하다.

### 여러 파일 동시 변경: quickfix list 활용

quickfix list는 여러 파일에 걸친 검색 결과를 한 곳에 모아 순차적으로 처리할 수 있는 도구다.

```vim
" Telescope live_grep 결과를 quickfix로 보내기
" 검색 결과 화면에서 <C-q>

" quickfix 탐색
:cnext       " 다음 항목 (]q로 매핑 권장)
:cprev       " 이전 항목 ([q로 매핑 권장)
:copen       " quickfix 창 열기
:cclose      " quickfix 창 닫기
```

#### 실전 시나리오: 여러 파일에서 API 엔드포인트 변경

1. `<leader>fg`로 `/api/v1/users`를 검색한다
2. 결과 목록에서 변경할 항목을 `<Tab>`으로 선택한다 (또는 전체 선택)
3. `<C-q>`로 quickfix list에 보낸다
4. `:cdo s/\/api\/v1/\/api\/v2/g | update`로 일괄 변경한다

```lua
-- quickfix 탐색 키맵
vim.keymap.set('n', ']q', ':cnext<CR>', { desc = '다음 quickfix 항목' })
vim.keymap.set('n', '[q', ':cprev<CR>', { desc = '이전 quickfix 항목' })
```

### 코드 블록 이동/복제: 줄 이동 키맵

코드 줄이나 블록을 위아래로 이동하는 키맵은 편집 속도를 크게 높인다.

```lua
-- 줄 이동 (Alt + j/k)
vim.keymap.set('n', '<A-j>', ':m .+1<CR>==', { desc = '줄 아래로 이동' })
vim.keymap.set('n', '<A-k>', ':m .-2<CR>==', { desc = '줄 위로 이동' })
vim.keymap.set('v', '<A-j>', ":m '>+1<CR>gv=gv", { desc = '선택 영역 아래로 이동' })
vim.keymap.set('v', '<A-k>', ":m '<-2<CR>gv=gv", { desc = '선택 영역 위로 이동' })

-- 줄 복제
vim.keymap.set('n', '<leader>d', 'yyp', { desc = '현재 줄 복제' })
```

## 리팩토링 워크플로우

### 함수 추출, 변수 추출

Neovim에서의 리팩토링은 Vim의 편집 기능과 LSP 코드 액션(code action)을 조합하여 수행한다.

**함수 추출 (수동)**

1. `V`로 추출할 코드 블록을 줄 단위로 선택한다
2. `d`로 잘라낸다
3. 적절한 위치로 이동하여 새 함수를 작성한다
4. `p`로 코드를 붙여넣는다
5. 원래 위치에 함수 호출을 추가한다

**변수 추출 (수동)**

1. 반복되는 표현식 위에서 `viw` 또는 적절한 텍스트 오브젝트로 선택한다
2. `c`로 변경하고 새 변수명을 입력한다
3. 윗줄에 변수 선언을 추가한다 (`O`로 위에 새 줄 추가)
4. `n.`으로 나머지 동일 표현식도 변경한다

### LSP 코드 액션 활용

LSP 서버가 제공하는 코드 액션은 자동 리팩토링의 핵심이다.

```lua
vim.keymap.set('n', '<leader>ca', vim.lsp.buf.code_action, { desc = '코드 액션' })
vim.keymap.set('v', '<leader>ca', vim.lsp.buf.code_action, { desc = '코드 액션 (선택)' })
```

커서를 코드 위에 놓고 `<leader>ca`를 누르면 사용 가능한 액션 목록이 표시된다:

- 누락된 import 추가
- 사용하지 않는 import 제거
- 함수 추출 (일부 LSP 서버 지원)
- 타입 어노테이션 추가
- Quick fix 제안

### 검색/치환으로 대규모 변경

프로젝트 전체에 걸친 대규모 변경은 검색/치환(search and replace)이 효율적이다.

```vim
" 현재 파일에서 치환
:%s/oldName/newName/gc     " c 플래그: 각 항목 확인

" 여러 파일에서 치환 (quickfix + cdo)
:grep oldName -r src/      " 또는 Telescope live_grep 후 <C-q>
:cdo s/oldName/newName/g | update

" 정규표현식 활용
:%s/console\.log(.*)/\/\/ console.log(\1)/g   " 로그 주석 처리
```

## 디버깅 워크플로우: nvim-dap 소개

nvim-dap(Debug Adapter Protocol)은 Neovim에서 본격적인 디버깅을 가능하게 하는 플러그인이다. VSCode와 동일한 DAP 프로토콜을 사용하므로 다양한 언어를 지원한다.

### nvim-dap 설치와 기본 설정

```lua
-- lazy.nvim 설정
{
  'mfussenegger/nvim-dap',
  dependencies = {
    'rcarriga/nvim-dap-ui',
    'nvim-neotest/nvim-nio',
    'theHamsta/nvim-dap-virtual-text',
  },
  config = function()
    local dap = require('dap')
    local dapui = require('dapui')

    dapui.setup()

    -- dap-ui 자동 열기/닫기
    dap.listeners.after.event_initialized['dapui_config'] = function()
      dapui.open()
    end
    dap.listeners.before.event_terminated['dapui_config'] = function()
      dapui.close()
    end
    dap.listeners.before.event_exited['dapui_config'] = function()
      dapui.close()
    end
  end,
}
```

### 브레이크포인트, 스텝 실행

디버깅의 기본 동작은 다음 키맵으로 제어한다:

```lua
local dap = require('dap')

vim.keymap.set('n', '<F5>', dap.continue, { desc = '디버그 시작/계속' })
vim.keymap.set('n', '<F10>', dap.step_over, { desc = '스텝 오버' })
vim.keymap.set('n', '<F11>', dap.step_into, { desc = '스텝 인투' })
vim.keymap.set('n', '<F12>', dap.step_out, { desc = '스텝 아웃' })
vim.keymap.set('n', '<leader>b', dap.toggle_breakpoint, { desc = '브레이크포인트 토글' })
vim.keymap.set('n', '<leader>B', function()
  dap.set_breakpoint(vim.fn.input('Breakpoint condition: '))
end, { desc = '조건부 브레이크포인트' })
```

디버깅 흐름:

```
1. <leader>b  → 브레이크포인트 설정
2. <F5>       → 디버그 시작
3. <F10>      → 한 줄씩 실행 (step over)
4. <F11>      → 함수 안으로 진입 (step into)
5. <F12>      → 함수 밖으로 나가기 (step out)
6. <F5>       → 다음 브레이크포인트까지 계속
```

### dap-ui로 디버깅 UI

nvim-dap-ui는 디버깅 중 변수, 콜 스택(call stack), 브레이크포인트 목록을 시각적으로 보여준다.

```
┌─────────────────┬──────────────────────────┬─────────────────┐
│   Variables     │                          │   Call Stack    │
│                 │                          │                 │
│ x = 42          │     소스 코드             │ main()          │
│ name = "hello" │     (브레이크포인트 표시)    │  └ calculate()  │
│ result = nil   │                          │    └ add()      │
│                 │                          │                 │
├─────────────────┤                          ├─────────────────┤
│   Watches       │                          │   Breakpoints   │
│                 │                          │                 │
│ x + y = ?      │                          │ main.py:10      │
│                 │                          │ utils.py:25     │
└─────────────────┴──────────────────────────┴─────────────────┘
│                        REPL / Console                        │
└──────────────────────────────────────────────────────────────┘
```

### 언어별 어댑터 설정

각 언어에 맞는 디버그 어댑터(debug adapter)를 설정해야 한다. 몇 가지 대표적인 예시를 소개한다:

```lua
-- Python (debugpy)
dap.adapters.python = {
  type = 'executable',
  command = 'python',
  args = { '-m', 'debugpy.adapter' },
}
dap.configurations.python = {
  {
    type = 'python',
    request = 'launch',
    name = 'Launch file',
    program = '${file}',
  },
}

-- Node.js / TypeScript 등은
-- 각 언어 전용 플러그인 활용 권장
-- 예: nvim-dap-python, nvim-dap-go 등
```

> **팁**: 모든 언어를 직접 설정할 필요는 없다. `nvim-dap-python`, `nvim-dap-go` 같은 언어별 플러그인이 어댑터 설정을 자동으로 처리해 준다.

## 터미널 통합

### Neovim 내장 터미널: :terminal

Neovim에는 터미널 에뮬레이터(terminal emulator)가 내장되어 있다. 편집기를 떠나지 않고 셸 명령을 실행할 수 있다.

```vim
:terminal          " 현재 창에서 터미널 열기
:split | terminal  " 수평 분할 후 터미널 열기
:vsplit | terminal " 수직 분할 후 터미널 열기
```

터미널에서 Normal 모드로 전환하려면 `<C-\><C-n>`을 누른다. 다시 터미널 입력 모드로 돌아가려면 `i` 또는 `a`를 누른다.

```lua
-- 터미널 모드에서 Esc로 Normal 모드 전환
vim.keymap.set('t', '<Esc><Esc>', '<C-\\><C-n>', { desc = '터미널 Normal 모드' })

-- 터미널에서 창 이동
vim.keymap.set('t', '<C-h>', '<C-\\><C-n><C-w>h', { desc = '왼쪽 창으로' })
vim.keymap.set('t', '<C-j>', '<C-\\><C-n><C-w>j', { desc = '아래 창으로' })
vim.keymap.set('t', '<C-k>', '<C-\\><C-n><C-w>k', { desc = '위 창으로' })
vim.keymap.set('t', '<C-l>', '<C-\\><C-n><C-w>l', { desc = '오른쪽 창으로' })
```

### toggleterm.nvim으로 터미널 관리

toggleterm.nvim은 터미널을 토글(toggle) 방식으로 열고 닫을 수 있는 플러그인이다.

```lua
{
  'akinsho/toggleterm.nvim',
  version = '*',
  opts = {
    size = 20,
    open_mapping = [[<C-\>]],
    direction = 'horizontal', -- horizontal, vertical, float, tab
    shade_terminals = true,
    float_opts = {
      border = 'curved',
    },
  },
}
```

설정 후 `<C-\>`로 터미널을 토글할 수 있다. 여러 터미널을 번호로 관리할 수도 있다:

```vim
:1ToggleTerm    " 1번 터미널
:2ToggleTerm    " 2번 터미널
```

특정 명령을 전용 터미널에서 실행하는 패턴도 유용하다:

```lua
local Terminal = require('toggleterm.terminal').Terminal

-- lazygit 전용 터미널
local lazygit = Terminal:new({
  cmd = 'lazygit',
  direction = 'float',
  hidden = true,
})

vim.keymap.set('n', '<leader>gg', function()
  lazygit:toggle()
end, { desc = 'Lazygit 열기' })
```

### 터미널 모드 키맵

Neovim 터미널의 모드 전환을 정리하면 다음과 같다:

```
┌─────────────────────────────────────────────┐
│  Terminal 모드 (입력이 셸로 전달됨)           │
│                                             │
│  <C-\><C-n> → Normal 모드로 전환             │
│  i 또는 a   ← Normal 모드에서 Terminal로 복귀 │
└─────────────────────────────────────────────┘
```

> **팁**: `<C-\><C-n>`이 길게 느껴지면 `<Esc><Esc>`로 매핑하는 것을 권장한다. 단, 단일 `<Esc>`로 매핑하면 셸에서 Escape가 필요한 경우 (예: Vi 모드) 충돌이 발생할 수 있다.

## Git 워크플로우

### gitsigns로 hunk 단위 작업

gitsigns.nvim은 Git 변경 사항을 줄 단위로 표시하고, hunk(변경 덩어리) 단위로 작업할 수 있게 해준다.

```lua
{
  'lewis6991/gitsigns.nvim',
  opts = {
    on_attach = function(bufnr)
      local gs = require('gitsigns')

      local function map(mode, l, r, desc)
        vim.keymap.set(mode, l, r, { buffer = bufnr, desc = desc })
      end

      -- hunk 탐색
      map('n', ']h', gs.next_hunk, '다음 hunk')
      map('n', '[h', gs.prev_hunk, '이전 hunk')

      -- hunk 조작
      map('n', '<leader>hs', gs.stage_hunk, 'Hunk 스테이지')
      map('n', '<leader>hr', gs.reset_hunk, 'Hunk 리셋')
      map('n', '<leader>hu', gs.undo_stage_hunk, 'Hunk 스테이지 취소')
      map('n', '<leader>hp', gs.preview_hunk, 'Hunk 미리보기')
      map('n', '<leader>hb', gs.blame_line, '줄 blame')

      -- Visual 모드에서 부분 스테이지
      map('v', '<leader>hs', function()
        gs.stage_hunk({ vim.fn.line('.'), vim.fn.line('v') })
      end, '선택 영역 스테이지')
    end,
  },
}
```

hunk 단위 워크플로우:

```
1. ]h / [h    → hunk 사이를 탐색
2. <leader>hp → 변경 내용 미리보기
3. <leader>hs → 원하는 hunk만 스테이지
4. <leader>hr → 불필요한 변경 되돌리기
```

### fugitive/neogit로 commit, push

**vim-fugitive**는 Vim에서 가장 오래되고 검증된 Git 플러그인이다. **neogit**은 Magit(Emacs)에서 영감받은 더 현대적인 인터페이스를 제공한다.

```lua
-- fugitive 기본 사용법
vim.keymap.set('n', '<leader>gs', ':Git<CR>', { desc = 'Git status' })
vim.keymap.set('n', '<leader>gc', ':Git commit<CR>', { desc = 'Git commit' })
vim.keymap.set('n', '<leader>gp', ':Git push<CR>', { desc = 'Git push' })
vim.keymap.set('n', '<leader>gl', ':Git log --oneline<CR>', { desc = 'Git log' })
```

```lua
-- 또는 neogit 사용
{
  'NeogitOrg/neogit',
  dependencies = { 'nvim-lua/plenary.nvim', 'sindrets/diffview.nvim' },
  opts = {},
  keys = {
    { '<leader>gn', '<cmd>Neogit<CR>', desc = 'Neogit 열기' },
  },
}
```

### diffview.nvim으로 diff 보기

diffview.nvim은 Git diff를 VSCode처럼 나란히(side-by-side) 보여준다.

```lua
{
  'sindrets/diffview.nvim',
  keys = {
    { '<leader>gd', '<cmd>DiffviewOpen<CR>', desc = 'Diff 보기' },
    { '<leader>gh', '<cmd>DiffviewFileHistory %<CR>', desc = '파일 히스토리' },
    { '<leader>gH', '<cmd>DiffviewFileHistory<CR>', desc = '프로젝트 히스토리' },
  },
}
```

```vim
:DiffviewOpen              " 현재 변경 사항 diff
:DiffviewOpen HEAD~3       " 3개 커밋 전과 비교
:DiffviewFileHistory %     " 현재 파일의 Git 히스토리
:DiffviewClose             " diff 뷰 닫기
```

### Git 워크플로우 종합

일상적인 Git 작업 흐름을 정리하면 다음과 같다:

```
1. 코드 작성
2. ]h / [h        → 변경된 hunk 확인
3. <leader>hp     → 변경 내용 미리보기
4. <leader>hs     → 원하는 변경만 스테이지
5. <leader>gd     → 전체 diff 확인
6. <leader>gc     → 커밋
7. <leader>gp     → 푸시
```

## 프로젝트별 설정: exrc, .nvim.lua

프로젝트마다 다른 포매터(formatter), 린터(linter), 빌드 명령이 필요할 수 있다. Neovim은 프로젝트 루트에 있는 설정 파일을 자동으로 로드하는 기능을 제공한다.

### exrc 활성화

```lua
-- init.lua
vim.o.exrc = true   -- 프로젝트 로컬 설정 파일 허용
```

이제 프로젝트 루트에 `.nvim.lua` 파일을 만들면 해당 디렉터리에서 Neovim을 열 때 자동으로 실행된다.

### .nvim.lua 활용 예시

```lua
-- 프로젝트 루트의 .nvim.lua

-- 프로젝트 전용 탭 크기
vim.bo.tabstop = 4
vim.bo.shiftwidth = 4

-- 프로젝트 전용 빌드 명령
vim.keymap.set('n', '<leader>mb', function()
  vim.cmd('split | terminal make build')
end, { desc = '프로젝트 빌드' })

-- 프로젝트 전용 테스트 명령
vim.keymap.set('n', '<leader>mt', function()
  vim.cmd('split | terminal npm test')
end, { desc = '테스트 실행' })
```

> **주의**: `exrc`를 활성화하면 신뢰할 수 없는 프로젝트의 `.nvim.lua`가 실행될 수 있다. Neovim 0.9 이상에서는 처음 실행 시 신뢰 여부를 묻는 프롬프트가 표시된다.

## 실습: 워크플로우 연습

다음 시나리오를 직접 연습해 보자:

**연습 1: 탐색 워크플로우**

1. 자신의 프로젝트를 Neovim으로 연다
2. `<leader>fg`로 특정 함수 이름을 검색하여 정의를 찾는다
3. `gd`로 관련 타입의 정의로 이동한다
4. `gr`로 해당 타입이 사용되는 모든 곳을 확인한다
5. `<C-o>`로 원래 위치로 돌아온다

**연습 2: 편집 워크플로우**

1. 변수명 하나를 선택하고 `<leader>rn`으로 LSP rename을 수행한다
2. `<leader>fg`로 특정 문자열을 검색하고 `<C-q>`로 quickfix에 보낸다
3. `:cdo s/old/new/g | update`로 일괄 변경한다

**연습 3: Git 워크플로우**

1. 파일을 수정한 뒤 `]h`로 변경된 hunk을 탐색한다
2. `<leader>hp`로 변경 내용을 확인한다
3. `<leader>hs`로 원하는 hunk만 스테이지한다
4. `:Git commit` 또는 neogit으로 커밋한다

## 핵심 정리

- **탐색**: Telescope(`<leader>ff`, `<leader>fg`) → LSP(`gd`, `gr`) → jump list(`<C-o>`)로 이어지는 흐름을 체화하라
- **편집**: LSP rename(`<leader>rn`)은 정확하고, `ciw` + `n.`은 유연하다. 상황에 맞게 선택한다
- **quickfix list**: 여러 파일에 걸친 변경의 핵심 도구. Telescope 검색 → `<C-q>` → `:cdo` 패턴을 기억하라
- **디버깅**: nvim-dap으로 브레이크포인트 기반 디버깅이 가능하다. dap-ui와 함께 사용하면 시각적 디버깅 환경이 완성된다
- **터미널**: Neovim 내장 터미널과 toggleterm.nvim으로 편집기를 떠나지 않고 셸 작업을 수행한다
- **Git**: gitsigns(hunk 단위) + fugitive/neogit(commit/push) + diffview(diff 확인)로 완전한 Git 워크플로우를 구성한다
- **프로젝트별 설정**: `.nvim.lua`로 프로젝트마다 다른 설정, 키맵, 빌드 명령을 정의할 수 있다
